# Account Setup

**Package:** `olakunlevpn/laravel-installer-account`

Adds a step to create the admin user during installation.

![welcome](/images/account-setup.png)


## Install

```bash
composer require olakunlevpn/laravel-installer-account
```

Done. The step registers automatically.

## Requirements

Your `users` table needs these columns:

```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email')->unique();
    $table->string('password');
    $table->string('role')->default('user'); // optional
    $table->timestamps();
});
```

Laravel's default migration already has `name`, `email`, and `password`.

Just add `role` if you want role assignment.

## Configuration

Only needed if changing defaults.

**Publish config:**

```bash
php artisan vendor:publish --tag=laravel-installer-account-config
```

**Config file** (`config/installer-account.php`):

```php
return [
    // Step position (default: 5)
    'step_position' => env('ACCOUNT_STEP_POSITION', 5),

    // Default role for admin (set to null to skip)
    'default_role' => env('ACCOUNT_DEFAULT_ROLE', 'admin'),

    // Column name for role
    'role_column' => 'role',
];
```

**Environment variables:**

```
ACCOUNT_STEP_POSITION=5
ACCOUNT_DEFAULT_ROLE=admin
```

## Role types

**String role:**
```php
// Migration
$table->string('role')->default('user');

// Config
'default_role' => 'admin',
```

**Integer role_id:**
```php
// Migration
$table->foreignId('role_id')->constrained();

// Config
'default_role' => 1,
'role_column' => 'role_id',
```

**No role:**
```php
// Config
'default_role' => null,
```

## License integration

If you also install the license package, the email field auto-fills from the license step.

```bash
composer require olakunlevpn/laravel-installer-license
composer require olakunlevpn/laravel-installer-account
```

Set positions so license comes first:

```
LICENSE_STEP_POSITION=2
ACCOUNT_STEP_POSITION=5
```

## Customize views

**Publish views:**

```bash
php artisan vendor:publish --tag=laravel-installer-account-views
```

Edit in `resources/views/vendor/account/`.

## Translate text

**Publish translations:**

```bash
php artisan vendor:publish --tag=laravel-installer-account-translations
```

Edit in `lang/en/account.php`.

## Events

Listen to account events in `app/Providers/EventServiceProvider.php`:

```php
protected $listen = [
    \Olakunlevpn\InstallerAccount\Events\AccountCreated::class => [
        \App\Listeners\SendWelcomeEmail::class,
    ],
];
```

## Next

- Install [License Verification](license-verification.md)
- Build your own: [Plugin Guide](../creating-plugins/plugin-structure.md)
