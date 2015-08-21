---
layout: page
title: Quick Start
permalink: /quick-start/
---

## Introduction

Using PHPFastCGI, applications can stay alive between HTTP requests whilst operating behind the protection of a FastCGI enabled web server.

PHPFastCGI is a collection of several packages including a core FastCGIDaemon package and packages built for easy integration with the Symfony, Silex and Slim frameworks.

## Warning

Before doing this, please read [the post on whether you should use PHPFastCGI with your application]({% post_url 2015-08-21-should-I-use-phpfastcgi %}).

## Usage

Below is an example of a simple 'Hello, World!' FastCGI application in PHP using the core FastCGIDaemon package:

{% highlight php startinline=true %}
<?php // command.php

// Include the composer autoloader
require_once dirname(__FILE__) . '/../vendor/autoload.php';

use PHPFastCGI\FastCGIDaemon\ApplicationFactory;
use Psr\Http\Message\ServerRequestInterface;
use Zend\Diactoros\Response\HtmlResponse;

// A simple kernel. This is the core of your application
$kernel = function (ServerRequestInterface $request) {
    return new HtmlResponse('<h1>Hello, World!</h1>');
};

// Create your Symfony console application using the factory
$application = (new ApplicationFactory)->createApplication($kernel);

// Run the Symfony console application
$application->run();
{% endhighlight %}

If you wish to configure your FastCGI application to work with the apache web server, you can use the apache FastCGI module to process manage your application.

This can be done by creating a FCGI script that launches your application and inserting a FastCgiServer directive into your virtual host configuration.

{% highlight bash %}
#!/bin/bash
php /path/to/command.php run
{% endhighlight %}

{% highlight bash %}
FastCgiServer /path/to/web/root/script.fcgi
{% endhighlight %}

By default, the daemon will listen on FCGI_LISTENSOCK_FILENO, but it can also be configured to listen on a TCP address. For example:

{% highlight bash %}
php /path/to/command.php run --port=5000 --host=localhost
{% endhighlight %}

If you are using a web server such as nginx, you will need to use a process manager to monitor and run your application.
