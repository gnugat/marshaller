# Marshaller [![SensioLabsInsight](https://insight.sensiolabs.com/projects/e3f1b42c-e796-40c8-bfb1-299a79983d2e/mini.png)](https://insight.sensiolabs.com/projects/e3f1b42c-e796-40c8-bfb1-299a79983d2e) [![Travis CI](https://travis-ci.org/gnugat/marshaller.png)](https://travis-ci.org/gnugat/marshaller)

A PHP library that converts from one format to another.

Marshaller doesn't try to guess how to convert the given input, instead it relies
on `MarshallerStrategies` implemented by the developers: it gives us full control
on the output formats.

To automatically convert an input into specific formats (like XML, JSON or YAML),
it might be better to use other tools (e.g. [JMS Serializer](http://jmsyst.com/libs/serializer)).

## Installation

Marshaller can be installed using [Composer](http://getcomposer.org/):

    composer require "gnugat/marshaller:~2.0"

## Simple conversion

Let's take the following object:

```php
<?php

class Article
{
    public function __construct($title, $content)
    {
        $this->title = $title;
        $this->content = $content;
    }

    public function getTitle()
    {
        return $this->title;
    }

    public function getContent()
    {
        return $this->content;
    }
}

$article = new Article('Nobody expects...', '... The Spanish Inquisition!');
```

If we want to convert it to the following format:

```php
// ...

array(
    'title' => 'Nobody expects...',
    'content' => '... The Spanish Inquisition!',
);
```

Then we have first to create a `MarshallerStrategy`:

```php
// ...

require __DIR__.'/vendor/autoload.php';

use Gnugat\Marshaller\MarshallerStrategy;

class ArticleMarshaller implements MarshallerStrategy
{
    public function supports($toMarshal, $category = null)
    {
        return $toMarshal instanceof Article;
    }

    public function marshal($toMarshal)
    {
        return array(
            'title' => $toMarshal->getTitle(),
            'content' => $toMarshal->getContent(),
        );
    }
}
```

The second step is to register it in `Marshaller`:

```php
// ...

use Gnugat\Marshaller\Marshaller;

$marshaller = new Marshaller();
$marshaller->add(new ArticleMarshaller());
```

Finally we can actually convert the object:

```php
// ...

$marshalledArticle = $marshaller->marshal($article);
```

## Partial conversion

If we need to convert `Article` into the following partial format:

```php
// ...

array('title' => 'Nobody expects...');
```

Then we first have to define a new `MarshallStrategy`:

```php
// ...

class PartialArticleMarshaller implements MarshallStrategy
{
    public function supports($toMarshal, $category = null)
    {
        return $toMarshal instanceof Article && 'partial' === $category;
    }

    public function marshal($toMarshal)
    {
        return array(
            'title' => $toMarshal->getTitle(),
        );
    }
}
```

Since this `MarshallerStrategy` has a more restrictive `support` condition, we'd
like it to be checked before `ArticleMarshaller`. This can be done by registering
`PartialArticleMarshaller` with a higher priority than `ArticleMarshaller`
(in this case with a priority higher than 0):

```php
// ...

$marshaller->add(new PartialArticleMarshaller, 1);
```

Finally we can call `Marshaller`, for the `partial` category:

```php
$marshaller->marshal($article, 'partial');
```

## Collection conversions

In order to avoid this:

```php
// ...

$articles = array($article);
foreach ($articles as $article) {
    $marshaller->marshal($article);
}
```

We can use the following short cut method:

```php
// ...

$marshaller->marshalCollection($articles);
```

## Further documentation

You can see the current and past versions using one of the following:

* the `git tag` command
* the [releases page on Github](https://github.com/gnugat/marshaller/releases)
* the file listing the [changes between versions](CHANGELOG.md)

You can find more documentation at the following links:

* [copyright and MIT license](LICENSE)
* [versioning and branching models](VERSIONING.md)
* [contribution instructions](CONTRIBUTING.md)
