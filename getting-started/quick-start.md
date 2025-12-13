# Quick Start

This guide walks you through a complete installation in about two minutes.

## Before you start

Make sure you've [installed the package](installation.md) and built your assets.

Your `.env` file should be empty or missing - the installer creates it for you.

## Start your server

```bash
php artisan serve
```

Visit `http://localhost:8000/install` in your browser.

## Step 1: Welcome

The welcome screen shows your app name and installer version. Click "Get Started" or "Next" to continue.

## Step 2: Requirements

The installer checks:

- PHP version (8.2+)
- Required PHP extensions (PDO, mbstring, etc.)
- File permissions (storage/, bootstrap/cache/)

Green checkmarks mean you're good. Red X's show what needs fixing.

**If you see errors:**

Missing extensions? Install them via your PHP manager.

Permission issues? Run:

```bash
chmod -R 775 storage bootstrap/cache
```

## Step 3: Permissions

Double-checks folder permissions for:

- `storage/framework/`
- `storage/logs/`
- `bootstrap/cache/`

Everything should be writable (775 or 777).

## Step 4: Environment

Configure your database connection:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=your_database
DB_USERNAME=your_username
DB_PASSWORD=your_password
```

The installer validates your connection before proceeding. If it fails, you'll see an error with details.

**Also configured here:**

- App name
- App URL
- App environment (local, production)
- Debug mode

## Step 5: Installing

The installer runs:

1. Creates/updates `.env` file
2. Generates app key
3. Runs database migrations
4. Executes plugin-specific setup (if any)
5. Clears caches

You'll see progress messages as each task completes.

## Step 6: Completed

Success! The installer shows:

- Installation summary
- Quick links to your app
- Next steps recommendations

Click "Launch Application" to visit your app's dashboard.

## What happens next?

The installer creates a marker file (`storage/installed`) to prevent re-running. If you need to reinstall, delete this file.

Your `.env` is now fully configured and your database is migrated. Your app is ready to use.

## Plugin steps

If you've installed plugins (license verification, account setup), they'll appear between the core steps. The installation flow expands automatically.

Example with plugins:

1. Welcome
2. License Verification ← Plugin step
3. Account Setup ← Plugin step
4. Requirements
5. Permissions
6. Environment
7. Installing
8. Completed

Each plugin step stores its data in session until the Installing step, then saves everything to the database.

## Troubleshooting

**Stuck on a step?**

Check your browser console for JavaScript errors and Laravel logs in `storage/logs/laravel.log`.

**Database connection fails?**

Verify your credentials. Create the database first - the installer doesn't create databases, only tables.

**Requirements not passing?**

Install missing PHP extensions or fix file permissions as indicated.

## Next steps

- Customize the installer: [Customization guide](../advanced/customization.md)
- Create custom steps: [Plugin creation guide](../creating-plugins/plugin-structure.md)
- Use components: [Components documentation](../components/form-components.md)
