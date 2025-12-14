---
title: "Alerts"
weight: 3
---

# Alerts

Alert components for success, error, warning, and info messages.

## Success alert

```blade
<x-installer::alert.success title="Success!">
    Your data has been saved successfully.
</x-installer::alert.success>
```

```html
<div class="p-4 rounded-lg bg-green-50 dark:bg-green-900/20 border-2 border-green-200 ...">
    <svg class="..."><!-- checkmark icon --></svg>
    <strong>Success!</strong>
    <div>Your data has been saved successfully.</div>
</div>
```

## Error alert

```blade
<x-installer::alert.error title="Error">
    Something went wrong. Please try again.
</x-installer::alert.error>
```

```html
<div class="p-4 rounded-lg bg-red-50 dark:bg-red-900/20 border-2 border-red-200 ...">
    <svg class="..."><!-- X icon --></svg>
    <strong>Error</strong>
    <div>Something went wrong. Please try again.</div>
</div>
```

## Warning alert

```blade
<x-installer::alert.warning title="Warning">
    Please review your settings before proceeding.
</x-installer::alert.warning>
```

```html
<div class="p-4 rounded-lg bg-yellow-50 dark:bg-yellow-900/20 border-2 border-yellow-200 ...">
    <svg class="..."><!-- exclamation icon --></svg>
    <strong>Warning</strong>
    <div>Please review your settings before proceeding.</div>
</div>
```

## Info alert

```blade
<x-installer::alert.info title="Did you know?">
    You can customize this step by editing the config file.
</x-installer::alert.info>
```

```html
<div class="p-4 rounded-lg bg-blue-50 dark:bg-blue-900/20 border-2 border-blue-200 ...">
    <svg class="..."><!-- info icon --></svg>
    <strong>Did you know?</strong>
    <div>You can customize this step by editing the config file.</div>
</div>
```

## Without title

```blade
<x-installer::alert.success>
    Saved successfully!
</x-installer::alert.success>
```

```html
<div class="...">
    <svg class="...">...</svg>
    Saved successfully!
</div>
```

## Generic alert

Use the base alert component with a type parameter:

```blade
<x-installer::alert type="success" title="Done">
    Operation completed.
</x-installer::alert>

<x-installer::alert type="error" title="Failed">
    Operation failed.
</x-installer::alert>
```

Available types: `success`, `error`, `warning`, `info`

## Rich content

Alerts accept any HTML in the slot:

```blade
<x-installer::alert.error title="Validation Failed">
    <p>Please fix the following errors:</p>
    <ul class="list-disc list-inside mt-2">
        <li>Email is required</li>
        <li>Password must be at least 8 characters</li>
    </ul>
</x-installer::alert.error>
```

## Conditional alerts

Show alerts based on conditions:

```blade
@if($errorMessage)
    <x-installer::alert.error>
        {{ $errorMessage }}
    </x-installer::alert.error>
@endif

@if($successMessage)
    <x-installer::alert.success>
        {{ $successMessage }}
    </x-installer::alert.success>
@endif
```

## Custom spacing

```blade
<x-installer::alert.info title="Tip" class="mt-6 mb-4">
    This alert has custom margins.
</x-installer::alert.info>
```

All alerts work in dark mode automatically - no configuration needed.
