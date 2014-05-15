# Eloquent-Taggable

Easily add the ability to tag your Eloquent models in Laravel 4.

[![Latest Stable Version](https://poser.pugx.org/cviebrock/eloquent-taggable/v/stable.png)](https://packagist.org/packages/cviebrock/eloquent-taggable)
[![Total Downloads](https://poser.pugx.org/cviebrock/eloquent-taggable/downloads.png)](https://packagist.org/packages/cviebrock/eloquent-taggable)

* [Installation and Requirements](#installation)
* [Updating your Eloquent Models](#eloquent)
* [Usage](#usage)
* [Configuration](#config)
* [Extending Taggable](#extending)
* [Bugs, Suggestions and Contributions](#bugs)
* [Copyright and License](#copyright)


<a name="installation"></a>
## Installation and Requirements

First, you'll need to add the package to the `require` attribute of your `composer.json` file:

```json
{
    "require": {
        "cviebrock/eloquent-taggable": "1.*"
    },
}
```

> **Note:** Eloquent-Taggable uses traits, so you will need to be running PHP 5.4 or higher.

Aftwards, run `composer update` from your command line.

Then, run the migration to create the required tables:

```sh
php artisan migrate --package=cviebrock/eloquent-taggable
```

Then, update `app/config/app.php` by adding an entry for the service provider.

```php
	'providers' => array(

		// ...

		'Cviebrock\EloquentTaggable\TaggableServiceProvider',

	);
```

Finally, from the command line again, publish the default configuration file:

```sh
php artisan config:publish cviebrock/eloquent-taggable
```



<a name="eloquent"></a>
## Updating your Eloquent Models

Your models should implement Taggable's interface and use it's trait:

```php
use Cviebrock\EloquentTaggable\Taggable;
use Cviebrock\EloquentTaggable\TaggableImpl;

class MyModel extends Eloquent implements Taggable
{

	use TaggableImpl;

}
```

That's it ... your model is now "taggable"!



<a name="usage"></a>
## Using the Class

Tag your models with the `tag()` method:

```php
// Pass in a delimited string:
$model->tag('Apple,Banana,Cherry');

// Or an array:
$model->tag(['Apple', 'Banana', 'Cherry']);  
```

The `tag()` method is additive, so you can tag the model again and those tags will be added to the previous ones:

```php
$model->tag('Apple,Banana,Cherry');

$model->tag('Durian');
// $model now has four tags
```

You can remove tags individually with `untag()` or entirely with `detag()`:

```php
$model->tag('Apple,Banana,Cherry');

$model->untag('Banana');
// $model is now just tagged with "Apple" and "Cherry"

$model->detag();
// $model has no tags
```

You can also completely retag a model (a short form for detagging then tagging):

```php
$model->tag('Apple,Banana,Cherry');

$model->retag('Etrog,Fig,Grape');
// $model is now just tagged with "Etrog", "Fig", and "Grape"
```

You can get the array of all tags (technically, an Eloquent Collection):

```php
foreach($model->tags as $tag)
{
    echo $tag->name;
}
```

You can also get the list of tags as a flattened array, or a delimited list:

```php
$model->tag('Apple,Banana,Cherry');

var_dump($model->tagList);

// string 'Apple,Banana,Cherry' (length=19)

var_dump($model->tagArray);

// array (size=3)
//  1 => string 'Apple' (length=5)
//  2 => string 'Banana' (length=6)
//  3 => string 'Cherry' (length=6)
```

Tag names are normalized (see below) so that duplicate tags aren't accidentally created:

```php
$model->tag('Apple');
$model->tag('apple');
$model->tag('APPLE');

var_dump($model->tagList);
// string 'Apple' (length=5)
```

Finally, you can easily find models with tags through some query scopes:

```php
Model::withAllTags('apple,banana,cherry');
// returns models that are tagged with all 3 of those tags

Model::withAnyTags('apple,banana,cherry');
// returns models with any one of those 3 tags

Model::withAnyTags();
// returns models with any tags at all
```


<a name="config"></a>
## Configuration

Configuration is handled through the settings in `/app/config/packages/cviebrock/eloquent-taggable/config.php`.  The default values are:

```php

return array(
    'delimiters' => ',;',
    'normalizer' => 'mb_strtolower',
);
```

### delimiters

These are the single-character strings that can delimit the list of tags passed to the `tag()` method.  By default, it's just the comma, but you can change it to another character, or use multiple characters.

For example, if __delimiters__ is set to ";,/", the this will work as expected:

```php
$model->tag('Apple/Banana;Cherry,Durian');
// $model will have four tags
```

When using multiple delimiters, the first one will be used to build strings for the `tagList` attribute.  So, in the above case:

```php
var_dump($model->tagList);

// string 'Apple;Banana;Cherry;Durian' (length=26)
```

### normalizer

Each tag is "normalized" before being stored in the database.  This is so that variations in the spelling or capitalization of tags don't generate duplicate tags.  For example, we don't want three different tags in the following case:

```php
$model->tag('Apple');
$model->tag('APPLE');
$model->tag('apple');
```

Normalization happens by passing each tag name through a normalizer function.  By default, this is PHP's `mb_strtolower()` function, but you can change this to any function or callable that takes a single string value and returns a string value.  Some ideas:

```php

    // default normalization
    'normalizer' => 'mb_strtolower',

    // same result, but using a closure
    'normalizer' => function($string) {
        return mb_strtolower($string);
    },

    // using a class method
    'normalizer' => array('Str','slug'),
```

You can access the normalized values of the tags through `$model->tagListNormalized` and `$model->tagArrayNormalized`, which work identically to `$model->tagList` and `$model->tagArray` (described above) except that they return the normalized values instead.

And you can, of course, access the normalized name directly from a tag:

```php
echo $tag->normalized;
```



<a name="extending"></a>
## Extending Taggable

_Coming soon._



<a name="bugs"></a>
## Bugs, Suggestions and Contributions

Please use Github for bugs, comments, suggestions.

1. Fork the project.
2. Create your bugfix/feature branch and write your (well-commented) code.
3. Create unit tests for your code:
	- Run `composer install --dev` in the root directory to install required testing packages.
	- Add your test methods to `eloquent-taggable/tests/TaggableTest.php`.
	- Run `vendor/bin/phpunit` to the new (and all previous) tests and make sure everything passes.
3. Commit your changes (and your tests) and push to your branch.
4. Create a new pull request against the eloquent-sluggable `develop` branch.

> **Note:** You must create your pull request against the `develop` branch.



<a name="copyright"></a>
## Copyright and License

Eloquent-Taggable was written by Colin Viebrock and released under the MIT License. See the LICENSE file for details.

Copyright 2014 Colin Viebrock
