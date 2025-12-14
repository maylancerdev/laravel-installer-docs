---
title: "Packages Index"
weight: 1
---

# Official Packages

Add-on packages that extend the installer with additional steps.

## Welcome

**Package:** `olakunlevpn/laravel-installer-welcome`

Adds a customizable welcome step with your branding.

```bash
composer require olakunlevpn/laravel-installer-welcome
```

[Full documentation →](welcome.md)

## License Verification

**Package:** `olakunlevpn/laravel-installer-license`

Adds a step to verify purchase codes via your API.

```bash
composer require olakunlevpn/laravel-installer-license
```

[Full documentation →](license-verification.md)

## Account Setup

**Package:** `olakunlevpn/laravel-installer-account`

Adds a step to create the admin user during installation.

```bash
composer require olakunlevpn/laravel-installer-account
```

[Full documentation →](account-setup.md)

## Installation

```bash
composer require olakunlevpn/laravel-installer-license
composer require olakunlevpn/laravel-installer-account
```

That's it. Packages auto-register and add their steps automatically.

## Configuration

Only configure if you need to change defaults.

**Set step positions** (where steps appear):

`.env`:
```
LICENSE_STEP_POSITION=2
ACCOUNT_STEP_POSITION=5
```

**Publish config files** (optional):

```bash
php artisan vendor:publish --tag=laravel-installer-license-config
php artisan vendor:publish --tag=laravel-installer-account-config
```

**Customize views** (optional):

```bash
php artisan vendor:publish --tag=laravel-installer-license-views
php artisan vendor:publish --tag=laravel-installer-account-views
```

**Translate text** (optional):

```bash
php artisan vendor:publish --tag=laravel-installer-license-translations
php artisan vendor:publish --tag=laravel-installer-account-translations
```

## Creating your own

See [plugin creation guide](../creating-plugins/plugin-structure.md).
