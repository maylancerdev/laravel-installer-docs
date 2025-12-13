# Form Components

All form components live under the `installer` namespace and support dark mode automatically.

## Form Group

Complete field wrapper with label, input, and error display.

```blade
<x-installer::form.group :field="$field" wireModel="formData" />
```

```php
// Field configuration
$field = [
    'name' => 'email',
    'type' => 'email',
    'label' => 'license::license.email',
    'placeholder' => 'license::license.email_placeholder',
    'required' => true,
];
```

```html
<div>
    <label for="email" class="...">Email *</label>
    <input type="email" id="email" wire:model="formData.email" />
    <!-- Error displays here if validation fails -->
</div>
```

The group handles everything - you just pass the field config and wire:model prefix.

## Input

Text, email, password, number fields.

```blade
<x-installer::form.input
    type="email"
    name="email"
    placeholder="your@email.com"
    wireModel="formData.email"
    required
/>
```

```html
<input type="email" name="email" wire:model="formData.email" required class="..." />
```

All input types work:

```blade
{{-- Text --}}
<x-installer::form.input name="name" />

{{-- Email --}}
<x-installer::form.input type="email" name="email" />

{{-- Password --}}
<x-installer::form.input type="password" name="password" />

{{-- Number --}}
<x-installer::form.input type="number" name="port" />
```

## Textarea

Multi-line text input.

```blade
<x-installer::form.textarea
    name="description"
    rows="4"
    wireModel="formData.description"
/>
```

```html
<textarea name="description" rows="4" wire:model="formData.description" class="..."></textarea>
```

## Select

Dropdown with options.

```blade
<x-installer::form.select
    name="role"
    :options="['admin' => 'Administrator', 'user' => 'User']"
    placeholder="Select role..."
    wireModel="formData.role"
/>
```

```html
<select name="role" wire:model="formData.role" class="...">
    <option value="">Select role...</option>
    <option value="admin">Administrator</option>
    <option value="user">User</option>
</select>
```

You can also use slots for custom options:

```blade
<x-installer::form.select name="country" wireModel="formData.country">
    <option value="us">United States</option>
    <option value="uk">United Kingdom</option>
    <option value="ca">Canada</option>
</x-installer::form.select>
```

## Checkbox

Checkbox with label.

```blade
<x-installer::form.checkbox
    name="terms"
    label="I agree to the terms"
    wireModel="formData.terms"
/>
```

```html
<div class="flex items-center">
    <input type="checkbox" id="terms" name="terms" wire:model="formData.terms" class="..." />
    <label for="terms" class="...">I agree to the terms</label>
</div>
```

Or use slot for custom label markup:

```blade
<x-installer::form.checkbox name="newsletter" wireModel="formData.newsletter">
    Subscribe to our <strong>weekly newsletter</strong>
</x-installer::form.checkbox>
```

## Radio

Radio button with label.

```blade
<x-installer::form.radio
    name="plan"
    value="pro"
    label="Pro Plan - $29/mo"
    wireModel="formData.plan"
/>

<x-installer::form.radio
    name="plan"
    value="enterprise"
    label="Enterprise - $99/mo"
    wireModel="formData.plan"
/>
```

```html
<div class="flex items-center">
    <input type="radio" id="plan_pro" name="plan" value="pro" wire:model="formData.plan" class="..." />
    <label for="plan_pro" class="...">Pro Plan - $29/mo</label>
</div>

<div class="flex items-center">
    <input type="radio" id="plan_enterprise" name="plan" value="enterprise" wire:model="formData.plan" class="..." />
    <label for="plan_enterprise" class="...">Enterprise - $99/mo</label>
</div>
```

## Label

Standalone label with optional required indicator.

```blade
<x-installer::form.label for="email" required>
    Email Address
</x-installer::form.label>
```

```html
<label for="email" class="...">
    Email Address
    <span class="text-red-500">*</span>
</label>
```

## Error

Display validation errors.

```blade
<x-installer::form.error name="formData.email" />
```

```html
<!-- Only shows if error exists -->
<p class="mt-2 text-sm text-red-600 dark:text-red-400 flex items-center gap-1">
    <svg class="w-4 h-4">...</svg>
    The email field is required.
</p>
```

## Complete form example

```blade
<form wire:submit.prevent="submit">
    <x-installer::form.group :field="[
        'name' => 'name',
        'type' => 'text',
        'label' => 'Full Name',
        'required' => true,
    ]" wireModel="formData" />

    <x-installer::form.group :field="[
        'name' => 'email',
        'type' => 'email',
        'label' => 'Email Address',
        'required' => true,
    ]" wireModel="formData" />

    <x-installer::form.group :field="[
        'name' => 'message',
        'type' => 'textarea',
        'label' => 'Message',
        'rows' => 4,
    ]" wireModel="formData" />

    <x-installer::button type="submit" wireTarget="submit">
        Send Message
    </x-installer::button>
</form>
```

All components automatically handle:
- Dark mode styling
- Focus states
- Disabled states
- Validation error display
- Livewire binding

## Custom styling

All components accept additional classes via attributes:

```blade
<x-installer::form.input name="email" class="mt-4 max-w-md" />
```

The classes merge with default styling - they don't replace it.
