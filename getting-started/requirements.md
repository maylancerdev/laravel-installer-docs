---
title: "Requirements"
weight: 3
---

# Requirements

System prerequisites for the installer.

## PHP 8.2+

Required extensions:

- `ctype`, `curl`, `dom`, `fileinfo`, `filter`, `hash`
- `mbstring`, `openssl`, `pcre`, `pdo`, `session`
- `tokenizer`, `xml`

Most PHP installations include these.

## Laravel 12+

Built for Laravel 12. Won't work on Laravel 11 or earlier.

## Livewire 3+

All steps are Livewire 3 components.

Livewire 2.x not compatible.

## Frontend

**Tailwind CSS via CDN** - built-in, no setup needed.

No npm. No Vite. No build step.

Just install and use.

## File Permissions

Write access needed (775):

- `storage/framework/`
- `storage/logs/`
- `bootstrap/cache/`

Installer checks these automatically.

## Database

MySQL 5.7+ or PostgreSQL 10+ recommended.

SQLite works for development.

Database must exist before installation - installer doesn't create it.

## Server

**Development:** `php artisan serve`

**Production:** Nginx/Apache with PHP-FPM

## Browser Support

- Chrome/Edge 90+
- Firefox 88+
- Safari 14+

## Automatic Verification

Requirements step checks:
- PHP version
- PHP extensions
- File permissions

Environment step verifies database connectivity.

Clear error messages show what's missing.

## Next Steps

- [Install the package](installation.md)
- [Quick start guide](quick-start.md)
