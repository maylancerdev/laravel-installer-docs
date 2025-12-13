# Events System

The installer fires events during the installation process. Listen to these events to add custom functionality.

## Available events

Five events are fired during installation:

- `InstallationStarted` - Installation begins
- `StepStarted` - Individual step begins
- `StepCompleted` - Individual step completes
- `StepFailed` - Individual step fails
- `InstallationCompleted` - Installation finishes

## InstallationStarted

Fired when the installation wizard is loaded.

```php
namespace Olakunlevpn\Installer\Events;

class InstallationStarted
{
    public string $installerId;
    public array $config;
}
```

**Listening:**

```php
namespace App\Listeners;

use Olakunlevpn\Installer\Events\InstallationStarted;
use Illuminate\Support\Facades\Log;

class LogInstallationStart
{
    public function handle(InstallationStarted $event): void
    {
        Log::info('Installation started', [
            'installer_id' => $event->installerId,
            'config' => $event->config,
        ]);

        // Send analytics
        // Create backup
        // Notify admin
    }
}
```

**Registering:**

`app/Providers/EventServiceProvider.php`:

```php
protected $listen = [
    InstallationStarted::class => [
        LogInstallationStart::class,
    ],
];
```

## StepStarted

Fired when a step begins.

```php
namespace Olakunlevpn\Installer\Events;

class StepStarted
{
    public string $stepId;
    public array $data;
}
```

**Example listener:**

```php
namespace App\Listeners;

use Olakunlevpn\Installer\Events\StepStarted;

class TrackStepProgress
{
    public function handle(StepStarted $event): void
    {
        session()->put("step_{$event->stepId}_started_at", now());

        if ($event->stepId === 'license') {
            // License step started, prepare validation
        }
    }
}
```

## StepCompleted

Fired when a step completes successfully.

```php
namespace Olakunlevpn\Installer\Events;

use Olakunlevpn\Installer\Contracts\Step;

class StepCompleted
{
    public Step $step;
    public string $stepId;
    public array $data;
}
```

**Example listener:**

```php
namespace App\Listeners;

use Olakunlevpn\Installer\Events\StepCompleted;
use Illuminate\Support\Facades\Log;

class ProcessStepData
{
    public function handle(StepCompleted $event): void
    {
        Log::info("Step completed: {$event->stepId}", $event->data);

        // Process completed step data
        if ($event->stepId === 'database') {
            $this->testDatabaseConnection($event->data);
        }

        if ($event->stepId === 'license') {
            $this->activateLicense($event->data);
        }
    }

    protected function testDatabaseConnection(array $data): void
    {
        // Test connection with provided credentials
    }

    protected function activateLicense(array $data): void
    {
        // Activate license on remote server
    }
}
```

## StepFailed

Fired when a step fails validation or execution.

```php
namespace Olakunlevpn\Installer\Events;

class StepFailed
{
    public string $stepId;
    public \Exception $exception;
    public array $data;
}
```

**Example listener:**

```php
namespace App\Listeners;

use Olakunlevpn\Installer\Events\StepFailed;
use Illuminate\Support\Facades\Log;

class HandleStepFailure
{
    public function handle(StepFailed $event): void
    {
        Log::error("Step failed: {$event->stepId}", [
            'exception' => $event->exception->getMessage(),
            'trace' => $event->exception->getTraceAsString(),
            'data' => $event->data,
        ]);

        // Send error report
        // Notify admin
        // Rollback changes
    }
}
```

## InstallationCompleted

Fired when the entire installation finishes.

```php
namespace Olakunlevpn\Installer\Events;

class InstallationCompleted
{
    public string $installerId;
    public array $completedSteps;
    public float $duration;
}
```

**Example listener:**

```php
namespace App\Listeners;

use Olakunlevpn\Installer\Events\InstallationCompleted;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Mail;
use App\Mail\InstallationComplete;

class FinalizeInstallation
{
    public function handle(InstallationCompleted $event): void
    {
        Log::info('Installation completed', [
            'duration' => $event->duration,
            'steps' => $event->completedSteps,
        ]);

        // Clear session data
        session()->forget('installer_*');

        // Send welcome email
        $adminEmail = config('installer.admin_email');
        if ($adminEmail) {
            Mail::to($adminEmail)->send(new InstallationComplete($event));
        }

        // Create initial data
        // Set up cron jobs
        // Clear caches
    }
}
```

## Emitting events from your step

Use the `EmitsInstallerEvents` trait in your step.

```php
namespace YourVendor\YourPlugin\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Olakunlevpn\Installer\Concerns\EmitsInstallerEvents;

class LicenseStep extends BaseStep
{
    use EmitsInstallerEvents;

    protected function mount(): void
    {
        $this->emitStepStarted('license', [
            'timestamp' => now(),
        ]);
    }

    protected function execute(): void
    {
        $this->storeInSession('license', $this->formData);

        $this->emitStepCompleted('license', [
            'license_key' => $this->formData['license_key'],
            'verified' => true,
        ]);
    }

    protected function validateStep(): void
    {
        try {
            $this->validate();
            $this->verifyLicense();
        } catch (\Exception $e) {
            $this->emitStepFailed($e, 'license', [
                'license_key' => $this->formData['license_key'] ?? null,
            ]);

            throw $e;
        }
    }
}
```

## Custom events

Create your own events for specific actions.

**Your event:**

```php
namespace YourVendor\YourPlugin\Events;

use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class LicenseVerified
{
    use Dispatchable, SerializesModels;

    public function __construct(
        public string $licenseKey,
        public string $email,
        public array $features
    ) {}
}
```

**Fire from your step:**

```php
use YourVendor\YourPlugin\Events\LicenseVerified;

protected function execute(): void
{
    $this->storeInSession('license', $this->formData);

    // Fire custom event
    event(new LicenseVerified(
        $this->formData['license_key'],
        $this->formData['email'],
        $this->formData['features']
    ));
}
```

**Or use the trait helper:**

```php
protected function execute(): void
{
    $this->storeInSession('license', $this->formData);

    $this->emitCustomEvent(LicenseVerified::class, [
        $this->formData['license_key'],
        $this->formData['email'],
        $this->formData['features'],
    ]);
}
```

**Listen to your event:**

```php
protected $listen = [
    LicenseVerified::class => [
        UpdateLicenseServer::class,
        EnableFeatures::class,
    ],
];
```

## Complete example

**Event listener that does post-installation setup:**

```php
namespace App\Listeners;

use Olakunlevpn\Installer\Events\InstallationCompleted;
use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\File;

class PerformPostInstallation
{
    public function handle(InstallationCompleted $event): void
    {
        // Clear installer session data
        $this->clearInstallerSession();

        // Create admin user from session data
        $this->createAdminUser();

        // Set up application from collected data
        $this->configureApplication();

        // Update .env file
        $this->updateEnvironmentFile();

        // Clear and cache config
        Artisan::call('config:cache');
        Artisan::call('route:cache');
        Artisan::call('view:cache');
    }

    protected function clearInstallerSession(): void
    {
        $keys = array_keys(session()->all());

        foreach ($keys as $key) {
            if (str_starts_with($key, 'installer_')) {
                session()->forget($key);
            }
        }
    }

    protected function createAdminUser(): void
    {
        $accountData = session('installer_account');

        if ($accountData) {
            User::create([
                'name' => $accountData['name'],
                'email' => $accountData['email'],
                'password' => bcrypt($accountData['password']),
                'role' => 'admin',
            ]);
        }
    }

    protected function configureApplication(): void
    {
        $appData = session('installer_app');

        if ($appData) {
            // Update settings table
            Setting::updateOrCreate(
                ['key' => 'app_name'],
                ['value' => $appData['app_name']]
            );

            Setting::updateOrCreate(
                ['key' => 'app_url'],
                ['value' => $appData['app_url']]
            );
        }
    }

    protected function updateEnvironmentFile(): void
    {
        $envPath = base_path('.env');

        if (!File::exists($envPath)) {
            return;
        }

        $env = File::get($envPath);

        // Update APP_NAME
        $appData = session('installer_app');
        if ($appData) {
            $env = preg_replace(
                '/APP_NAME=.*/',
                'APP_NAME="' . $appData['app_name'] . '"',
                $env
            );
        }

        File::put($envPath, $env);
    }
}
```

**Register the listener:**

```php
use Olakunlevpn\Installer\Events\InstallationCompleted;
use App\Listeners\PerformPostInstallation;

protected $listen = [
    InstallationCompleted::class => [
        PerformPostInstallation::class,
    ],
];
```

## Event data access

All events provide access to relevant data:

```php
// StepStarted
$stepId = $event->stepId;
$customData = $event->data;

// StepCompleted
$step = $event->step; // The actual step instance
$stepId = $event->stepId;
$customData = $event->data;

// StepFailed
$stepId = $event->stepId;
$exception = $event->exception;
$message = $event->exception->getMessage();
$customData = $event->data;

// InstallationCompleted
$installerId = $event->installerId;
$completedSteps = $event->completedSteps; // ['welcome', 'database', 'license', ...]
$duration = $event->duration; // Time in seconds
```

## Queued listeners

Make listeners run in the background:

```php
namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueue;
use Olakunlevpn\Installer\Events\InstallationCompleted;

class SendInstallationReport implements ShouldQueue
{
    public function handle(InstallationCompleted $event): void
    {
        // Runs asynchronously
        Mail::to('admin@example.com')->send(
            new InstallationReport($event)
        );
    }
}
```

## Next steps

- Customize installer appearance: [Customization](customization.md)
- See complete plugin examples: [Complete Plugin Example](../examples/complete-plugin.md)
- Review helper traits: [Helper Traits](helper-traits.md)
