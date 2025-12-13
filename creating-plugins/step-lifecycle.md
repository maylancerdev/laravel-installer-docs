# Step Lifecycle

Understanding how and when your step methods are called.

## The flow

When a user lands on your step, here's what happens:

1. **mount()** - Component initializes, load existing data
2. User fills the form
3. User clicks "Continue"
4. **validateStep()** - Validate form data
5. **execute()** - Save data if validation passes
6. Navigate to next step

If validation fails, the user stays on your step and sees error messages.

## Method execution order

```
Component Created
       ↓
   mount()
       ↓
   render() ← Shows the form
       ↓
User clicks "Continue"
       ↓
validateStep()
       ↓
Did validation pass?
   ↙        ↘
  No        Yes
   ↓         ↓
render()  execute()
(errors)     ↓
          render()
             ↓
       Next step
```

## Detailed breakdown

### Component initialization

```php
protected function mount(): void
{
    $this->loadFormData();
}
```

Runs once when the step loads. Use it to:
- Load data from session
- Set default values
- Initialize component state

**Example with defaults:**

```php
protected function mount(): void
{
    $this->loadFormData();

    if (empty($this->formData['port'])) {
        $this->formData['port'] = 3306;
    }
}
```

### User interaction

User fills the form. Livewire automatically syncs `wire:model` fields to `$this->formData`.

No code needed - it just works.

### Form submission

User clicks your submit button:

```blade
<x-installer::button type="submit" wireTarget="submit">
    Continue
</x-installer::button>
```

This calls the `submit()` method inherited from `BaseStep`. You never override this method.

### Validation phase

```php
protected function validateStep(): void
{
    $this->validate();
}
```

Called automatically by `submit()`. Validation rules come from `rules()`.

**If validation fails:**
- User stays on current step
- Error messages display via `<x-installer::form.error />`
- `execute()` is NOT called

**If validation passes:**
- Continue to execution phase

### Execution phase

```php
protected function execute(): void
{
    $this->saveToSession('yourplugin', $this->formData);
}
```

Only runs if validation passed. Use it to:
- Save data to session
- Make API calls
- Update database (if migrations ran)
- Set success messages

**Example with API call:**

```php
protected function execute(): void
{
    // Always save to session first
    $this->saveToSession('license', $this->formData);

    // Then do additional work
    try {
        $response = Http::post('https://api.example.com/verify', [
            'license' => $this->formData['license_key'],
        ]);

        if ($response->successful()) {
            $this->successMessage = __('license::license.verified');
        } else {
            $this->errorMessage = __('license::license.invalid');
        }
    } catch (\Exception $e) {
        $this->errorMessage = __('license::license.error');
    }
}
```

### Navigation

After `execute()` completes, the installer:
- Calls `render()` one final time
- Navigates to the next step
- The next step's `mount()` is called

## Property persistence

Livewire persists these properties between requests:

```php
public $formData = [];
public $errorMessage = '';
public $successMessage = '';
```

Set them in any method, they'll be available in the view:

```php
protected function execute(): void
{
    $this->saveToSession('app', $this->formData);
    $this->successMessage = 'Settings saved!';
}
```

```blade
@if($successMessage)
    <x-installer::alert.success>
        {{ $successMessage }}
    </x-installer::alert.success>
@endif
```

## Session data flow

Session is the single source of truth until migrations run:

```
Step 1: execute()
    ↓
saveToSession('welcome', [...])
    ↓
Session: {welcome: {...}}

Step 2: execute()
    ↓
saveToSession('database', [...])
    ↓
Session: {welcome: {...}, database: {...}}

Step 3: execute()
    ↓
Database migrations run
    ↓
Move session data to database
    ↓
Session cleared
```

Access data from previous steps:

```php
protected function mount(): void
{
    $this->loadFormData();

    $databaseConfig = $this->getFromSession('database');
    $appName = $this->getFromSession('app.app_name');
}
```

## Skipping steps

Steps can be conditional. Override `shouldDisplay()`:

```php
public function shouldDisplay(): bool
{
    // Only show this step if license verification is enabled
    return config('installer.require_license', false);
}
```

If `shouldDisplay()` returns `false`, the step is skipped entirely. None of its methods run.

## Error handling

Validation errors show automatically via `<x-installer::form.error />`.

For custom errors, use `$this->errorMessage`:

```php
protected function execute(): void
{
    try {
        // Some operation
    } catch (\Exception $e) {
        $this->errorMessage = __('yourplugin::yourplugin.error');
        return; // Stop execution
    }

    $this->successMessage = __('yourplugin::yourplugin.success');
}
```

Display in the view:

```blade
@if($errorMessage)
    <x-installer::alert.error>
        {{ $errorMessage }}
    </x-installer::alert.error>
@endif
```

## Re-rendering

Livewire re-renders your component automatically:
- After `mount()`
- After any Livewire action (like `submit()`)
- When properties change

Each time, `render()` is called with fresh data:

```php
public function render()
{
    return view('yourplugin::livewire.steps.your-step', [
        'fields' => config('installer-yourplugin.form_fields'),
        'isVerified' => $this->isVerified(),
    ]);
}
```

## Next steps

- Follow best practices: [Best Practices](best-practices.md)
- See helper traits in action: [Helper Traits](../advanced/helper-traits.md)
- View a complete example: [Complete Plugin Example](../examples/complete-plugin.md)
