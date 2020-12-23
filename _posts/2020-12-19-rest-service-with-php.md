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
PHP has many mature frameworks.
