## Blog App


**`resources/views`**

**resources/views/components/form/button.blade.php**
```html
<button type="submit"
        class="bg-blue-500 text-white uppercase font-semibold text-xs py-2 px-10 rounded-2xl hover:bg-blue-600 mt-5">
    {{ $slot }}
</button>

```

**resources/views/components/form/error.blade.php**
```php
@props(['name'])

@error($name)
<p class="text-red-500 text-xs mt-2">{{ $message }}</p>
@enderror
```

**resources/views/components/form/field.blade.php**
```php
<div class="mt-6">
    {{ $slot }}
</div>
```

**resources/views/components/form/form-input.blade.php**
```php
@props(['name'])
<x-form.field>
    <x-form.label name="{{ $name }}"/>
    <input name="{{ $name }}" id="{{ $name }}"
           {{ $attributes(['value' => old($name)]) }}
           class="border border-gray-200 p-2 w-full rounded">
    <x-form.error name="{{ $name }}"/>
</x-form.field>
```

**resources/views/components/form/label.blade.php**
```php
@props(['name'])

<label for="{{ $name }}" class="block mb-2 uppercase font-bold text-xs text-gray-700">{{ ucwords($name) }}</label>
```

**resources/views/components/form/textarea.blade.php**
```php
@props(['name'])
<x-form.field>
    <x-form.label name="{{ $name }}"/>
    <textarea class="border border-gray-200 rounded p-2 w-full" name="{{ $name }}" id="{{ $name }}"
              required
            {{ $attributes  }}> {{ $slot ?? old($name)}}
    </textarea>

    <x-form.error name="{{ $name }}"/>
</x-form.field>

```

