---
title: "Views and Assets"
weight: 4
---

# Views and Assets

Customize installer appearance from your plugin.

## No Build Step Required

The installer uses **Tailwind CSS via CDN** - shipped ready to use.

No `package.json`. No `npm install`. No `npm run build`.

Just Composer and you're done.

## Publishing Views

Override any installer view:

```bash
php artisan vendor:publish --tag=laravel-installer-views
```

Views copy to `resources/views/vendor/installer/`.

## Your Plugin's Views

Load views from your plugin:

```php
// In your service provider
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'yourplugin');
}
```

Reference in your step:

```php
public function render()
{
    return view('yourplugin::livewire.steps.your-step');
}
```

## Override Installer Views

Your plugin can replace installer views.

### Service provider setup

```php
public function boot(): void
{
    // Publish your overrides
    $this->publishes([
        __DIR__.'/../resources/views/installer-overrides' => resource_path('views/vendor/installer'),
    ], 'yourplugin-installer-views');
}
```

### Available views to override

**Layout:**
- `app.blade.php` - Main HTML wrapper
- `livewire/installer-wizard.blade.php` - Wizard container

**Components:**
- `components/card.blade.php`
- `components/button.blade.php`
- `components/section.blade.php`
- `components/icon.blade.php`
- `components/alert/*.blade.php`
- `components/form/*.blade.php`

**Steps:**
- `livewire/steps/welcome.blade.php`
- `livewire/steps/requirements.blade.php`
- `livewire/steps/permissions.blade.php`
- `livewire/steps/environment.blade.php`
- `livewire/steps/installing.blade.php`
- `livewire/steps/completed.blade.php`

## Tailwind Customization

Customize colors in your layout override:

```blade
<!-- resources/views/vendor/installer/app.blade.php -->
<script src="https://cdn.tailwindcss.com"></script>
<script>
    tailwind.config = {
        darkMode: 'class',
        theme: {
            extend: {
                colors: {
                    primary: '#8b5cf6', // Your brand color
                    success: '#10b981',
                    danger: '#ef4444',
                },
            }
        }
    }
</script>
```

No compilation needed - changes apply instantly.

## Custom Fonts

Add fonts in layout override:

```blade
<head>
    <!-- Your custom font -->
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap" rel="stylesheet">

    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Poppins', 'sans-serif'],
                    },
                }
            }
        }
    </script>
</head>
```

## Plugin Assets

Ship custom CSS or JS with your plugin.

### Publish assets

```php
// Service provider
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../public' => public_path('vendor/yourplugin'),
    ], 'yourplugin-assets');
}
```

### Include in your step view

```blade
<!-- yourplugin::livewire.steps.your-step -->
<div>
    <link rel="stylesheet" href="{{ asset('vendor/yourplugin/custom.css') }}">

    <!-- Your step content -->

    <script src="{{ asset('vendor/yourplugin/custom.js') }}"></script>
</div>
```

## Example: Custom Card Component

Override card styling:

```blade
<!-- resources/views/vendor/installer/components/card.blade.php -->
@props([
    'title' => null,
    'description' => null,
])

<div {{ $attributes->merge(['class' => 'bg-gradient-to-br from-purple-900 to-indigo-900 rounded-2xl p-8 shadow-2xl border border-purple-500/20']) }}>
    @if($title)
        <div class="mb-6">
            <h3 class="text-2xl font-bold text-white">{{ $title }}</h3>
            @if($description)
                <p class="mt-2 text-purple-200">{{ $description }}</p>
            @endif
        </div>
    @endif

    {{ $slot }}
</div>
```

All your steps automatically use the new design.

## Example: Brand Logo in Header

Override wizard view to add logo:

```blade
<!-- resources/views/vendor/installer/livewire/installer-wizard.blade.php -->
<div class="flex flex-col lg:flex-row gap-0 rounded-2xl overflow-hidden shadow-2xl bg-white dark:bg-gray-800 max-w-6xl mx-auto">
    <!-- Left Sidebar -->
    <div class="lg:w-96 bg-gradient-to-br from-indigo-600 to-purple-700">
        <!-- Logo at top -->
        <div class="p-6 border-b border-white/10">
            <img src="{{ asset('images/logo.svg') }}" alt="Logo" class="h-12">
        </div>

        <!-- Steps navigation here -->
    </div>

    <!-- Content area -->
    <div class="flex-1">
        <!-- Step content -->
    </div>
</div>
```

## User Instructions

Tell users how to apply your customizations:

**In your README:**

```markdown
## Customization

Publish our custom installer views:

```bash
php artisan vendor:publish --tag=yourplugin-installer-views
```

This applies our branded installer design.
```

## Next Steps

- See [component documentation](../components/form-components.md)
- Check [customization options](../advanced/customization.md)
- Review [complete example](../examples/complete-plugin.md)
