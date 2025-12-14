---
title: "Helper Traits"
weight: 1
---

# Helper Traits

Four traits provide common functionality for step classes. Use them instead of writing your own implementations.

## HasFormData

Manages form data and validation rules.

### initializeFormData()

Creates empty formData array from field configuration.

```php
use HasFormData;

protected function mount(): void
{
    $fields = config('installer-yourplugin.form_fields');
    $this->initializeFormData($fields);
}
```

### buildRulesFromFields()

Generates validation rules from field configuration.

```php
protected function rules(): array
{
    return $this->buildRulesFromFields(
        config('installer-yourplugin.form_fields')
    );
}
```

Automatically removes database-dependent rules (`unique`, `exists`) since database doesn't exist yet during installation.

### removeDbValidationRules()

Removes database rules from a validation string or array.

```php
// String format
$rules = 'required|email|unique:users,email';
$clean = $this->removeDbValidationRules($rules);
// Result: 'required|email'

// Array format
$rules = ['required', 'email', 'unique:users,email'];
$clean = $this->removeDbValidationRules($rules);
// Result: ['required', 'email']
```

Removes: `unique`, `exists`, `unique_translation`

### getDbData()

Extracts database-ready data from formData.

```php
protected function execute(): void
{
    $this->storeInSession('settings', $this->formData);

    // Only store fields marked with 'store_in_db' => true
    $dbData = $this->getDbData(
        config('installer-yourplugin.form_fields')
    );

    Setting::insert($dbData);
}
```

Field configuration:

```php
'form_fields' => [
    [
        'name' => 'api_key',
        'store_in_db' => true,
        'column' => 'key', // Optional, defaults to 'name'
    ],
    [
        'name' => 'temp_token',
        'store_in_db' => false, // Exclude from database
    ],
],
```

### getFormFields()

Override to provide custom fields.

```php
protected function getFormFields(): array
{
    return [
        ['name' => 'email', 'validation' => 'required|email'],
        ['name' => 'password', 'validation' => 'required|min:8'],
    ];
}
```

Most steps load from config instead:

```php
protected function getFormFields(): array
{
    return config('installer-yourplugin.form_fields', []);
}
```

## ManagesSessionData

Session storage for installation data.

### storeInSession()

Save data to session.

```php
use ManagesSessionData;

protected function execute(): void
{
    $this->storeInSession('database', $this->formData);
    // Stored as: session('installer_database')
}
```

All keys are automatically prefixed with `installer_` to avoid conflicts.

### retrieveFromSession()

Load data from session.

```php
protected function mount(): void
{
    $databaseConfig = $this->retrieveFromSession('database');

    if ($databaseConfig) {
        $this->formData = $databaseConfig;
    }
}
```

With default value:

```php
$port = $this->retrieveFromSession('database.port', 3306);
```

### clearFromSession()

Remove data from session.

```php
protected function execute(): void
{
    // Clear temporary data
    $this->clearFromSession('temp_verification');
}
```

### hasInSession()

Check if key exists in session.

```php
if ($this->hasInSession('license')) {
    // License was already verified
}
```

### storeStepData()

Save step-specific data.

```php
$this->storeStepData('license', [
    'key' => $this->formData['license_key'],
    'verified_at' => now(),
]);
// Stored as: session('installer_step_license')
```

### retrieveStepData()

Load step-specific data.

```php
$licenseData = $this->retrieveStepData('license');
```

### storeFormData() / retrieveFormData()

Convenience methods for form data.

```php
$this->storeFormData($this->formData, 'myform');
$data = $this->retrieveFormData('myform', []);
```

### clearAllInstallerData()

Remove all installer session data.

```php
protected function execute(): void
{
    // Installation complete, clean up
    $this->clearAllInstallerData();
}
```

Removes all keys starting with `installer_`.

### storeVerificationStatus() / isVerified()

Track verification state.

```php
protected function execute(): void
{
    $response = Http::post('https://api.example.com/verify', [...]);

    if ($response->successful()) {
        $this->storeVerificationStatus(true, 'license_verified');
    }
}

if ($this->isVerified('license_verified')) {
    // Already verified
}
```

## ValidatesRequirements

System requirements validation.

### checkPhpVersion()

Verify PHP version meets requirement.

```php
use ValidatesRequirements;

protected function validateStep(): void
{
    if (!$this->checkPhpVersion('8.2.0')) {
        $this->errorMessage = __('requirements::requirements.php_version_failed');
        return;
    }
}
```

### checkExtension()

Check if PHP extension is loaded.

```php
if (!$this->checkExtension('pdo_mysql')) {
    $this->errorMessage = 'MySQL extension is required';
}
```

### checkExtensions()

Check multiple extensions at once.

```php
$results = $this->checkExtensions([
    'mbstring',
    'openssl',
    'pdo',
    'tokenizer',
    'xml',
]);

// Returns: ['mbstring' => true, 'openssl' => true, ...]
```

### getMissingExtensions()

Get list of missing extensions.

```php
$required = ['mbstring', 'openssl', 'pdo', 'tokenizer'];
$missing = $this->getMissingExtensions($required);

if (!empty($missing)) {
    $this->errorMessage = 'Missing extensions: ' . implode(', ', $missing);
}
```

### checkFilePermissions()

Verify file/directory permissions.

```php
if (!$this->checkFilePermissions(storage_path(), '775')) {
    $this->errorMessage = 'Storage directory needs 775 permissions';
}
```

### checkPermissions()

Check multiple paths at once.

```php
$paths = [
    storage_path() => '775',
    base_path('.env') => '644',
    base_path('bootstrap/cache') => '775',
];

$results = $this->checkPermissions($paths);

foreach ($results as $path => $result) {
    if (!$result['passed']) {
        $this->addError('permissions', "Fix {$path}");
    }
}
```

Result format:

```php
[
    '/path/to/storage' => [
        'required' => '775',
        'current' => '755',
        'passed' => false,
    ],
]
```

### checkWritable() / checkReadable()

Quick permission checks.

```php
if (!$this->checkWritable(storage_path('logs'))) {
    $this->errorMessage = 'Logs directory must be writable';
}

if (!$this->checkReadable(base_path('.env'))) {
    $this->errorMessage = '.env file must be readable';
}
```

### getFailedRequirements()

Filter results to show only failures.

```php
$results = $this->checkExtensions(['mbstring', 'openssl', 'pdo']);
$failed = $this->getFailedRequirements($results);

// Returns only extensions that failed
```

## EmitsInstallerEvents

Event dispatching for step lifecycle.

### emitStepStarted()

Fire event when step begins.

```php
use EmitsInstallerEvents;

protected function mount(): void
{
    $this->emitStepStarted('license', [
        'user_email' => $this->formData['email'] ?? null,
    ]);
}
```

Dispatches `StepStarted` event.

### emitStepCompleted()

Fire event when step succeeds.

```php
protected function execute(): void
{
    $this->storeInSession('license', $this->formData);

    $this->emitStepCompleted('license', [
        'license_key' => $this->formData['license_key'],
    ]);
}
```

Dispatches `StepCompleted` event.

### emitStepFailed()

Fire event when step fails.

```php
protected function validateStep(): void
{
    try {
        $this->validate();
        $this->verifyLicenseKey();
    } catch (\Exception $e) {
        $this->emitStepFailed($e, 'license');
        $this->errorMessage = __('license::license.verification_failed');
    }
}
```

Dispatches `StepFailed` event.

### executeWithEvents()

Wrap code with automatic event firing.

```php
protected function execute(): void
{
    $result = $this->executeWithEvents(function () {
        // This code is wrapped with StepStarted and StepCompleted events
        $this->verifyLicense();
        $this->storeInSession('license', $this->formData);

        return ['verified' => true];
    }, 'license_verification');

    if ($result['verified']) {
        $this->successMessage = __('license::license.verified');
    }
}
```

Automatically catches exceptions and emits `StepFailed`.

### emitCustomEvent()

Dispatch your own events.

```php
protected function execute(): void
{
    $this->storeInSession('license', $this->formData);

    // Fire custom event
    $this->emitCustomEvent(LicenseVerified::class, [
        $this->formData['license_key'],
        now(),
    ]);
}
```

Your event class:

```php
namespace YourVendor\YourPlugin\Events;

class LicenseVerified
{
    public function __construct(
        public string $licenseKey,
        public \Carbon\Carbon $verifiedAt
    ) {}
}
```

## Using multiple traits

Most steps use several traits together:

```php
namespace YourVendor\YourPlugin\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Olakunlevpn\Installer\Concerns\HasFormData;
use Olakunlevpn\Installer\Concerns\ManagesSessionData;
use Olakunlevpn\Installer\Concerns\ValidatesRequirements;
use Olakunlevpn\Installer\Concerns\EmitsInstallerEvents;

class YourStep extends BaseStep
{
    use HasFormData;
    use ManagesSessionData;
    use ValidatesRequirements;
    use EmitsInstallerEvents;

    protected function mount(): void
    {
        $this->emitStepStarted();
        $this->initializeFormData(config('installer-yourplugin.form_fields'));
    }

    protected function rules(): array
    {
        return $this->buildRulesFromFields(
            config('installer-yourplugin.form_fields')
        );
    }

    protected function validateStep(): void
    {
        $this->validate();

        if (!$this->checkPhpVersion('8.2.0')) {
            $this->errorMessage = 'PHP 8.2+ required';
        }
    }

    protected function execute(): void
    {
        $this->storeInSession('yourplugin', $this->formData);
        $this->emitStepCompleted();
    }

    public function render()
    {
        return view('yourplugin::livewire.steps.your-step');
    }
}
```

## Next steps

- Learn about helper classes: [Helper Classes](helper-classes.md)
- Understand the event system: [Events System](events.md)
- See complete examples: [Complete Plugin Example](../examples/complete-plugin.md)
