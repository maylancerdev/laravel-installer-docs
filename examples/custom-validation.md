---
title: "Custom Validation"
weight: 2
---

# Custom Validation Examples

Advanced validation patterns for installer steps.

## Database connection validation

Test database credentials before saving.

```php
namespace App\Installer\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Olakunlevpn\Installer\Concerns\HasFormData;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Config;

class DatabaseStep extends BaseStep
{
    use HasFormData;

    protected function validateStep(): void
    {
        // First run basic validation
        $this->validate();

        // Then test the connection
        if (!$this->testDatabaseConnection()) {
            $this->errorMessage = __('database::database.connection_failed');
            return;
        }

        $this->successMessage = __('database::database.connection_success');
    }

    protected function testDatabaseConnection(): bool
    {
        try {
            // Temporarily set database config
            Config::set('database.connections.test', [
                'driver' => $this->formData['driver'],
                'host' => $this->formData['host'],
                'port' => $this->formData['port'],
                'database' => $this->formData['database'],
                'username' => $this->formData['username'],
                'password' => $this->formData['password'],
                'charset' => 'utf8mb4',
                'collation' => 'utf8mb4_unicode_ci',
            ]);

            // Test connection
            DB::connection('test')->getPdo();

            // Try to select from database
            DB::connection('test')->select('SELECT 1');

            return true;
        } catch (\Exception $e) {
            Log::error('Database connection test failed', [
                'error' => $e->getMessage(),
                'host' => $this->formData['host'],
            ]);

            return false;
        }
    }
}
```

## API key validation

Verify API key with remote server.

```php
protected function validateStep(): void
{
    $this->validate();

    // Validate API key format
    if (!$this->isValidKeyFormat($this->formData['api_key'])) {
        $this->addError('formData.api_key', __('api::api.invalid_format'));
        return;
    }

    // Verify with remote server
    if (!$this->verifyApiKey($this->formData['api_key'])) {
        $this->addError('formData.api_key', __('api::api.verification_failed'));
        return;
    }

    $this->successMessage = __('api::api.verified');
}

protected function isValidKeyFormat(string $key): bool
{
    // Example: key must be 40 characters, alphanumeric
    return preg_match('/^[a-zA-Z0-9]{40}$/', $key);
}

protected function verifyApiKey(string $key): bool
{
    try {
        $response = Http::timeout(10)
            ->withHeaders(['X-API-Key' => $key])
            ->get('https://api.example.com/validate');

        return $response->successful() && $response->json('valid');
    } catch (\Exception $e) {
        return false;
    }
}
```

## Email validation with DNS check

Verify email exists and domain has MX records.

```php
protected function validateStep(): void
{
    $this->validate();

    $email = $this->formData['email'];

    // Check DNS for MX records
    if (!$this->hasValidMxRecords($email)) {
        $this->addError('formData.email', __('account::account.invalid_email_domain'));
        return;
    }

    // Optional: Send verification code
    if (config('installer.verify_email', false)) {
        $this->sendVerificationCode($email);
        $this->successMessage = __('account::account.verification_sent');
    }
}

protected function hasValidMxRecords(string $email): bool
{
    $domain = substr(strrchr($email, '@'), 1);

    return checkdnsrr($domain, 'MX');
}

protected function sendVerificationCode(string $email): void
{
    $code = rand(100000, 999999);

    $this->storeInSession('verification_code', $code);
    $this->storeInSession('verification_email', $email);

    Mail::to($email)->send(new VerificationCode($code));
}
```

## URL reachability check

Verify URL is accessible.

```php
protected function validateStep(): void
{
    $this->validate();

    $url = $this->formData['webhook_url'];

    // Validate URL format
    if (!filter_var($url, FILTER_VALIDATE_URL)) {
        $this->addError('formData.webhook_url', __('webhook::webhook.invalid_url'));
        return;
    }

    // Check if URL is reachable
    if (!$this->isUrlReachable($url)) {
        $this->addError('formData.webhook_url', __('webhook::webhook.unreachable'));
        return;
    }

    $this->successMessage = __('webhook::webhook.verified');
}

protected function isUrlReachable(string $url): bool
{
    try {
        $response = Http::timeout(5)->head($url);

        return $response->successful() || $response->status() === 405;
    } catch (\Exception $e) {
        return false;
    }
}
```

## File upload validation

Validate uploaded files.

```php
namespace App\Installer\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Illuminate\Support\Facades\Storage;

class LogoUploadStep extends BaseStep
{
    public $logo;

    protected function rules(): array
    {
        return [
            'logo' => 'required|image|mimes:png,jpg,svg|max:2048',
        ];
    }

    protected function validateStep(): void
    {
        $this->validate();

        // Additional validation
        if (!$this->isValidImageDimensions()) {
            $this->addError('logo', __('logo::logo.invalid_dimensions'));
            return;
        }
    }

    protected function isValidImageDimensions(): bool
    {
        $path = $this->logo->getRealPath();
        $imageSize = getimagesize($path);

        if (!$imageSize) {
            return false;
        }

        [$width, $height] = $imageSize;

        // Logo must be at least 200x200
        return $width >= 200 && $height >= 200;
    }

    protected function execute(): void
    {
        $path = $this->logo->store('logos', 'public');

        $this->storeInSession('logo_path', $path);

        Setting::updateOrCreate(
            ['key' => 'app_logo'],
            ['value' => $path]
        );
    }
}
```

## Multi-field validation

Validate fields together.

```php
protected function validateStep(): void
{
    $this->validate();

    // Password confirmation
    if ($this->formData['password'] !== $this->formData['password_confirmation']) {
        $this->addError('formData.password_confirmation', __('account::account.passwords_must_match'));
        return;
    }

    // Port range based on protocol
    $protocol = $this->formData['mail_protocol'];
    $port = $this->formData['mail_port'];

    if ($protocol === 'smtp' && !in_array($port, [25, 465, 587])) {
        $this->addError('formData.mail_port', __('mail::mail.invalid_smtp_port'));
        return;
    }

    // Validate SMTP credentials if enabled
    if ($this->formData['mail_enabled'] && !$this->testSmtpConnection()) {
        $this->errorMessage = __('mail::mail.connection_failed');
        return;
    }

    $this->successMessage = __('mail::mail.validated');
}
```

## Async validation with progress

Show validation progress for slow checks.

```php
namespace App\Installer\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Livewire\Attributes\On;

class SystemCheckStep extends BaseStep
{
    public array $checks = [];
    public bool $validating = false;
    public int $progress = 0;

    protected function mount(): void
    {
        $this->checks = [
            'php_version' => ['status' => 'pending', 'message' => ''],
            'extensions' => ['status' => 'pending', 'message' => ''],
            'permissions' => ['status' => 'pending', 'message' => ''],
            'database' => ['status' => 'pending', 'message' => ''],
        ];
    }

    public function runChecks(): void
    {
        $this->validating = true;
        $this->progress = 0;

        // Check PHP version
        $this->checkPhpVersion();
        $this->progress = 25;

        // Check extensions
        $this->checkExtensions();
        $this->progress = 50;

        // Check permissions
        $this->checkPermissions();
        $this->progress = 75;

        // Check database
        $this->checkDatabase();
        $this->progress = 100;

        $this->validating = false;
    }

    protected function checkPhpVersion(): void
    {
        $required = '8.2.0';

        if (version_compare(PHP_VERSION, $required, '>=')) {
            $this->checks['php_version'] = [
                'status' => 'passed',
                'message' => "PHP " . PHP_VERSION,
            ];
        } else {
            $this->checks['php_version'] = [
                'status' => 'failed',
                'message' => "PHP {$required}+ required",
            ];
        }
    }

    protected function checkExtensions(): void
    {
        $required = ['pdo', 'mbstring', 'openssl', 'tokenizer', 'xml'];
        $missing = array_filter($required, fn($ext) => !extension_loaded($ext));

        if (empty($missing)) {
            $this->checks['extensions'] = [
                'status' => 'passed',
                'message' => 'All extensions loaded',
            ];
        } else {
            $this->checks['extensions'] = [
                'status' => 'failed',
                'message' => 'Missing: ' . implode(', ', $missing),
            ];
        }
    }

    protected function validateStep(): void
    {
        $failed = array_filter($this->checks, fn($check) => $check['status'] === 'failed');

        if (!empty($failed)) {
            $this->errorMessage = __('system::system.checks_failed');
            return;
        }
    }
}
```

View with progress:

```blade
<div>
    <x-installer::card title="System Requirements">
        @if($validating)
            <div class="mb-4">
                <div class="w-full bg-gray-700 rounded-full h-2">
                    <div class="bg-primary h-2 rounded-full transition-all" style="width: {{ $progress }}%"></div>
                </div>
                <p class="text-sm text-gray-400 mt-2">Checking system... {{ $progress }}%</p>
            </div>
        @endif

        <div class="space-y-3">
            @foreach($checks as $name => $check)
                <div class="flex items-center gap-3">
                    @if($check['status'] === 'passed')
                        <x-installer::icon name="check" class="text-green-500" />
                    @elseif($check['status'] === 'failed')
                        <x-installer::icon name="x" class="text-red-500" />
                    @else
                        <x-installer::icon name="loading" class="text-gray-500 animate-spin" />
                    @endif

                    <div>
                        <p class="font-medium">{{ ucfirst(str_replace('_', ' ', $name)) }}</p>
                        <p class="text-sm text-gray-400">{{ $check['message'] }}</p>
                    </div>
                </div>
            @endforeach
        </div>

        <div class="mt-6">
            <x-installer::button
                type="button"
                wire:click="runChecks"
                :disabled="$validating"
            >
                Run Checks
            </x-installer::button>
        </div>
    </x-installer::card>
</div>
```

## Conditional validation

Validate fields based on other fields.

```php
protected function rules(): array
{
    $rules = [
        'formData.mail_driver' => 'required|in:smtp,sendmail,mailgun',
    ];

    // SMTP-specific fields
    if ($this->formData['mail_driver'] === 'smtp') {
        $rules['formData.smtp_host'] = 'required|string';
        $rules['formData.smtp_port'] = 'required|integer|min:1|max:65535';
        $rules['formData.smtp_username'] = 'required|string';
        $rules['formData.smtp_password'] = 'required|string';
    }

    // Mailgun-specific fields
    if ($this->formData['mail_driver'] === 'mailgun') {
        $rules['formData.mailgun_domain'] = 'required|string';
        $rules['formData.mailgun_secret'] = 'required|string';
    }

    return $rules;
}
```

## Summary

Key validation patterns:
- Test external connections
- Verify API responses
- Check file formats and dimensions
- Validate related fields together
- Show progress for slow checks
- Use conditional validation
- Provide clear error messages

Always validate thoroughly to prevent installation issues.
