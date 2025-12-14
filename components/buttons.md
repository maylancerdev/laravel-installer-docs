---
title: "Buttons"
weight: 2
---

# Buttons

Button components with built-in loading states.

## Primary button

```blade
<x-installer::button type="submit">
    Save Changes
</x-installer::button>
```

```html
<button type="submit" class="... bg-primary text-white ...">
    Save Changes
</button>
```

## With loading state

```blade
<x-installer::button
    type="submit"
    wireTarget="saveData"
    loadingText="Saving..."
>
    Save Data
</x-installer::button>
```

When Livewire calls `saveData`, the button shows:

```html
<!-- Before loading -->
<button type="submit" class="...">
    Save Data
</button>

<!-- During loading -->
<button type="submit" disabled class="...">
    <svg class="animate-spin ...">...</svg>
    Saving...
</button>
```

## Secondary button

```blade
<x-installer::button variant="secondary">
    Cancel
</x-installer::button>
```

```html
<button class="... border-2 border-gray-300 bg-white text-gray-700 ...">
    Cancel
</button>
```

## Button types

```blade
{{-- Submit button (default) --}}
<x-installer::button type="submit">Submit</x-installer::button>

{{-- Regular button --}}
<x-installer::button type="button">Click Me</x-installer::button>

{{-- Reset button --}}
<x-installer::button type="reset">Reset</x-installer::button>
```

## Complete example

```blade
<form wire:submit.prevent="updateProfile">
    {{-- Form fields here --}}

    <div class="flex gap-3">
        <x-installer::button
            type="submit"
            wireTarget="updateProfile"
            loadingText="Updating..."
        >
            Update Profile
        </x-installer::button>

        <x-installer::button
            variant="secondary"
            type="button"
            wire:click="cancel"
        >
            Cancel
        </x-installer::button>
    </div>
</form>
```

## Custom styling

Add classes for spacing, width, etc:

```blade
<x-installer::button class="mt-6 w-full">
    Full Width Button
</x-installer::button>
```

## Disabled state

```blade
<x-installer::button disabled>
    Cannot Click
</x-installer::button>
```

The button automatically disables during loading when `wireTarget` is set.
