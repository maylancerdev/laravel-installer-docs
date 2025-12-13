# Database Integration Examples

Working with the database during installation.

## The timing problem

During installation, the database isn't always available:
- Early steps: No database connection yet
- Middle steps: Connection configured but tables don't exist
- Later steps: Migrations run, tables available

Always check database availability before queries.

## Checking database availability

```php
protected function isDatabaseReady(): bool
{
    try {
        DB::connection()->getPdo();
        return true;
    } catch (\Exception $e) {
        return false;
    }
}

protected function areTablesReady(): bool
{
    try {
        // Try to query a table
        DB::table('settings')->exists();
        return true;
    } catch (\Exception $e) {
        return false;
    }
}
```

## Session-first approach

Always save to session. Save to database if available.

```php
protected function execute(): void
{
    // Always save to session first
    $this->storeStepData('app', $this->formData);

    // Try to save to database if ready
    if ($this->isDatabaseReady() && $this->areTablesReady()) {
        $this->saveToDatabase();
    }
}

protected function saveToDatabase(): void
{
    Setting::updateOrCreate(
        ['key' => 'app_name'],
        ['value' => $this->formData['app_name']]
    );

    Setting::updateOrCreate(
        ['key' => 'app_url'],
        ['value' => $this->formData['app_url']]
    );
}
```

## Creating admin user

Create admin user after migrations run.

```php
namespace App\Installer\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Olakunlevpn\Installer\Concerns\HasFormData;
use Olakunlevpn\Installer\Concerns\ManagesSessionData;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class AccountStep extends BaseStep
{
    use HasFormData;
    use ManagesSessionData;

    protected function mount(): void
    {
        $existing = $this->retrieveStepData('account');

        if ($existing) {
            $this->formData = $existing;
        }
    }

    protected function rules(): array
    {
        return [
            'formData.name' => 'required|string|max:255',
            'formData.email' => 'required|email',
            'formData.password' => 'required|min:8',
            'formData.password_confirmation' => 'required|same:formData.password',
        ];
    }

    protected function execute(): void
    {
        // Save to session
        $this->storeStepData('account', [
            'name' => $this->formData['name'],
            'email' => $this->formData['email'],
            'password' => $this->formData['password'],
        ]);

        // Create user if database is ready
        if ($this->isDatabaseReady() && $this->areTablesReady()) {
            $this->createAdminUser();
        }
    }

    protected function createAdminUser(): void
    {
        User::updateOrCreate(
            ['email' => $this->formData['email']],
            [
                'name' => $this->formData['name'],
                'password' => Hash::make($this->formData['password']),
                'role' => 'admin',
                'email_verified_at' => now(),
            ]
        );
    }

    protected function isDatabaseReady(): bool
    {
        try {
            DB::connection()->getPdo();
            return true;
        } catch (\Exception $e) {
            return false;
        }
    }

    protected function areTablesReady(): bool
    {
        try {
            DB::table('users')->exists();
            return true;
        } catch (\Exception $e) {
            return false;
        }
    }
}
```

## Seeding initial data

Insert default settings after installation.

```php
namespace App\Installer\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use App\Models\Setting;
use Illuminate\Support\Facades\DB;

class InstallingStep extends BaseStep
{
    protected function mount(): void
    {
        $this->runInstallation();
    }

    protected function runInstallation(): void
    {
        try {
            // Run migrations
            Artisan::call('migrate', ['--force' => true]);

            // Seed initial data
            $this->seedInitialData();

            // Move session data to database
            $this->migrateSessionToDatabase();

            $this->redirect(route('installer.completed'));
        } catch (\Exception $e) {
            $this->errorMessage = $e->getMessage();
            Log::error('Installation failed', ['error' => $e->getMessage()]);
        }
    }

    protected function seedInitialData(): void
    {
        // Default settings
        $defaults = [
            'app_timezone' => 'UTC',
            'date_format' => 'Y-m-d',
            'time_format' => 'H:i:s',
            'items_per_page' => 15,
            'maintenance_mode' => false,
        ];

        foreach ($defaults as $key => $value) {
            Setting::firstOrCreate(
                ['key' => $key],
                ['value' => is_bool($value) ? (int)$value : $value]
            );
        }
    }

    protected function migrateSessionToDatabase(): void
    {
        // App settings
        $appData = session('installer_step_app');
        if ($appData) {
            Setting::updateOrCreate(
                ['key' => 'app_name'],
                ['value' => $appData['app_name']]
            );

            Setting::updateOrCreate(
                ['key' => 'app_url'],
                ['value' => $appData['app_url']]
            );
        }

        // License data
        $licenseData = session('installer_step_license');
        if ($licenseData) {
            Setting::updateOrCreate(
                ['key' => 'license_key'],
                ['value' => $licenseData['purchase_code']]
            );

            Setting::updateOrCreate(
                ['key' => 'license_email'],
                ['value' => $licenseData['email']]
            );
        }
    }
}
```

## Updating .env file

Write database credentials to .env.

```php
namespace App\Installer\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Illuminate\Support\Facades\File;

class DatabaseStep extends BaseStep
{
    protected function execute(): void
    {
        // Save to session
        $this->storeStepData('database', $this->formData);

        // Update .env file
        $this->updateEnvFile();
    }

    protected function updateEnvFile(): void
    {
        $envPath = base_path('.env');

        if (!File::exists($envPath)) {
            // Create from .env.example
            File::copy(base_path('.env.example'), $envPath);
        }

        $env = File::get($envPath);

        $replacements = [
            'DB_CONNECTION' => $this->formData['driver'],
            'DB_HOST' => $this->formData['host'],
            'DB_PORT' => $this->formData['port'],
            'DB_DATABASE' => $this->formData['database'],
            'DB_USERNAME' => $this->formData['username'],
            'DB_PASSWORD' => $this->formData['password'],
        ];

        foreach ($replacements as $key => $value) {
            // Escape special characters in password
            if ($key === 'DB_PASSWORD') {
                $value = $this->escapeEnvValue($value);
            }

            // Replace existing value or append
            if (preg_match("/^{$key}=/m", $env)) {
                $env = preg_replace(
                    "/^{$key}=.*/m",
                    "{$key}={$value}",
                    $env
                );
            } else {
                $env .= "\n{$key}={$value}";
            }
        }

        File::put($envPath, $env);

        // Reload config
        Artisan::call('config:clear');
    }

    protected function escapeEnvValue(string $value): string
    {
        // Wrap in quotes if contains special characters
        if (preg_match('/[^\w.]/', $value)) {
            return '"' . addslashes($value) . '"';
        }

        return $value;
    }
}
```

## Migration verification

Check if required columns exist before installation.

```php
namespace App\Installer\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Olakunlevpn\Installer\Support\MigrationParser;

class RequirementsStep extends BaseStep
{
    protected function validateStep(): void
    {
        $this->validate();

        if (!$this->verifyMigrations()) {
            $this->errorMessage = __('requirements::requirements.migration_check_failed');
            return;
        }
    }

    protected function verifyMigrations(): bool
    {
        $parser = new MigrationParser();

        // Check settings table structure
        if (!$parser->hasMigration('settings')) {
            $this->errorMessage = 'Settings table migration not found';
            return false;
        }

        $required = ['key', 'value', 'group'];
        $missing = $parser->getMissingColumns('settings', $required);

        if (!empty($missing)) {
            $this->errorMessage = 'Settings table missing columns: ' . implode(', ', $missing);
            return false;
        }

        // Check users table
        if (!$parser->hasMigration('users')) {
            $this->errorMessage = 'Users table migration not found';
            return false;
        }

        $requiredUserColumns = ['name', 'email', 'password'];
        $missingUserColumns = $parser->getMissingColumns('users', $requiredUserColumns);

        if (!empty($missingUserColumns)) {
            $this->errorMessage = 'Users table missing columns: ' . implode(', ', $missingUserColumns);
            return false;
        }

        return true;
    }
}
```

## Post-installation data migration

Move all session data to database after installation completes.

```php
namespace App\Listeners;

use Olakunlevpn\Installer\Events\InstallationCompleted;
use App\Models\Setting;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class MigrateInstallerData
{
    public function handle(InstallationCompleted $event): void
    {
        $this->migrateAppSettings();
        $this->migrateAdminAccount();
        $this->migrateLicenseData();
        $this->migrateMailSettings();

        // Clear installer session
        $this->clearInstallerSession();
    }

    protected function migrateAppSettings(): void
    {
        $data = session('installer_step_app');

        if ($data) {
            Setting::updateOrCreate(
                ['key' => 'app_name'],
                ['value' => $data['app_name'], 'group' => 'app']
            );

            Setting::updateOrCreate(
                ['key' => 'app_url'],
                ['value' => $data['app_url'], 'group' => 'app']
            );
        }
    }

    protected function migrateAdminAccount(): void
    {
        $data = session('installer_step_account');

        if ($data) {
            User::updateOrCreate(
                ['email' => $data['email']],
                [
                    'name' => $data['name'],
                    'password' => Hash::make($data['password']),
                    'role' => 'admin',
                    'email_verified_at' => now(),
                ]
            );
        }
    }

    protected function migrateLicenseData(): void
    {
        $data = session('installer_step_license');

        if ($data) {
            Setting::updateOrCreate(
                ['key' => 'license_key'],
                ['value' => $data['purchase_code'], 'group' => 'license']
            );

            Setting::updateOrCreate(
                ['key' => 'license_email'],
                ['value' => $data['email'], 'group' => 'license']
            );

            Setting::updateOrCreate(
                ['key' => 'license_verified_at'],
                ['value' => now(), 'group' => 'license']
            );
        }
    }

    protected function migrateMailSettings(): void
    {
        $data = session('installer_step_mail');

        if ($data) {
            $mailSettings = [
                'mail_driver' => $data['driver'],
                'mail_host' => $data['host'],
                'mail_port' => $data['port'],
                'mail_username' => $data['username'],
                'mail_password' => $data['password'],
                'mail_encryption' => $data['encryption'],
            ];

            foreach ($mailSettings as $key => $value) {
                Setting::updateOrCreate(
                    ['key' => $key],
                    ['value' => $value, 'group' => 'mail']
                );
            }
        }
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
}
```

Register listener:

```php
protected $listen = [
    InstallationCompleted::class => [
        MigrateInstallerData::class,
    ],
];
```

## Checking table existence

Before querying a table, verify it exists.

```php
use Illuminate\Support\Facades\Schema;

protected function execute(): void
{
    $this->storeStepData('features', $this->formData);

    if (Schema::hasTable('feature_flags')) {
        foreach ($this->formData['features'] as $feature => $enabled) {
            DB::table('feature_flags')->updateOrInsert(
                ['name' => $feature],
                ['enabled' => $enabled, 'updated_at' => now()]
            );
        }
    }
}
```

## Summary

Database integration best practices:
- Always save to session first
- Check database availability before queries
- Check table existence before inserts
- Update .env for persistent config
- Seed default data after migrations
- Migrate session data after installation
- Use event listeners for cleanup
- Handle exceptions gracefully

The session-first approach ensures data isn't lost if database isn't ready.
