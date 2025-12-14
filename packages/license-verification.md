---
title: "License Verification"
weight: 3
---

# License Verification

**Package:** `olakunlevpn/laravel-installer-license`

Adds a step that verifies purchase codes via your API.

![welcome](/images/license-verification.png)


## Install

```bash
composer require olakunlevpn/laravel-installer-license
```

Done. The step registers automatically.

## Setup your API

The package calls your API to verify licenses.

**Set API URL:**

`.env`:
```
LICENSE_API_URL=https://api.yoursite.com/verify
```

**Your API receives:**
```json
{
    "license_key": "xxxx-xxxx-xxxx-xxxx",
    "email": "user@example.com",
    "domain": "customer-site.com"
}
```

**Return this for valid licenses:**
```json
{
    "success": true,
    "message": "License verified",
    "data": {}
}
```

**Return this for invalid licenses:**
```json
{
    "success": false,
    "message": "Invalid license key"
}
```

That's all your API needs to do.

## Database table

The verified license saves to a `licenses` table.

Create the migration:

```bash
php artisan make:migration create_licenses_table
```

```php
Schema::create('licenses', function (Blueprint $table) {
    $table->id();
    $table->string('license_key');
    $table->string('email');
    $table->json('data')->nullable();
    $table->timestamp('verified_at');
    $table->timestamps();
});
```

The package saves data automatically after installation completes.

## Configuration

Only needed if changing defaults.

**Publish config:**

```bash
php artisan vendor:publish --tag=laravel-installer-license-config
```

**Config file** (`config/installer-license.php`):

```php
return [
    // Step position (default: 2)
    'step_position' => env('LICENSE_STEP_POSITION', 2),

    // API endpoint
    'api_url' => env('LICENSE_API_URL'),

    // Timeout in seconds (default: 10)
    'timeout' => env('LICENSE_API_TIMEOUT', 10),

    // Skip verification (dev only, default: false)
    'skip_verification' => env('LICENSE_SKIP_VERIFICATION', false),
];
```

**Environment variables:**

```
LICENSE_STEP_POSITION=2
LICENSE_API_URL=https://api.yoursite.com/verify
LICENSE_API_TIMEOUT=10
LICENSE_SKIP_VERIFICATION=false
```

## Customize views

**Publish views:**

```bash
php artisan vendor:publish --tag=laravel-installer-license-views
```

Edit in `resources/views/vendor/license/`.

## Translate text

**Publish translations:**

```bash
php artisan vendor:publish --tag=laravel-installer-license-translations
```

Edit in `lang/en/license.php`.

## Development mode

Skip API calls during development:

`.env`:
```
LICENSE_SKIP_VERIFICATION=true
```

**Never use in production.**

## Events

Listen to license events in `app/Providers/EventServiceProvider.php`:

```php
protected $listen = [
    \Olakunlevpn\InstallerLicense\Events\LicenseVerified::class => [
        \App\Listeners\SendNotification::class,
    ],
    \Olakunlevpn\InstallerLicense\Events\LicenseVerificationFailed::class => [
        \App\Listeners\LogFailedAttempt::class,
    ],
];
```

## Next

- Install [Account Setup](account-setup.md)
- Build your own: [Plugin Guide](../creating-plugins/plugin-structure.md)
