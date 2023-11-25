# PHP 8.3 Changes

### Readonly amendments

overwriting readonly property values within __clone(), in order to allow deep cloning readonly properties:

```php
<?php

readonly class Post
{
    public function __construct
    (
        public string $title,
    ) {}

    public function __clone()
    {
        $this->title = 'cloned title';
        // This is allowed,
        // even though `title` is a readonly property.
    }
}

$post = new Post('post title');

echo $post->title . PHP_EOL;
//post title

$post_clone = clone $post;
echo $post_clone->title . PHP_EOL;
//cloned title

```

![php-version-82](https://shields.io/badge/php=<-8.2-blue)

```php
PHP Fatal error:  Uncaught Error: Cannot modify readonly property Post::$title in /tmp/8.php:14
Stack trace:
#0 /tmp/8.php(24): Post->__clone()
#1 {main}
  thrown in /tmp/main.php on line 14
```

### Typed class constants

You can now typehint class constants:

```php
<?php

class Post
{
    const string TITLE = 'this is title';
}
echo Post::TITLE . PHP_EOL;
// this is title
```

![php-version-82](https://shields.io/badge/php=<-8.2-blue)
```php
PHP Parse error:  syntax error, unexpected identifier "TITLE", expecting "=" in /tmp/8.php on line 5
```

### #[Override] attribute
The new #[Override] attribute is used to show a programmer's intent. It basically says "I know this method is overriding a parent method. If that would ever change, please let me know".

Here's an example:
```php
class User
{
    public function methodWithDefaultImplementation(): int
    {
        return 1;
    }
}

class Admin extends User
{
    #[Override]
    public function methodWithDefaultImplementation(): int
    {
        return 2; // The overridden method
    }
}

$admin = new Admin();

echo $admin->methodWithDefaultImplementation();
```
this code works perfectly but if I change `User/methodWithDefaultImplementation` to `methodWithNewImplementation`, what would happen?

![php-version-82](https://shields.io/badge/php=8.3-blue)
```php
PHP Fatal error:  Admin::methodWithDefaultImplementation() has #[\Override] attribute, but no matching parent method exists in /tmp/8.php on line 14
```

![php-version-82](https://shields.io/badge/php=<-8.2-blue)
```php
2
```