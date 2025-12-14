---
title: "Complete Plugin Example"
weight: 1
---

# Complete Plugin Example

A full working example of an installer plugin that verifies a license key via API.

## Package structure

```
packages/license/
├── config/
│   └── installer_license.php
├── lang/
│   └── en/
│       └── license.php
├── resources/
│   └── views/
│       └── livewire/
│           └── steps/
│               └── license-verification.blade.php
├── src/
│   ├── Services/
│   │   └── LicenseVerifier.php
│   ├── Steps/
│   │   └── LicenseVerificationStep.php
│   └── LicenseServiceProvider.php
└── composer.json
```

## Configuration file

`config/installer_license.php`:

```php
<?php

return [
    'step_position' => env('LICENSE_STEP_POSITION', 5),

    'api_url' => env('LICENSE_API_URL', 'https://api.example.com/verify'),

    'timeout' => env('LICENSE_API_TIMEOUT', 10),

    'form_fields' => [
        [
            'name' => 'purchase_code',
            'type' => 'text',
            'label' => 'license::license.purchase_code',
            'placeholder' => 'license::license.purchase_code_placeholder',
            'required' => true,
            'validation' => 'required|string|min:36|max:36',
            'store_in_db' => true,
            'column' => 'license_key',
        ],
        [
            'name' => 'email',
            'type' => 'email',
            'label' => 'license::license.email',
            'placeholder' => 'license::license.email_placeholder',
            'required' => true,
            'validation' => 'required|email',
            'store_in_db' => true,
        ],
    ],
];
```

## Translation file

`lang/en/license.php`:

```php
<?php

return [
    'card_title' => 'License Verification',
    'card_description' => 'Enter your purchase code to activate the application',

    'purchase_code' => 'Purchase Code',
    'purchase_code_placeholder' => 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx',

    'email' => 'Email Address',
    'email_placeholder' => 'your@email.com',

    'submit' => 'Verify License',
    'verifying' => 'Verifying...',

    'verified' => 'License verified successfully!',
    'invalid_code' => 'Invalid purchase code. Please check and try again.',
    'api_error' => 'Unable to verify license. Please check your internet connection.',
    'already_used' => 'This purchase code has already been used on another domain.',

    'help_title' => 'Where do I find my purchase code?',
    'help_text' => 'Your purchase code was sent to your email after purchase. You can also find it in your account dashboard.',
];
```

## Service provider

`src/LicenseServiceProvider.php`:

```php
<?php

namespace Olakunlevpn\License;

use Illuminate\Support\ServiceProvider;
use Olakunlevpn\Installer\Installer;
use Olakunlevpn\License\Steps\LicenseVerificationStep;

class LicenseServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Register step
        $installer = $this->app->make(Installer::class);
        $installer->addStep(LicenseVerificationStep::class);

        // Publish config
        $this->publishes([
            __DIR__.'/../config/installer_license.php' =>
                config_path('installer_license.php'),
        ], 'installer-license-config');

        // Load views
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'license');

        // Load translations
        $this->loadTranslationsFrom(__DIR__.'/../lang', 'license');

        // Merge config
        $this->mergeConfigFrom(
            __DIR__.'/../config/installer_license.php',
            'installer-license'
        );
    }

    public function register(): void
    {
        $this->app->singleton(Services\LicenseVerifier::class);
    }
}
```

## License verifier service

`src/Services/LicenseVerifier.php`:

```php
<?php

namespace Olakunlevpn\License\Services;

use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class LicenseVerifier
{
    protected string $apiUrl;
    protected int $timeout;

    public function __construct()
    {
        $this->apiUrl = config('installer-license.api_url');
        $this->timeout = config('installer-license.timeout', 10);
    }

    public function verify(string $purchaseCode, string $email): array
    {
        try {
            $response = Http::timeout($this->timeout)
                ->post($this->apiUrl, [
                    'purchase_code' => $purchaseCode,
                    'email' => $email,
                    'domain' => request()->getHost(),
                ]);

            if ($response->successful()) {
                $data = $response->json();

                return [
                    'success' => $data['valid'] ?? false,
                    'message' => $data['message'] ?? 'Unknown response',
                    'features' => $data['features'] ?? [],
                ];
            }

            if ($response->status() === 422) {
                return [
                    'success' => false,
                    'message' => __('license::license.invalid_code'),
                ];
            }

            if ($response->status() === 409) {
                return [
                    'success' => false,
                    'message' => __('license::license.already_used'),
                ];
            }

            return [
                'success' => false,
                'message' => __('license::license.api_error'),
            ];
        } catch (\Exception $e) {
            Log::error('License verification failed', [
                'purchase_code' => $purchaseCode,
                'email' => $email,
                'error' => $e->getMessage(),
            ]);

            return [
                'success' => false,
                'message' => __('license::license.api_error'),
            ];
        }
    }
}
```

## Step class

`src/Steps/LicenseVerificationStep.php`:

```php
<?php

namespace Olakunlevpn\License\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Olakunlevpn\Installer\Concerns\HasFormData;
use Olakunlevpn\Installer\Concerns\ManagesSessionData;
use Olakunlevpn\Installer\Concerns\EmitsInstallerEvents;
use Olakunlevpn\License\Services\LicenseVerifier;
use Illuminate\Support\Facades\DB;
use App\Models\Setting;

class LicenseVerificationStep extends BaseStep
{
    use HasFormData;
    use ManagesSessionData;
    use EmitsInstallerEvents;

    public string $successMessage = '';
    public string $errorMessage = '';

    protected function mount(): void
    {
        $this->emitStepStarted('license');

        // Load existing data from session
        $existingData = $this->retrieveStepData('license');

        if ($existingData) {
            $this->formData = $existingData;
        } else {
            $this->initializeFormData(
                config('installer-license.form_fields')
            );
        }
    }

    protected function rules(): array
    {
        return $this->buildRulesFromFields(
            config('installer-license.form_fields')
        );
    }

    protected function validateStep(): void
    {
        $this->validate();

        // Clear previous messages
        $this->successMessage = '';
        $this->errorMessage = '';

        // Verify license with API
        $verifier = app(LicenseVerifier::class);

        $result = $verifier->verify(
            $this->formData['purchase_code'],
            $this->formData['email']
        );

        if (!$result['success']) {
            $this->errorMessage = $result['message'];
            $this->addError('formData.purchase_code', $result['message']);
            return;
        }

        // Store verification result
        $this->successMessage = __('license::license.verified');
        $this->storeVerificationStatus(true, 'license_verified');

        // Store features if provided
        if (!empty($result['features'])) {
            $this->storeInSession('license_features', $result['features']);
        }
    }

    protected function execute(): void
    {
        // Always save to session
        $this->storeStepData('license', $this->formData);

        // Try to save to database if it's ready
        try {
            DB::connection()->getPdo();

            // Database is ready, save license data
            Setting::updateOrCreate(
                ['key' => 'license_key'],
                ['value' => $this->formData['purchase_code']]
            );

            Setting::updateOrCreate(
                ['key' => 'license_email'],
                ['value' => $this->formData['email']]
            );

            Setting::updateOrCreate(
                ['key' => 'license_verified_at'],
                ['value' => now()]
            );

            // Save features if available
            $features = $this->retrieveFromSession('license_features');
            if ($features) {
                Setting::updateOrCreate(
                    ['key' => 'license_features'],
                    ['value' => json_encode($features)]
                );
            }
        } catch (\Exception $e) {
            // Database not ready yet, session storage is enough
            // Data will be saved after migrations run
        }

        $this->emitStepCompleted('license', [
            'purchase_code' => $this->formData['purchase_code'],
            'email' => $this->formData['email'],
        ]);
    }

    public function render()
    {
        return view('license::livewire.steps.license-verification', [
            'fields' => config('installer-license.form_fields'),
        ]);
    }
}
```

## View template

`resources/views/livewire/steps/license-verification.blade.php`:

```blade
<div>
    <x-installer::card
        title="{{ __('license::license.card_title') }}"
        description="{{ __('license::license.card_description') }}"
    >
        <form wire:submit.prevent="submit">
            <div class="space-y-4">
                @foreach($fields as $field)
                    <x-installer::form.group :field="$field" wireModel="formData" />
                @endforeach
            </div>

            <div class="mt-6">
                <x-installer::button
                    type="submit"
                    wireTarget="submit"
                    loadingText="{{ __('license::license.verifying') }}"
                >
                    {{ __('license::license.submit') }}
                </x-installer::button>
            </div>
        </form>
    </x-installer::card>

    @if($successMessage)
        <x-installer::alert.success class="mt-6">
            {{ $successMessage }}
        </x-installer::alert.success>
    @endif

    @if($errorMessage)
        <x-installer::alert.error class="mt-6">
            {{ $errorMessage }}
        </x-installer::alert.error>
    @endif

    <x-installer::alert.info title="{{ __('license::license.help_title') }}" class="mt-6">
        {{ __('license::license.help_text') }}
    </x-installer::alert.info>
</div>
```

## Composer.json

`composer.json`:

```json
{
    "name": "olakunlevpn/installer-license",
    "description": "License verification step for Laravel Installer",
    "type": "library",
    "license": "MIT",
    "autoload": {
        "psr-4": {
            "Olakunlevpn\\License\\": "src/"
        }
    },
    "require": {
        "php": "^8.2",
        "olakunlevpn/laravel-installer": "^1.0",
        "illuminate/support": "^12.0",
        "illuminate/http": "^12.0"
    },
    "extra": {
        "laravel": {
            "providers": [
                "Olakunlevpn\\License\\LicenseServiceProvider"
            ]
        }
    }
}
```

## Installation

Users install via Composer:

```bash
composer require olakunlevpn/installer-license
```

The step automatically appears in the installer at position 5.

## Customization

Users can customize via config:

```bash
php artisan vendor:publish --tag=laravel-installer-license-config
```

Edit `config/installer_license.php` or set environment variables:

```
LICENSE_STEP_POSITION=4
LICENSE_API_URL=https://custom-api.com/verify
LICENSE_API_TIMEOUT=15
```

## Testing

Test the verifier service:

```php
namespace Tests\Feature;

use Tests\TestCase;
use Olakunlevpn\License\Services\LicenseVerifier;
use Illuminate\Support\Facades\Http;

class LicenseVerifierTest extends TestCase
{
    public function test_successful_verification()
    {
        Http::fake([
            '*' => Http::response([
                'valid' => true,
                'message' => 'License valid',
                'features' => ['feature1', 'feature2'],
            ], 200),
        ]);

        $verifier = new LicenseVerifier();
        $result = $verifier->verify('test-code', 'test@example.com');

        $this->assertTrue($result['success']);
        $this->assertEquals('License valid', $result['message']);
        $this->assertNotEmpty($result['features']);
    }

    public function test_invalid_license()
    {
        Http::fake([
            '*' => Http::response([], 422),
        ]);

        $verifier = new LicenseVerifier();
        $result = $verifier->verify('invalid-code', 'test@example.com');

        $this->assertFalse($result['success']);
    }
}
```

## Summary

This complete plugin demonstrates:
- Configuration management
- Translation keys
- Form field definition
- API integration
- Session data management
- Database integration
- Error handling
- Event emission
- View templates
- Service provider setup
- Composer package structure

Copy this structure for your own plugins.
