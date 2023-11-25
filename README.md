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

![php-version-82](https://shields.io/badge/php-<=8.2-blue)

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

![php-version-82](https://shields.io/badge/php-<=8.2-blue)
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

![php-version-83](https://shields.io/badge/php->=8.3-blue)

PHP will be able to detect that `Admin::methodWithDefaultImplementation()` doesn't override anything anymore, and it will throw an error:
```php
PHP Fatal error:  Admin::methodWithDefaultImplementation() has #[\Override] attribute, but no matching parent method exists in /tmp/8.php on line 14
```

![php-version-82](https://shields.io/badge/php-<=8.2-blue)
```php
2
```

### Negative indices in arrays

If you have an empty array, add an item with a negative index, and then add another item, that second item would always start at index 0:


```php
$array = [];

$array[-5] = 'a';
$array[] = 'b';

var_export($array);
```
![php-version-82](https://shields.io/badge/php-<=8.2-blue)
```php
//array (
//  -5 => 'a',
//  0 => 'b',
//)
```
![php-version-83](https://shields.io/badge/php->=8.3-blue)
```php
//array (
//  -5 => 'a',
//  -4 => 'b',
//)
```

### Anonymous readonly classes
Previously, you weren't able to mark anonymous classes as readonly. That's fixed in PHP 8.3:

```php
$class = new readonly class {
    public function __construct(
        public string $title = 'this is title',
    ) {}
};

echo $class->title;
```
![php-version-82](https://shields.io/badge/php-<=8.2-blue)
```php
PHP Parse error:  syntax error, unexpected token "readonly" in /tmp/8.php on line 3
```
![php-version-83](https://shields.io/badge/php->=8.3-blue)
```php
this is title
```
### The new `json_validate()` function
Previously, the only way to validate whether a string was valid JSON, was to decode it and detect whether any errors were thrown. This new json_validate() function is beneficial if you only need to know whether the input is valid JSON, since it uses less memory compared to decoding the string.

![php-version-82](https://shields.io/badge/php-<=8.2-blue)
```php
function is_valid_json($json_string) {
    $json_data = json_decode($json_string);
    return (json_last_error() === JSON_ERROR_NONE) ? true : false;
}

$json_string = '{"name": "John Doe", "age": 30}';
if (is_valid_json($json_string)) {
    echo "Valid JSON string";
} else {
    echo "Invalid JSON string";
}

// Valid JSON string
```
![php-version-83](https://shields.io/badge/php->=8.3-blue)
```php
$json_string = '{"name": "John Doe", "age": 30}';
if (json_validate($json_string)) {
    echo "Valid JSON string";
} else {
    echo "Invalid JSON string";
}

// Valid JSON string
```

### `Randomizer` additions
PHP 8.2 added the new Randomizer class. This update brings some small additions:

```php
<?php

use Random\Randomizer;

$random = new Randomizer();
echo $random->getBytesFromString('milad',2);

// ma
```

`getFloat()` returns a float between $min and $max
```php
<?php

use Random\Randomizer;

$random = new Randomizer();
echo $random->getFloat(min: 1,max: 10);
//4.2490697405757
```

`nextFloat()` is a shorthand for `getFloat(0, 1, IntervalBoundary::ClosedOpen)`, in other words: it'll give you a random float between 0 and 1, where 1 is excluded.
```php
<?php

use Random\Randomizer;

$random = new Randomizer();
echo $random->nextFloat();
// 0.13968670649046
```
### Dynamic class constant fetch
PHP 8.3 allows you to fetch constants with a more dynamic syntax:

![php-version-82](https://shields.io/badge/php-<=8.2-blue)
```php
<?php
class Post
{
    const TITLE = 'this is title';
}

$name = 'TITLE';

echo constant(Post::class . '::' . $name);
// this is title
```
![php-version-83](https://shields.io/badge/php->=8.3-blue)
```php
<?php
class Post
{
    const TITLE = 'this is title';
}

$name = 'TITLE';

echo Post::{$name};

// this is title
```

### Traits and static properties
Uses of traits with static properties will now redeclare static properties inherited from the parent class. This will create a separate static property storage for the current class. This is analogous to adding the static property to the class directly without traits.

```php
<?php
trait Commentable
{
    public static string $comment = 'this is comment';
}

class Post
{
    use Commentable;

    public function getStaticPro()
    {
        self::$comment = 'post comment';
        return self::$comment;
    }
}

class Child extends Post
{
    use Commentable;
    
    public function getStaticPro()
    {
        return self::$comment;
    }
}

$post = new Post();
$child = new Child();
echo $post->getStaticPro() . PHP_EOL;
echo $child->getStaticPro() . PHP_EOL;
```


![php-version-82](https://shields.io/badge/php-<=8.2-blue)
```php
post comment
post comment
```
![php-version-83](https://shields.io/badge/php->=8.3-blue)
```php
post comment
this is comment
```