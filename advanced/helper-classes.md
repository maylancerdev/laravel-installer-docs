---
title: "Helper Classes"
weight: 2
---

# Helper Classes

Four utility classes for working with installer configuration, migrations, validation, and installation management.

## InstallerConfig

Static class for accessing installer configuration.

### get()

Get any installer config value using dot notation.

```php
use Olakunlevpn\Installer\Support\InstallerConfig;

$brandName = InstallerConfig::get('brandName', 'My App');
$steps = InstallerConfig::get('steps', []);
```

### stepPosition()

Get step position from plugin config.

```php
$position = InstallerConfig::stepPosition('installer-license', 50);
// Returns config('installer-license.step_position', 50)
```

### formFields()

Get form fields from plugin config.

```php
$fields = InstallerConfig::formFields('installer-database');
// Returns config('installer-database.form_fields', [])
```

### steps()

Get all registered steps.

```php
$allSteps = InstallerConfig::steps();
// Returns config('installer.steps', [])
```

### Brand methods

```php
$brandName = InstallerConfig::brandName();
// Default: 'Laravel Application'

$brandLogo = InstallerConfig::brandLogo();
// Default: null
```

### Color methods

```php
$primary = InstallerConfig::primaryColor();
// Default: '#272531'

$colors = InstallerConfig::colors();
// Returns all color configuration
```

### Mode checks

```php
if (InstallerConfig::isDarkMode()) {
    // Dark mode is enabled
}

if (InstallerConfig::isDevelopment()) {
    // Development mode active
}
```

### Requirements methods

```php
$phpVersion = InstallerConfig::requiredPhpVersion();
// Returns: '8.2'

$extensions = InstallerConfig::requiredExtensions();
// Returns: ['mbstring', 'openssl', ...]

$allRequirements = InstallerConfig::phpRequirements();
// Returns complete requirements config
```

### Permissions

```php
$permissions = InstallerConfig::permissions();
// Returns: ['/storage' => '775', ...]
```

### Completion settings

```php
$redirectUrl = InstallerConfig::completionRedirect();
// Default: '/admin'

$nextSteps = InstallerConfig::nextSteps();
// Returns post-installation steps
```

### has() and set()

```php
if (InstallerConfig::has('customKey')) {
    // Config key exists
}

InstallerConfig::set('customKey', 'value');
// Set at runtime (not persisted)
```

### Package config methods

```php
$value = InstallerConfig::getPackageConfig('installer-license', 'api_url');
// Returns: config('installer-license.api_url')

if (InstallerConfig::hasPackageConfig('installer-license')) {
    // Package config exists
}
```

## MigrationParser

Parse migration files to understand database structure.

### getColumns()

Get all columns for a table from migration files.

```php
use Olakunlevpn\Installer\Support\MigrationParser;

$parser = new MigrationParser();
$columns = $parser->getColumns('users');
// Returns: ['id', 'name', 'email', 'password', 'created_at', 'updated_at']
```

### getMissingColumns()

Check which required columns are missing.

```php
$required = ['id', 'name', 'email', 'license_key'];
$missing = $parser->getMissingColumns('users', $required);

if (!empty($missing)) {
    // Some columns don't exist in the migration
}
```

### hasMigration()

Check if migration file exists for a table.

```php
if ($parser->hasMigration('settings')) {
    // Settings table migration exists
}
```

### hasColumn()

Check if specific column exists.

```php
if ($parser->hasColumn('users', 'license_key')) {
    // Column exists in migration
}
```

### getAllTables()

Get all tables from migration files.

```php
$tables = $parser->getAllTables();
// Returns: ['users', 'settings', 'posts', ...]
```

### Custom migrations path

```php
$parser = new MigrationParser('/custom/path/migrations');

// Or set after construction
$parser->setMigrationsPath('/custom/path/migrations');
```

### Example: Verify migration structure

```php
protected function validateStep(): void
{
    $this->validate();

    $parser = new MigrationParser();

    // Check if table exists
    if (!$parser->hasMigration('settings')) {
        $this->errorMessage = 'Settings table migration not found';
        return;
    }

    // Check required columns
    $required = ['key', 'value', 'group'];
    $missing = $parser->getMissingColumns('settings', $required);

    if (!empty($missing)) {
        $this->errorMessage = 'Missing columns: ' . implode(', ', $missing);
        return;
    }
}
```

## StepValidator

Validate steps and their dependencies.

### validate()

Validate a single step.

```php
use Olakunlevpn\Installer\Support\StepValidator;

$validator = new StepValidator();
$step = new LicenseStep();

if ($validator->validate($step)) {
    // Step is valid
} else {
    $error = $validator->getError($step->getId());
}
```

### validateSteps()

Validate multiple steps.

```php
$steps = [
    new DatabaseStep(),
    new LicenseStep(),
    new AccountStep(),
];

if ($validator->validateSteps($steps)) {
    // All steps valid
} else {
    $errors = $validator->getErrors();
}
```

### Error methods

```php
// Get all errors
$errors = $validator->getErrors();
// Returns: ['database' => 'Connection failed', 'license' => 'Invalid key']

// Get error for specific step
$error = $validator->getError('database');

// Check if errors exist
if ($validator->hasErrors()) {
    // Has validation errors
}

// Get first error
$firstError = $validator->getFirstError();

// Clear errors
$validator->clearErrors();
```

### validateDependencies()

Check if step dependencies are met.

```php
$completedSteps = ['welcome', 'requirements', 'database'];
$step = new LicenseStep();

if ($validator->validateDependencies($step, $completedSteps)) {
    // All dependencies met
}
```

Your step must implement `getDependencies()`:

```php
public function getDependencies(): array
{
    return ['database', 'requirements'];
}
```

### Example: Pre-validate before submission

```php
public function submit()
{
    $validator = new StepValidator();

    if (!$validator->validate($this)) {
        $this->errorMessage = $validator->getFirstError();
        return;
    }

    $this->execute();
    $this->goToNextStep();
}
```

## InstallationManager

Manages the installation process.

### Basic usage

```php
use Olakunlevpn\Installer\Support\InstallationManager;

$manager = new InstallationManager(
    migrate: true,
    seed: true,
    storageLink: true,
    fresh: false
);

if ($manager->execute()) {
    $result = $manager->getResult();
    // Installation successful
} else {
    $result = $manager->getResult();
    // Installation failed
}
```

### Constructor parameters

```php
new InstallationManager(
    migrate: true,         // Run migrations
    seed: true,           // Run seeders
    storageLink: true,    // Create storage symlink
    fresh: false          // Use migrate:fresh instead of migrate
);
```

### execute()

Run the installation process.

```php
$success = $manager->execute();

if ($success) {
    $output = $manager->getResult()['output'];
    echo $output; // Migration and seeder output
}
```

Installation steps:
1. Clear all caches
2. Run migrations (if enabled)
3. Execute all step execute() methods
4. Create storage link (if enabled)

### generateKey()

Generate application key.

```php
$output = $manager->generateKey();
// Runs: php artisan key:generate --force
```

### rollback()

Rollback the last migration.

```php
if ($manager->rollback()) {
    echo "Rolled back successfully";
}
```

### getResult()

Get installation result.

```php
$result = $manager->getResult();

// Success result:
[
    'status' => 'success',
    'message' => 'Installation completed successfully',
    'output' => '...',
]

// Error result:
[
    'status' => 'error',
    'message' => 'Migration failed...',
    'output' => '...',
]
```

### isSuccessful()

Check if installation succeeded.

```php
if ($manager->isSuccessful()) {
    // Installation was successful
}
```

### Example: Installing step

```php
namespace Olakunlevpn\Installer\Steps;

use Olakunlevpn\Installer\Support\InstallationManager;

class InstallingStep extends BaseStep
{
    protected function mount(): void
    {
        $this->install();
    }

    protected function install(): void
    {
        $manager = new InstallationManager(
            migrate: true,
            seed: config('installer.seed', true),
            storageLink: true,
            fresh: false
        );

        if ($manager->execute()) {
            $this->storeInSession('installation_complete', true);
            $this->successMessage = __('installer::installer.installation_success');

            // Redirect to completion
            $this->redirect(route('installer.completed'));
        } else {
            $result = $manager->getResult();
            $this->errorMessage = $result['message'];

            Log::error('Installation failed', $result);
        }
    }

    public function render()
    {
        return view('installer::livewire.installing');
    }
}
```

## Using helper classes together

```php
use Olakunlevpn\Installer\Support\InstallerConfig;
use Olakunlevpn\Installer\Support\MigrationParser;
use Olakunlevpn\Installer\Support\StepValidator;

protected function validateStep(): void
{
    $this->validate();

    // Check PHP requirements
    $requiredPhp = InstallerConfig::requiredPhpVersion();
    if (!$this->checkPhpVersion($requiredPhp)) {
        $this->errorMessage = "PHP {$requiredPhp}+ required";
        return;
    }

    // Verify migration structure
    $parser = new MigrationParser();
    $required = ['key', 'value'];
    $missing = $parser->getMissingColumns('settings', $required);

    if (!empty($missing)) {
        $this->errorMessage = 'Migration missing columns: ' . implode(', ', $missing);
        return;
    }

    // Validate dependencies
    $validator = new StepValidator();
    $completedSteps = session('installer_completed_steps', []);

    if (!$validator->validateDependencies($this, $completedSteps)) {
        $this->errorMessage = $validator->getFirstError();
        return;
    }
}
```

## Next steps

- Learn about the event system: [Events System](events.md)
- Customize installer appearance: [Customization](customization.md)
- See complete examples: [Complete Plugin Example](../examples/complete-plugin.md)
