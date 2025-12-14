---
title: "Plugin Structure"
weight: 1
---

# Plugin Structure

Installer plugins are Laravel packages that add custom steps to the installation wizard.

## Package anatomy

A minimal plugin needs:

```
packages/your-plugin/
├── config/
│   └── installer_yourplugin.php    # Configuration
├── lang/
│   └── en/
│       └── yourplugin.php           # Translations
├── resources/
│   └── views/
│       └── livewire/
│           └── steps/
│               └── your-step.blade.php
├── src/
│   ├── Steps/
│   │   └── YourStep.php             # Step class
│   └── YourPluginServiceProvider.php
└── composer.json
```

That's it. No migrations, routes, or controllers needed (unless your step requires them).

## Quick start

Create a new package directory:

```bash
mkdir -p packages/your-plugin/{config,lang/en,resources/views/livewire/steps,src/Steps}
```

## Configuration file

`config/installer_yourplugin.php`:

```php
<?php

return [
    'step_position' => env('YOURPLUGIN_STEP_POSITION', 5),

    'form_fields' => [
        [
            'name' => 'api_key',
            'type' => 'text',
            'label' => 'yourplugin::yourplugin.api_key',
            'placeholder' => 'yourplugin::yourplugin.api_key_placeholder',
            'required' => true,
            'validation' => 'required|string|min:20',
            'store_in_db' => true,
            'column' => 'api_key',
        ],
    ],
];
```

Lower `step_position` numbers appear earlier in the wizard.

## Translation file

`lang/en/yourplugin.php`:

```php
<?php

return [
    'api_key' => 'API Key',
    'api_key_placeholder' => 'Enter your API key',
    'submit' => 'Verify & Continue',
    'verifying' => 'Verifying...',
    'success' => 'API key verified successfully!',
    'help_title' => 'Need help?',
    'help_text' => 'Find your API key in your account dashboard.',
];
```

Always use translation keys - never hardcode text.

## Service provider

`src/YourPluginServiceProvider.php`:

```php
<?php

namespace YourVendor\YourPlugin;

use Illuminate\Support\ServiceProvider;
use Olakunlevpn\Installer\Installer;
use YourVendor\YourPlugin\Steps\YourStep;

class YourPluginServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        // Register step
        $installer = $this->app->make(Installer::class);
        $installer->addStep(YourStep::class);

        // Publish config
        $this->publishes([
            __DIR__.'/../config/installer_yourplugin.php' =>
                config_path('installer_yourplugin.php'),
        ], 'installer-yourplugin-config');

        // Load views
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'yourplugin');

        // Load translations
        $this->loadTranslationsFrom(__DIR__.'/../lang', 'yourplugin');

        // Merge config
        $this->mergeConfigFrom(
            __DIR__.'/../config/installer_yourplugin.php',
            'installer-yourplugin'
        );
    }
}
```

The installer automatically discovers and loads your step.

## Composer.json

```json
{
    "name": "yourvendor/installer-yourplugin",
    "autoload": {
        "psr-4": {
            "YourVendor\\YourPlugin\\": "src/"
        }
    },
    "extra": {
        "laravel": {
            "providers": [
                "YourVendor\\YourPlugin\\YourPluginServiceProvider"
            ]
        }
    },
    "require": {
        "olakunlevpn/laravel-installer": "^1.0"
    }
}
```

Laravel auto-discovers the service provider via the `extra` section.

## Installation

Users install your plugin via Composer:

```bash
composer require yourvendor/installer-yourplugin
```

Your step automatically appears in the installer at the configured position. No manual configuration required.

## Next steps

- Create your first step: [Your First Step](your-first-step.md)
- Understand the lifecycle: [Step Lifecycle](step-lifecycle.md)
- Customize views and assets: [Views and Assets](views-and-assets.md)
- Follow best practices: [Best Practices](best-practices.md)
