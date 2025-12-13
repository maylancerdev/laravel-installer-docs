# Your First Step

Now that you understand the plugin structure, let's create a working step.

## The Step class

A step is a Livewire component that extends `Olakunlevpn\Installer\Steps\BaseStep`.

`src/Steps/YourStep.php`:

```php
<?php

namespace YourVendor\YourPlugin\Steps;

use Olakunlevpn\Installer\Steps\BaseStep;
use Olakunlevpn\Installer\Traits\HasFormData;
use Olakunlevpn\Installer\Traits\ManagesSessionData;

class YourStep extends BaseStep
{
    use HasFormData;
    use ManagesSessionData;

    protected function mount(): void
    {
        $this->loadFormData();
    }

    protected function rules(): array
    {
        return $this->buildValidationRules(
            config('installer-yourplugin.form_fields')
        );
    }

    protected function validateStep(): void
    {
        $this->validate();
    }

    protected function execute(): void
    {
        $this->saveToSession('yourplugin', $this->formData);
    }

    public function render()
    {
        return view('yourplugin::livewire.steps.your-step', [
            'fields' => config('installer-yourplugin.form_fields'),
        ]);
    }
}
```

That's your complete step class. Let's break down what each method does.

## Lifecycle methods

### mount()

Runs when the step is loaded. Load any existing data here.

```php
protected function mount(): void
{
    $this->loadFormData();
}
```

`loadFormData()` comes from the `HasFormData` trait. It populates `$this->formData` from session if available.

### rules()

Returns validation rules for the form.

```php
protected function rules(): array
{
    return $this->buildValidationRules(
        config('installer-yourplugin.form_fields')
    );
}
```

`buildValidationRules()` reads your config and creates Laravel validation rules automatically.

### validateStep()

Runs when user clicks "Continue". Validate the form here.

```php
protected function validateStep(): void
{
    $this->validate();
}
```

Just call `$this->validate()` - the rules from `rules()` are applied automatically.

### execute()

Runs after validation passes. Save your data here.

```php
protected function execute(): void
{
    $this->saveToSession('yourplugin', $this->formData);
}
```

Data goes in the session until migrations run. After that, you can save to the database.

### render()

Returns the view with any data needed.

```php
public function render()
{
    return view('yourplugin::livewire.steps.your-step', [
        'fields' => config('installer-yourplugin.form_fields'),
    ]);
}
```

Pass your form fields to the view.

## The view template

`resources/views/livewire/steps/your-step.blade.php`:

```blade
<div>
    <x-installer::card
        title="{{ __('yourplugin::yourplugin.card_title') }}"
        description="{{ __('yourplugin::yourplugin.card_description') }}"
    >
        <form wire:submit.prevent="submit">
            <div class="space-y-4">
                @foreach($fields as $field)
                    <x-installer::form.group :field="$field" wireModel="formData" />
                @endforeach
            </div>

            <div class="mt-6">
                <x-installer::button
                    type="submit"
                    wireTarget="submit"
                    loadingText="{{ __('yourplugin::yourplugin.verifying') }}"
                >
                    {{ __('yourplugin::yourplugin.submit') }}
                </x-installer::button>
            </div>
        </form>
    </x-installer::card>

    @if($successMessage)
        <x-installer::alert.success class="mt-6">
            {{ $successMessage }}
        </x-installer::alert.success>
    @endif

    @if($errorMessage)
        <x-installer::alert.error class="mt-6">
            {{ $errorMessage }}
        </x-installer::alert.error>
    @endif
</div>
```

The view uses the installer's components. Loop through your fields and render them with `form.group`.

## Helper traits

Your step uses two helper traits:

**HasFormData**
- Manages `$this->formData` property
- `loadFormData()` - loads from session
- `buildValidationRules()` - creates rules from config

**ManagesSessionData**
- `saveToSession($key, $data)` - saves data
- `getFromSession($key)` - retrieves data
- `removeFromSession($key)` - deletes data

Use these instead of writing your own session logic.

## Adding custom validation

Sometimes you need more than config-based validation:

```php
protected function validateStep(): void
{
    $this->validate();

    // Custom validation
    if ($this->formData['api_key'] === 'invalid') {
        $this->addError('formData.api_key', __('yourplugin::yourplugin.invalid_key'));
        return;
    }

    // Make API call to verify
    $response = Http::post('https://api.example.com/verify', [
        'key' => $this->formData['api_key'],
    ]);

    if (!$response->successful()) {
        $this->errorMessage = __('yourplugin::yourplugin.api_error');
        return;
    }

    $this->successMessage = __('yourplugin::yourplugin.verified');
}
```

## Saving to database

After migrations run, you can save to the database:

```php
protected function execute(): void
{
    // Save to session (always do this first)
    $this->saveToSession('yourplugin', $this->formData);

    // Check if database is ready
    try {
        DB::connection()->getPdo();
    } catch (\Exception $e) {
        // Database not ready yet, session is enough
        return;
    }

    // Database is ready, save settings
    Setting::updateOrCreate(
        ['key' => 'api_key'],
        ['value' => $this->formData['api_key']]
    );
}
```

## Testing your step

Start the dev server:

```bash
composer run dev
```

Visit `http://localhost:8000/install` and navigate to your step.

Your step appears at the position configured in `config/installer_yourplugin.php`.

## What if something goes wrong?

Check the browser console for JavaScript errors.

Check Laravel logs in `storage/logs/laravel.log`.

Make sure your translation keys exist in `lang/en/yourplugin.php`.

Verify your config file is published:

```bash
php artisan vendor:publish --tag=installer-yourplugin-config
```

## Next steps

- Understand the complete lifecycle: [Step Lifecycle](step-lifecycle.md)
- Customize views and override installer UI: [Views and Assets](views-and-assets.md)
- Follow best practices: [Best Practices](best-practices.md)
- See a complete example: [Complete Plugin Example](../examples/complete-plugin.md)
