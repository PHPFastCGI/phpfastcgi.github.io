---
layout: post
title:  "280 rq/s to 1770 rq/s - Speed up Symfony, Silex and Slim with PHPFastCGI"
date:   2015-11-04 01:00:00
categories: general
---

### The "What"

Let me start by clearing up a possible source of confusion for those of you familiar with FastCGI. This project is not about PHP-FPM. PHP-FPM is a great way of improving the performance of PHP applications, but the way it works is different to PHPFastCGI.

PHP-FPM keeps the **PHP interpreter** alive between HTTP request cycles.

PHPFastCGI keeps the **PHP application** alive between HTTP request cycles.

That's all of your services, configuration... everything.

### The "Why"

For a simple 500 page Symfony application, PHPFastCGI can take the performance from 280 rq/s to 1770 rq/s. If you're interested, check the [post on how the benchmarks were conducted]({ post_url 2015-08-24-phpfastcgi-benchmarks-symfony-silex-slim %}). To summarize, I tried to make the Symfony application as fast as I could (OPCache enabled, PHP-FPM, NGINX). Then, I ran it as a PHPFastCGI application and compared the performance.

Please do not take those benchmarks too seriously (though I believe that they are significant). The top results were collected using a PHP extension that is currently lacking a few features (and not as stable as the default PHP implementation of the protocol). Also, a very helpful commenter has pointed out some ways that I could improve the benchmarks. As a result, I will be re-running them shortly.

Regardless, any application that has to reload itself on **every HTTP request cycle** will be significantly slower than a version of the same application that does not.

### The "How"

Before doing this with an application, please read the post on [things to consider when using PHPFastCGI with your application]({% post_url 2015-08-21-things-to-consider-using-phpfastcgi %}).

If your still think that your application is ready to be run in this way, it is **really easy** to create a FastCGI application using PHPFastCGI. With a Symfony application, you can do it by just [installing a bundle]({% post_url 2015-07-14-making-symfony-faster %}).

The project also includes a [Silex Adapter](http://github.com/PHPFastCGI/SilexAdapter) and a [Slim Adapter](http://github.com/PHPFastCGI/SlimAdapter). These provide wrappers for the core application classes of each framework.

You can also use the project without a framework, using the [core FastCGIDaemon](http://github.com/PHPFastCGI/FastCGIDaemon).

A framework-less PHPFastCGI application looks like this:

{% highlight php startinline=true %}
<?php // command.php

// Include the composer autoloader
require_once dirname(__FILE__) . '/../vendor/autoload.php';

use PHPFastCGI\FastCGIDaemon\ApplicationFactory;
use PHPFastCGI\FastCGIDaemon\Http\RequestInterface;
use Zend\Diactoros\Response\HtmlResponse;

// A simple kernel. This is the core of your application
$kernel = function (RequestInterface $request) {
    // $request->getServerRequest()         returns PSR-7 server request object
    // $request->getHttpFoundationRequest() returns HTTP foundation request object
    return new HtmlResponse('<h1>Hello, World!</h1>');
};

// Create your Symfony console application using the factory
$application = (new ApplicationFactory)->createApplication($kernel);

// Run the Symfony console application
$application->run();
{% endhighlight %}

### References

- [FastCGIDaemon](http://github.com/PHPFastCGI/FastCGIDaemon)
- [Speedfony Bundle](http://github.com/PHPFastCGI/SpeedfonyBundle)
- [Slim v3 Adapter](http://github.com/PHPFastCGI/SlimAdapter)
- [Silex Adapter](http://github.com/PHPFastCGI/SilexAdapter)

If you have any suggestions or queries, please use the comment section below, [GitHub](http://github.com/PHPFastCGI/FastCGIDaemon) or [Twitter](http://twitter.com/AndrewCarterUK).

Also - as a cheeky request - if you like the sound of this project it would make me very happy if you shared it with your developer friends.
