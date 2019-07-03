---
title: "Custom Fields"
weight: "600"
menu:
  main:
    parent: "guides"
---

Timber tries to make it as easy as possible for you to retrieve custom meta data for Post, Term, User and Comment objects. And it works with a range of plugins that make it easier for you to create custom fields, like Advanced Custom Fields. While most of this guide applies to everything you do with custom fields, we have separate guides in the [Integrations section](https://timber.github.io/docs/integrations/).

## Accessing custom values

When you have a custom field that’s named `my_custom_field`, you can access it in 3 different ways:

1. Access it [through the `meta()` method](#the-meta-method).
2. Access it [through the `raw_meta()` method](#the-raw-meta-method).
3. Define [your own method](#define-your-own-method).
4. Directly access it [through the field’s name](#direct-access-through-custom-field-name).

Each of these methods behaves a little differently. Let’s look at each one in more detail.

### The `meta()` method

This method is the **recommended way to access meta values**. You’ll get values that are **filtered** by third-party plugins (e.g. Advanced Custom Fields).

**Twig**

```twig
{{ post.meta('my_custom_field') }}
```

**PHP**

```php
$my_custom_field = $post->meta( 'my_custom_field' );
```

### The `raw_meta()` method

With this method, values are **raw** (directly from the database) and are **not filtered** by third-party plugins (e.g. Advanced Custom Fields).

**Twig**

```twig
{{ post.raw_meta('my_custom_field') }}
```

**PHP**

```
$my_custom_field = $post->raw_meta( 'my_custom_field' );
```

### Define your own method

Sometimes you need to modify a meta value before it is returned. You can do that by [extending a Timber object](https://timber.github.io/docs/guides/extending-timber/) and defining your own method. In the following example, we set your custom `price()` method to format the price that’s saved in a custom field named `price`.

**PHP**

```php
class CustomPost extends Timber\Post {
    /**
     * Gets formatted price.
     */
    public function price() {
        $price = $this->meta( 'price' );
        
        // Remove decimal digits.
        return number_format( $price, 0, '', '' );
    }
}
```

In Twig, you would access it like this:

**Twig**

```twig
{{ post.price }}
```

You’ll know from looking at the code that you call a defined property or method. Be aware that through this method, you might overwrite a method that already exists on for a `Timber\Post` object (like [`date`](https://timber.github.io/docs/reference/timber-post/#date)), which is totally fine if you know what you’re doing.

### Direct access through custom field name

If a directly accessed property or method doesn’t exist, Timber will fall back to the equivalent `{{ post.meta('post') }}` call. This means that you can always write `{{ post.price }}`, even if you don’t have a method defined.

**We don’t recommend to use this method, because you might run into conflicts with existing properties or methods on an object.**

For example, when you use a custom field that you name `date` and try to get its value through `{{ post.date }}`, it won’t work. That’s because [`date`](https://timber.github.io/docs/reference/timber-post/#date) is a method of the `Timber\Post` object that returns the date a post was published.

In PHP, you’d access the property through `$post->date` and call the `date` method through `$post->date()`. But in Twig, you can call methods without using brackets. And methods take precedence over properties. Timber uses a PHP technique called [Overloading](http://de.php.net/manual/en/language.oop5.overloading.php#language.oop5.overloading.members) to get meta values using PHP’s [`__get` magic method](http://php.net/manual/en/language.oop5.overloading.php#object.get) on `Timber\Post`, `Timber\Term`, `Timber\User` and `Timber\Comment` objects. This means that when you use `{{ post.date}}`, it will

- Check if method method `date` exists. If it does, it will return what the method produces.
- Otherwise, it will check if a property `date` exists. If it does, it will return its value.
- Otherwise, it will hit the `__get()` method, which handles undefined or inaccessible properties. Timber’s `__get()` method will look for existing meta and return these.

Now, if you ever run into this problem, you should either use the [`meta()`](#the-meta-method) or [`raw_meta()`](#the-raw-meta-method) method.

### The `$custom` property

The `custom` property that’s always set on an object is an array that holds the values of all the meta values from the meta database table (`wp_postmeta`, `wp_termmeta`, `wp_usermeta`or  `wp_commentmeta`, depending on the object you have at hand).

This property only acts **as a reference** for all the existing meta values. You **can’t access the values through that property**, because it’s protected. You need to use the `meta()` or `raw_meta()` method instead. So instead of doing:

```twig
{{ post.my_custom_field }}
```

You would use one of the following:

```twig
{{ post.meta('my_custom_field') }}
{{ post.raw_meta('my_custom_field') }}
```

You can still use the `$custom` property to get a list of all the meta values on a post:

**PHP**

```php
var_dump( $post->custom );
```

**Twig**

```twig
{{ dump(post.custom) }}
```

## Site options

You can get site options through a similar method. Instead of `meta()`, it’s called `option()`. Here’s an example to retrieve the admin email address:

```twig
{{ site.option('admin_email') }}
```

For site options, it’s also possible to access it directly through its name:

```twig
{{ site.admin_email }}
```

Please be aware that using this might conflict with existing Timber methods on the `Timber\Site` object. That’s why the `option()` method is the preferred way to retrieve site options.

## Query by custom field value

This example that uses a [WP_Query](http://codex.wordpress.org/Class_Reference/WP_Query) array shows the arguments to find all posts where a custom field called `color` has a value of `red`.

```php
$args = array(
    'numberposts' => -1,
    'post_type'   => 'post',
    'meta_key'    => 'color',
    'meta_value'  => 'red',
);

$context['posts'] = new Timber\PostQuery($args);
```