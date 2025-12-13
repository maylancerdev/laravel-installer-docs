# Best Practices

Guidelines for building reliable, maintainable installer plugins.

## Always use translation keys

Never hardcode text. Always use translation files.

**Bad:**

```blade
<x-installer::card title="Enter your license key">
```

**Good:**

```blade
<x-installer::card title="{{ __('license::license.card_title') }}">
```

This allows users to customize text and translate to other languages.

## Use helper traits

Don't reimplement session management or form data handling.

**Bad:**

```php
protected function execute(): void
{
    session()->put('mydata', $this->formData);
}
```

**Good:**

```php
use ManagesSessionData;

protected function execute(): void
{
    $this->saveToSession('mydata', $this->formData);
}
```

The trait handles edge cases and follows the installer's conventions.

## Validate everything

Never trust user input. Always validate in `validateStep()`.

**Bad:**

```php
protected function execute(): void
{
    // No validation, just save
    $this->saveToSession('database', $this->formData);
}
```

**Good:**

```php
protected function validateStep(): void
{
    $this->validate();

    // Additional validation
    if (!$this->canConnectToDatabase()) {
        $this->addError('formData.host', __('database::database.connection_failed'));
    }
}

protected function execute(): void
{
    // Validation passed, safe to save
    $this->saveToSession('database', $this->formData);
}
```

## Handle errors gracefully

Always catch exceptions and show user-friendly messages.

**Bad:**

```php
protected function execute(): void
{
    Http::post('https://api.example.com/verify', [
        'key' => $this->formData['api_key'],
    ]); // Throws if network fails
}
```

**Good:**

```php
protected function execute(): void
{
    try {
        $response = Http::timeout(10)->post('https://api.example.com/verify', [
            'key' => $this->formData['api_key'],
        ]);

        if ($response->successful()) {
            $this->successMessage = __('license::license.verified');
        } else {
            $this->errorMessage = __('license::license.invalid');
        }
    } catch (\Exception $e) {
        $this->errorMessage = __('license::license.connection_error');
        \Log::error('License verification failed', ['error' => $e->getMessage()]);
    }
}
```

## Check database availability

Database isn't available until after migrations run.

**Bad:**

```php
protected function execute(): void
{
    Setting::create(['key' => 'api_key', 'value' => $this->formData['api_key']]);
    // Fails if migrations haven't run yet
}
```

**Good:**

```php
protected function execute(): void
{
    $this->saveToSession('settings', $this->formData);

    try {
        DB::connection()->getPdo();
        // Database is ready
        Setting::updateOrCreate(
            ['key' => 'api_key'],
            ['value' => $this->formData['api_key']]
        );
    } catch (\Exception $e) {
        // Database not ready, session is enough
    }
}
```

## Keep steps focused

One step = one concern. Don't try to do everything in one step.

**Bad:**

A single step that collects:
- Database settings
- Mail settings
- App settings
- License key

**Good:**

Four separate steps:
- Database step
- Mail step
- Application step
- License step

Each step is simple, focused, and easy to validate.

## Provide sensible defaults

Set default values in `mount()` so users can skip fields.

```php
protected function mount(): void
{
    $this->loadFormData();

    $defaults = [
        'host' => 'localhost',
        'port' => 3306,
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
    ];

    foreach ($defaults as $key => $value) {
        if (!isset($this->formData[$key])) {
            $this->formData[$key] = $value;
        }
    }
}
```

## Use form.group component

Don't manually build label + input + error. Use the group component.

**Bad:**

```blade
<label for="api_key">{{ __('license::license.api_key') }}</label>
<input type="text" id="api_key" wire:model="formData.api_key" />
<x-installer::form.error name="formData.api_key" />
```

**Good:**

```blade
<x-installer::form.group :field="$field" wireModel="formData" />
```

The component handles everything consistently.

## Make config publishable

Always let users customize your plugin.

In your service provider:

```php
$this->publishes([
    __DIR__.'/../config/installer_yourplugin.php' => config_path('installer_yourplugin.php'),
], 'installer-yourplugin-config');
```

Users can then:

```bash
php artisan vendor:publish --tag=installer-yourplugin-config
```

And customize step position, fields, or validation.

## Publish views for customization

Let users override your plugin's views and even the main installer UI.

In your service provider:

```php
// Publish plugin views
$this->publishes([
    __DIR__.'/../resources/views' => resource_path('views/vendor/yourplugin'),
], 'installer-yourplugin-views');

// Publish installer overrides (optional)
$this->publishes([
    __DIR__.'/../resources/views/installer-overrides' => resource_path('views/vendor/installer'),
], 'yourplugin-installer-views');
```

Users can customize:
- Your plugin's step views
- Installer's core components (cards, buttons, forms)
- Main layout and branding

See [Views and Assets](views-and-assets.md) for details.

## Document your plugin

Create a README.md explaining:
- What your step does
- What data it collects
- How to configure it
- Any API keys or credentials needed

Example:

```markdown
# License Verification Step

Verifies user's license key during installation.

## Installation

composer require yourvendor/installer-license

## Configuration

Publish the config:

php artisan vendor:publish --tag=laravel-installer-license-config

Edit `config/installer_license.php` to customize.

## Environment Variables

- `LICENSE_STEP_POSITION` - Step position (default: 5)
- `LICENSE_API_URL` - Verification API URL

## License

MIT
```

## Follow Laravel conventions

Use Laravel's patterns and conventions:
- Translation files in `lang/`
- Config files in `config/`
- Views in `resources/views/`
- PSR-4 autoloading
- Package auto-discovery

## Test your plugin

Before releasing, test thoroughly:

1. Fresh installation
2. Validation errors display correctly
3. Session data persists across steps
4. Database integration works after migrations
5. Dark mode styling looks good
6. Translation keys resolve
7. Config customization works

## Version your config

When updating your plugin, version the config file:

```php
return [
    'version' => '1.0',
    'step_position' => env('YOURPLUGIN_STEP_POSITION', 5),
    // ... rest of config
];
```

This helps users know if they need to republish after updates.

## Use environment variables

Allow configuration via `.env`:

```php
return [
    'step_position' => env('LICENSE_STEP_POSITION', 5),
    'api_url' => env('LICENSE_API_URL', 'https://api.example.com'),
    'timeout' => env('LICENSE_API_TIMEOUT', 10),
];
```

Makes it easy to change settings per environment.

## Handle step position conflicts

If two plugins use the same position, they appear in registration order. Choose unique positions.

Common positions:
- 1-3: Early steps (requirements, permissions)
- 4-6: Configuration steps (database, mail)
- 7-9: Optional features (license, integrations)
- 10+: Final steps (admin user, completion)

## Clean up after yourself

If your step creates temporary files or makes test API calls, clean up:

```php
protected function execute(): void
{
    $tempFile = storage_path('installer/temp.txt');

    try {
        // Use temp file
    } finally {
        if (file_exists($tempFile)) {
            unlink($tempFile);
        }
    }
}
```

## Next steps

- See helper traits documentation: [Helper Traits](../advanced/helper-traits.md)
- View a complete working example: [Complete Plugin Example](../examples/complete-plugin.md)
- Learn about customization: [Customization](../advanced/customization.md)
