---
layout: post
title: Creating REST Service with PHP from a Java programmer's perspective
author:
  display_name: paawak
  login: paawak
  email: paawak@gmail.com
  url: ''
author_login: paawak
author_email: paawak@gmail.com
date: '2020-12-23 23:21:25 +0530'
categories:
- php
tags:
- php
- rest
---
# Prelude
My brush with PHP was quite accidental. I was a young Java programmer way back in 2002, and a proud one. A friend and mentor, was playing around with this crazy thing, that took hours of effort to get installed on Windows (I did not know Linux very well that time). First you install Apache Server, and then PHP extension on it. That, as I said was quite an effort. However, he did it so many times, that he could do it almost in his sleep. I got curious and started playing around with it too. Its very C-like syntax made it very easy to learn. But its total lack of type safety and dynamic nature, made me love to hate it. It was 2002 and, we were on PHP 4, I believe. When I look back, I feel, I was lucky that I learnt PHP: it gave me my first major break. Though I was not an expert in PHP, at some point, within a couple of years, I had become comfortable with PHP, and had gained a good insight into its working. However, since I always belonged to the Java world, I could never completely love PHP. So, probably after 2005, I almost completely stopped coding in PHP, to focus on Java.

Fast forward to 2020. During the brutal lockdown due to Covid 19, we were cooped up like chicken, not able to venture out much. I was trying to do something with Tesseract in Bangla language. I thought of creating a [website](http://ocr.paawak.me/). Now, though the [backend](https://github.com/paawak/porua-ocr-service) was written in Java, hosting it would be a rather costly affair. So, I decided to re-write some parts of it in [PHP](https://github.com/paawak/porua-correction-service).

Thus, I had to kind of re-discover my old friend PHP.

In the span of roughly one and a half decades, things have drastically changed. PHP 8 has just been released, and in terms of features and frameworks, it can match Java to a large extent.

In this post, I will highlight how to write a fully featured, enterprise grade REST Service using PHP. This will have a Dependency Injection Framework, a ORM Framework etc.

# What are we building?
We would be building a simple REST Framework for adding and retrieving Books. Each book would have an Author and a Genre.

## Choosing a Package Manager
[Composer](https://getcomposer.org/) is the ubiquitous dependency manager, or in Java terms, a package manager. Well, its really very similar to *npm* in the JavaScript world. So we would use Composer to manage our package dependencies.

Well its very simple: you define a *composer.json* in your root folder. A typical one looks like this:

```json
{
    "name": "paawak-blog/rest-service-with-php-slim4",
    "description": "Example of a REST Service with PHP Slim4",
    "license": "proprietary",
    "require": {
        "slim/slim": "4.*",
        "slim/psr7": "^1.1",
        "doctrine/orm": "^2.7",
        "monolog/monolog": "^2.1",
        "php-di/php-di": "^6.2",
        "php-di/slim-bridge": "^3.0"
    }
}
```

## Choosing a framework
PHP has many mature frameworks. I chose [Slim](https://www.slimframework.com/), which is a lightweight, no-frills framework for writing REST services. As of this writing, the latest version is [Slim4](https://www.slimframework.com/docs/v4/). Slim4 supports (PSR-7)[https://www.php-fig.org/psr/psr-7/] interfaces for its Request/Response objects. We would be using PSR7 interfaces wherever possible, so that we are not tightly coupled to Slim framework.

Add the below lines to your *composer.json* to add Slim4 dependency:

```json
{
    "name": "paawak-blog/rest-service-with-php-slim4",
    "description": "Example of a REST Service with PHP Slim4",
    "license": "proprietary",
    "require": {
        "slim/slim": "4.*",
        "slim/psr7": "^1.1",
...
    }
}
```

Note that we need to add PSR7 as well.

## Dependency Injection
Coming from Java background, I started looking for a good Dependency Injection Framework. I found [PHP DI](https://php-di.org/) to be pretty good. To use it with Slim4, we would need the [DI Slim Bridge](https://php-di.org/doc/frameworks/slim.html). We would need to add the below lines to our *composer.json*:

```json
{
    "name": "paawak-blog/rest-service-with-php-slim4",
    "description": "Example of a REST Service with PHP Slim4",
    "license": "proprietary",
    "require": {
...
        "php-di/php-di": "^6.2",
        "php-di/slim-bridge": "^3.0"
    }
}
```

### DI Goals
We would like to achieve the below objectives:
  1. __Code to interface:__ This is our primary focus. We would like to define an interface and tie it to its implementation class, so that it can either be autowired, or fetched from the *DI Container*, given its interface.
  2. __Environment configuration details:__ We would like to store our environment specific configuration details like DB username/password, so that we could retrieve it using a *key*.

### Basic concepts of PHP-DI
At the heart of the *PHP-DI* is the *container*, which is analogous to the *ApplicationContext* in Spring. *PHP-DI* works with PSR's *Psr\Container\ContainerInterface* interface, which ensures that our code is not too much tied to the *PHP-DI Framework*, but more loosely coupled. The *PHP-DI* defines a builder called *ContainerBuilder*, which helps us to create an instance of *ContainerInterface*. The *ContainerBuilder* has an utility method *addDefinitions()*, which takes in an *Associative Array*. This Array (basically *Map* in Java), has the *key* as either a *string* or a *class*, and the *value* as an *Object*.

After we obtain an instance of the *ContainerInterface*, we could pass it around and obtain the various *objects* that we have defined by their respective *keys*. We would also use the *ContainerInterface* to obtain an instance of Slim's *App*, so that autowiring works seamlessly.

We would demo these in the subsequent sections.

### Bean definition
We define beans as simple *Associative Arrays*. The below file *LocalApplicationConfig.php*, defines environment specific configuration parameters as shown below:

```php
return [
    'database.host' => 'localhost',
    'database.name' => 'rest_service_with_php_slim4',
    'database.driver' => 'pdo_mysql',
    'cors.allow-origin' => '*',
    'logger.fileName' => '../logs/rest-service-with-php-slim4.log',
    'logger.maxFiles' => 10,
    'logger.console' => true
];
```

We could do a more elegant definition, where we tie an interface to its implementation. This is done in the file *DIConfiguration.php* as shown below:

```php
return [
MyAwesomInterface::class => function (ContainerInterface $container) {
    $awesome = new MyAwesomInterfaceImpl($container->get('myKey_1'), $container->get('myKey_2'));
    return $awesome;
}
];
```

Note that we are passing in the *ContainerInterface*, and getting our other beans from there as well.

### Creating the DI Container
We would use the *ContainerBuilder* to obtain an instance of the *ContainerInterface* as shown below:

```php
$containerBuilder = new ContainerBuilder;
$containerBuilder->addDefinitions(__DIR__ . '/DIConfiguration.php');

$containerBuilder->addDefinitions(__DIR__ . '/LocalApplicationConfig.php');

$container = $containerBuilder->build();

return $container;
```

### Creating the Slim App
The final piece in the puzzle is creating Slim's *App* using the instance of the freshly minted *ContainerInterface*, so that DI works seamlessly with autowiring. This is how it is done:

```php
use DI\Bridge\Slim\Bridge;

$app = Bridge::create($container);
```
