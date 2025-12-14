---
title: "Layout Components"
weight: 4
---

# Layout Components

Helper components for structuring your installer steps.

## Card

Container with optional header and description.

```blade
<x-installer::card title="Database Settings" description="Configure your database connection">
    {{-- Card content --}}
    <x-installer::form.input name="host" label="Database Host" />
    <x-installer::form.input name="port" label="Port" />
</x-installer::card>
```

```html
<div class="rounded-lg bg-white dark:bg-gray-800 border border-gray-200 ... p-6">
    <div class="mb-4">
        <h3 class="text-lg font-semibold ...">Database Settings</h3>
        <p class="mt-1 text-sm ...">Configure your database connection</p>
    </div>
    <div>
        <!-- Card content -->
    </div>
</div>
```

**Without header:**

```blade
<x-installer::card>
    Just some content in a nice card.
</x-installer::card>
```

**Custom padding:**

```blade
<x-installer::card padding="p-8" title="Larger Padding">
    Content with more space
</x-installer::card>
```

## Section

Content section with title and description.

```blade
<x-installer::section title="Email Configuration" description="Set up your SMTP settings">
    <x-installer::form.input name="smtp_host" label="SMTP Host" />
    <x-installer::form.input name="smtp_port" label="SMTP Port" />
</x-installer::section>
```

```html
<div class="space-y-4">
    <div>
        <h2 class="text-xl font-bold ...">Email Configuration</h2>
        <p class="mt-1 text-sm ...">Set up your SMTP settings</p>
    </div>
    <div>
        <!-- Section content -->
    </div>
</div>
```

**Without description:**

```blade
<x-installer::section title="Settings">
    Content here
</x-installer::section>
```

## Icon

SVG icons for various purposes.

```blade
<x-installer::icon name="check" size="w-6 h-6" />
```

```html
<svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
    <path d="..." />
</svg>
```

**Available icons:**

```blade
{{-- Success/check --}}
<x-installer::icon name="check" />

{{-- Error/close --}}
<x-installer::icon name="x" />

{{-- Warning --}}
<x-installer::icon name="exclamation" />

{{-- Info --}}
<x-installer::icon name="info" />

{{-- Navigation --}}
<x-installer::icon name="arrow-right" />
<x-installer::icon name="arrow-left" />

{{-- Loading --}}
<x-installer::icon name="loading" class="animate-spin" />

{{-- Others --}}
<x-installer::icon name="key" />
<x-installer::icon name="user" />
<x-installer::icon name="cog" />
```

**Outlined icons:**

```blade
<x-installer::icon name="check" :solid="false" />
```

**Different sizes:**

```blade
<x-installer::icon name="check" size="w-4 h-4" />
<x-installer::icon name="check" size="w-8 h-8" />
<x-installer::icon name="check" size="w-12 h-12" />
```

## Complete layout example

```blade
<div class="max-w-4xl mx-auto space-y-8">
    <x-installer::section
        title="Application Setup"
        description="Configure your application's basic settings"
    >
        <x-installer::card title="Database Connection">
            <div class="space-y-4">
                <x-installer::form.group :field="$dbHostField" wireModel="formData" />
                <x-installer::form.group :field="$dbPortField" wireModel="formData" />
            </div>
        </x-installer::card>

        <x-installer::card title="Application Details" class="mt-6">
            <div class="space-y-4">
                <x-installer::form.group :field="$appNameField" wireModel="formData" />
                <x-installer::form.group :field="$appUrlField" wireModel="formData" />
            </div>
        </x-installer::card>
    </x-installer::section>

    <x-installer::alert.info title="Quick Tip" class="mt-8">
        <div class="flex items-start gap-2">
            <x-installer::icon name="info" size="w-5 h-5" class="flex-shrink-0" />
            <span>You can change these settings later in your .env file</span>
        </div>
    </x-installer::alert.info>
</div>
```

All layout components adapt to dark mode and handle responsive breakpoints automatically.
