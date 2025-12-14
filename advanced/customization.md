---
title: "Customization"
weight: 4
---

# Customization

Customize the installer's appearance, branding, and behavior.

## Publishing the config

```bash
php artisan vendor:publish --provider="Olakunlevpn\Installer\InstallerServiceProvider"
```

This creates `config/installer.php` with all customization options.

## Branding

### Application name

```php
'brandName' => env('APP_NAME', 'My Application'),
```

Appears in the header and page titles.

### Logo

```php
'brandLogo' => '/images/logo.svg',
```

URL to your logo image. Shows in the installer header. Leave null to use text-only branding.

**Example with logo:**

```php
'brandLogo' => asset('images/company-logo.png'),
```

## Colors

Customize the color scheme for dark mode.

```php
'colors' => [
    'primary' => '#272531',      // Primary brand color
    'secondary' => '#4a5568',    // Secondary elements
    'success' => '#10b981',      // Success messages
    'error' => '#ef4444',        // Error messages
    'warning' => '#f59e0b',      // Warning messages
    'info' => '#3b82f6',         // Info messages
],
```

All installer components use these colors automatically.

## Dark mode

```php
'darkMode' => true,
```

Set to `false` to use light mode instead. The installer is optimized for dark mode.

## Steps configuration

### Registering steps

```php
'steps' => [
    \Olakunlevpn\Installer\Steps\WelcomeStep::class,
    \Olakunlevpn\Installer\Steps\RequirementsStep::class,
    \Olakunlevpn\Installer\Steps\PermissionsStep::class,
    \Olakunlevpn\Installer\Steps\EnvironmentStep::class,
    \YourVendor\YourPlugin\Steps\LicenseStep::class, // Your plugin step
    \Olakunlevpn\Installer\Steps\AccountStep::class,
    \Olakunlevpn\Installer\Steps\InstallingStep::class,
    \Olakunlevpn\Installer\Steps\CompletedStep::class,
],
```

Steps appear in the order listed. Plugin steps are usually added via service providers, not here.

### Step order

Plugin steps use `step_position` from their own config:

`config/installer-license.php`:

```php
'step_position' => env('LICENSE_STEP_POSITION', 5),
```

Common positions:
- 1-3: Early steps (requirements, permissions)
- 4-6: Configuration (database, environment)
- 7-9: Features (license, integrations)
- 10+: Finalization (admin account, completion)

## Requirements

### PHP version

```php
'requirements' => [
    'php' => '8.2.0',
],
```

### PHP extensions

```php
'requirements' => [
    'extensions' => [
        'php' => [
            'openssl',
            'pdo',
            'mbstring',
            'tokenizer',
            'xml',
            'ctype',
            'json',
            'bcmath',
        ],
    ],
],
```

Add extensions your application requires.

## Permissions

File and directory permission requirements.

```php
'permissions' => [
    storage_path() => '775',
    storage_path('app') => '775',
    storage_path('framework') => '775',
    storage_path('logs') => '775',
    base_path('.env') => '644',
    base_path('bootstrap/cache') => '775',
],
```

Format: `'path' => 'required_permissions'`

## Middleware

```php
'middleware' => ['web'],
```

Middleware applied to installer routes. Keep `web` for session support.

Add custom middleware:

```php
'middleware' => ['web', 'throttle:10,1'],
```

## Completion settings

### Redirect after installation

```php
'completed' => [
    'redirectTo' => '/admin/dashboard',
],
```

Where to redirect after installation completes.

### Next steps

Show post-installation instructions.

```php
'completed' => [
    'nextSteps' => [
        [
            'title' => 'installer::installer.set_up_cron',
            'description' => 'installer::installer.set_up_cron_description',
            'icon' => 'clock',
        ],
        [
            'title' => 'installer::installer.configure_email',
            'description' => 'installer::installer.configure_email_description',
            'icon' => 'mail',
        ],
    ],
],
```

These appear on the completion page as checklist items.

## Development mode

```php
'development' => env('INSTALLER_DEV', false),
```

When true:
- Shows detailed error messages
- Skips some validations
- Allows re-running installer

Set in `.env`:

```
INSTALLER_DEV=true
```

Never enable in production.

## Custom views

Override any installer view.

**Publish views:**

```bash
php artisan vendor:publish --tag=laravel-installer-views
```

Views are copied to `resources/views/vendor/installer/`.

**Override a specific view:**

Create `resources/views/vendor/installer/livewire/welcome.blade.php`:

```blade
<div>
    <h1>Welcome to {{ config('app.name') }} Installation</h1>
    <p>Your custom welcome message here.</p>

    <!-- Keep the original form structure -->
    <form wire:submit.prevent="submit">
        <x-installer::button type="submit">
            Get Started
        </x-installer::button>
    </form>
</div>
```

The installer uses your view instead of the default.

## Custom components

Create your own components using installer styles.

**Your component:**

`resources/views/components/custom-card.blade.php`:

```blade
<div class="rounded-lg bg-gray-800 border border-gray-700 p-6">
    <h3 class="text-lg font-semibold text-white mb-4">
        {{ $title }}
    </h3>

    <div class="text-gray-300">
        {{ $slot }}
    </div>
</div>
```

**Use in your step view:**

```blade
<x-custom-card title="License Information">
    <p>Enter your license details below.</p>
</x-custom-card>
```

## Custom translations

Publish and modify translation files.

**Publish translations:**

```bash
php artisan vendor:publish --tag=laravel-installer-translations
```

**Edit translations:**

`lang/en/installer.php`:

```php
return [
    'welcome_title' => 'Welcome to Installation',
    'welcome_description' => 'We will guide you through the setup process.',
    // ... your custom messages
];
```

**Add new language:**

Create `lang/es/installer.php` for Spanish:

```php
return [
    'welcome_title' => 'Bienvenido a la instalación',
    'welcome_description' => 'Te guiaremos a través del proceso de configuración.',
];
```

Set locale in config:

```php
'locale' => 'es',
```

## Custom CSS

Add your own styles.

**Create stylesheet:**

`public/css/installer-custom.css`:

```css
.installer-header {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.installer-card {
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.3);
}
```

**Include in layout:**

Override `resources/views/vendor/installer/layouts/app.blade.php`:

```blade
<head>
    <!-- ... existing head content -->
    <link rel="stylesheet" href="{{ asset('css/installer-custom.css') }}">
</head>
```

## Complete customization example

`config/installer.php`:

```php
return [
    // Branding
    'brandName' => env('APP_NAME', 'Acme CRM'),
    'brandLogo' => asset('images/acme-logo.svg'),

    // Colors
    'colors' => [
        'primary' => '#6366f1',    // Indigo
        'secondary' => '#8b5cf6',  // Purple
        'success' => '#10b981',    // Green
        'error' => '#ef4444',      // Red
        'warning' => '#f59e0b',    // Amber
        'info' => '#3b82f6',       // Blue
    ],

    // Dark mode only
    'darkMode' => true,

    // Steps
    'steps' => [
        \Olakunlevpn\Installer\Steps\WelcomeStep::class,
        \Olakunlevpn\Installer\Steps\RequirementsStep::class,
        \Olakunlevpn\Installer\Steps\PermissionsStep::class,
        \Olakunlevpn\Installer\Steps\EnvironmentStep::class,
        \Acme\License\Steps\LicenseVerificationStep::class,
        \Olakunlevpn\Installer\Steps\AccountStep::class,
        \Olakunlevpn\Installer\Steps\InstallingStep::class,
        \Olakunlevpn\Installer\Steps\CompletedStep::class,
    ],

    // Requirements
    'requirements' => [
        'php' => '8.2.0',
        'extensions' => [
            'php' => [
                'openssl',
                'pdo',
                'pdo_mysql',
                'mbstring',
                'tokenizer',
                'xml',
                'ctype',
                'json',
                'bcmath',
                'gd',
            ],
        ],
    ],

    // Permissions
    'permissions' => [
        storage_path() => '775',
        storage_path('app') => '775',
        storage_path('framework') => '775',
        storage_path('framework/cache') => '775',
        storage_path('framework/sessions') => '775',
        storage_path('framework/views') => '775',
        storage_path('logs') => '775',
        base_path('.env') => '644',
        base_path('bootstrap/cache') => '775',
    ],

    // Completion
    'completed' => [
        'redirectTo' => '/admin/dashboard',
        'nextSteps' => [
            [
                'title' => 'installer::installer.configure_cron',
                'description' => 'installer::installer.configure_cron_desc',
                'icon' => 'clock',
            ],
            [
                'title' => 'installer::installer.setup_email',
                'description' => 'installer::installer.setup_email_desc',
                'icon' => 'mail',
            ],
            [
                'title' => 'installer::installer.review_settings',
                'description' => 'installer::installer.review_settings_desc',
                'icon' => 'cog',
            ],
        ],
    ],

    // Development
    'development' => env('INSTALLER_DEV', false),

    // Middleware
    'middleware' => ['web', 'throttle:10,1'],
];
```

## Environment variables

Control settings via `.env`:

```
APP_NAME="Acme CRM"
INSTALLER_DEV=false
LICENSE_STEP_POSITION=5
```

Reference in config:

```php
'brandName' => env('APP_NAME', 'Default Name'),
'development' => env('INSTALLER_DEV', false),
```

## Plugin customization

Plugins can provide their own customization options.

`config/installer-license.php`:

```php
return [
    'step_position' => env('LICENSE_STEP_POSITION', 5),
    'api_url' => env('LICENSE_API_URL', 'https://api.example.com'),
    'timeout' => env('LICENSE_API_TIMEOUT', 10),

    'form_fields' => [
        [
            'name' => 'license_key',
            'type' => 'text',
            'label' => 'license::license.key_label',
            'required' => true,
            'validation' => 'required|string|min:20',
        ],
    ],
];
```

Users customize by editing the config or setting environment variables.

## Next steps

- See complete plugin example: [Complete Plugin Example](../examples/complete-plugin.md)
- Review component documentation: [Components](../components/form-components.md)
- Understand the event system: [Events System](events.md)
