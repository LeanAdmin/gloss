# 🔍 Gloss ✨ — Brilliant localization for Laravel

Gloss is a Laravel package for advanced localization.

Laravel's localization system is perfect for many languages, but it breaks down when you need a bit more control.

For example, some languages change words depending on the context they're used in. The words may get prefixed, suffixed, or even changed completely.

Aside from adding support for complex languages, Gloss also ships with quality of life improvements such as the ability to add formatting to a translation string.

## Installation

Laravel 6 or 8 is required, PHP 7.4 is required.

```
composer require leanadmin/gloss
```

## Configuration

By default, Gloss comes with the `gloss()` helper and `___()` helper.

If you wish, you may disable the `___()` helper by setting:
```php
use Gloss;

Gloss::$underscoreHelper = false;
```

And if you wish to make all existing `__()` calls Gloss-aware:
```php
use Gloss;

Gloss::$shouldReplaceTranslator = true;
```

A good place for these calls is the `boot()` method of your `LeanServiceProvider` or `AppServiceProvider`.

## Usage

Gloss can be used just like the standard Laravel localization helpers:

```php
___('Create :Resource', ['resource' => 'product']);

// 'resources.edit' => 'Show :Resource :title'
gloss('resources.create', ['resource' => 'product', 'title' => 'MacBook Pro 2019']); // Show Product MacBook Pro 2019

// 'notifications.updated' => ':Resource :title has been updated!'
Gloss::get('resources.edit', ['resource' => 'product', 'title' => 'iPhone 12']); // Product iPhone 12 has been updated!

// 'foo.apple' => 'There is one apple|There are many apples'
Gloss::choice('foo.apple', ['count' => 2]); // There are many apples
```

However, unlike the standard localization, it lets you make changes to these strings on the fly:

### Value aliases

Imagine that you're editing the `Order` resource in an admin panel. Your resource's singular label is `Objednávka`, which is Czech for `Order`.

The language string for the create button is
```php
// Original in English: 'create' => 'Create :Resource',
'create' => 'Vytvořit :Resource',
```

If we fill the value with the resource name, we get `Vytvořit Objednávka`. Unfortunately, that's wrong not once, but twice.

Firstly, it should be Objednávk**u**, because the suffix changes with context. And secondly, it's grammatically incorrect to capitalize the word here.

```diff
- Vytvořit Objednávka
+ Vytvořit objednávku
```

So we want to specify a **complete override** of that language string whenever we're in the `Order` resource context.

To do this, simply call:
```
Gloss::value('resource.create', 'Vytvořit objednávku');
```

(From an appropriate place, where the application is working with the Order resource.)

If you're using Lean, this is taken care of for you. You can simply define entire language strings in the `$lang` property of your resource, and they'll be used in all templates which use the resource.

Also note that the example above mentions resources, but that's just how Lean is implemented. Gloss will work with any setup.

### Key aliases

To build up on the example above, let's say our admin panel uses multiple languages. So replacing the language string with a translation that's part of the code isn't feasible.

For this reason, Gloss also lets you alias keys to another keys:

```php
// 'orders.create' => 'Vytvořit objednávku',
// 'resources.create' => 'Vytvořit :resource',

Gloss::key('resource.create', 'orders.create');
```

This is equivalent to fetching the value using a translation helper.
```php
Gloss::value('resource.create', gloss('orders.create'));
Gloss::key('resource.create', 'orders.create');
```

### Extending values

You may also build upon fully resolved language strings.

For example, consider the following example:
```html
Showing <strong>10</strong> to <strong>20</strong> of <strong>50</strong> results.
```

To localize this, we'd either have to localize each word separately (which is what Laravel does, and it breaks down similarly to the "Order" word example), or we'd have to add the markup to the translation strings (which sucks for security, translator life quality), or we'd have to ditch the formatting completely.

All of those are unnecessary trade-offs.

Gloss lets you add formatting after the string is fully built:

```php
// 'pagination' => 'Showing :start to :end of :total results',

Gloss::extend('foo.pagination', fn ($value, $replace) => $replace($value, [
    ':start' => '<span class="font-medium">:start</span>',
    ':end' => '<span class="font-medium">:end</span>',
    ':total' => '<span class="font-medium">:total</span>',
]));

Gloss::get('test.pagination', ['start' => 10, 'end' => 20, 'total' => 50])
// Showing <span class="font-medium">10</span> to <span class="font-medium">20</span> of <span class="font-medium">50</span> results
```

Of course, `extend()` works perfectly with localized strings:
```php
// 'pagination' => 'Zobrazeno :start až :end z :total výsledků',

// Zobrazeno <span class="font-medium">10</span> až <span class="font-medium">20</span> z <span class="font-medium">50</span> výsledků
```

It even works with pluralized/choice-based strings:

```php
// 'apples' => '{0} There are no apples|[1,*]There are :count apples'

Gloss::extend('foo.apples', fn ($apples, $replace) => $replace($apples, [
    ':count' => '<span class="font-medium">:count</span>',
]));

gloss()->choice('foo.apples', 0); // There are no apples
gloss()->choice('foo.apples', 5); // There are <span class="font-medium">5</span> apples
```

The second argument is a callable that can return any string, but in most cases this will simply be the resolved string with a few segments replaced.

For that reason, Gloss automatically passes `$replace` to the callable, which lets you replace parts of the string using beautiful, arrow function-compatible syntax.

```php
// So elegant!

fn ($string, $replace) => $replace($string, [
   'elegant' => 'eloquent',
]);

// So eloquent!
```
