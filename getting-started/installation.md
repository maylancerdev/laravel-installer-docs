# Installation

Add the installer to your Laravel app in five minutes.

## Requirements

- **PHP 8.2+**
- **Laravel 12+**
- **Livewire 3+**

Fresh Laravel app recommended - no existing database tables or `.env` configuration.

The installer uses **Tailwind CSS via CDN** - no build step required.

## Install via Composer

```bash
composer require olakunlevpn/laravel-installer
```

Auto-discovers its service provider.

## Publish Configuration

```bash
php artisan vendor:publish --provider="Olakunlevpn\Installer\InstallerServiceProvider"
```

Creates `config/installer.php`.

## Configure

### Route

```php
// config/installer.php
'path' => 'install', // Access at /install
```

### Middleware

```php
'middleware' => ['web'], // No auth required
```

### Steps

Default steps pre-configured:

```php
'steps' => [
    \Olakunlevpn\Installer\Steps\WelcomeStep::class,
    \Olakunlevpn\Installer\Steps\RequirementsStep::class,
    \Olakunlevpn\Installer\Steps\PermissionsStep::class,
    \Olakunlevpn\Installer\Steps\EnvironmentStep::class,
    \Olakunlevpn\Installer\Steps\InstallingStep::class,
    \Olakunlevpn\Installer\Steps\CompletedStep::class,
],
```

Plugin packages auto-register their steps.

## Install Plugins (Optional)

```bash
# Custom Welcome page
composer require olakunlevpn/laravel-installer-welcome

# License verification
composer require olakunlevpn/laravel-installer-license

# Account setup
composer require olakunlevpn/laravel-installer-account
```

See [packages documentation](../packages/index.md).

## Launch

Visit `/install` in your browser.

No database connection needed yet - requirements checked first.

## Next Steps

- [Quick start walkthrough](quick-start.md)
- [Customize appearance](../advanced/customization.md)
- [Create custom steps](../creating-plugins/plugin-structure.md)
