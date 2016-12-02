---
layout: page
title: Quick Start
permalink: /quick-start/
---

## Introduction

Using PHPFastCGI, applications can stay alive between HTTP requests whilst operating behind the protection of a FastCGI enabled web server.

PHPFastCGI is a collection of several packages including a [core FastCGIDaemon package](http://github.com/PHPFastCGI/FastCGIDaemon) and easy integration adapters built for the [Symfony](http://github.com/PHPFastCGI/SpeedfonyBundle), [Silex](http://github.com/PHPFastCGI/SilexAdapter) and [Slim](http://github.com/PHPFastCGI/SlimAdapter) frameworks.

## Warning

Before doing this, please read the post on [things to consider when using PHPFastCGI with your application]({% post_url 2015-08-21-things-to-consider-using-phpfastcgi %}).

## Usage

Below is an example of a simple 'Hello, World!' FastCGI application in PHP using the core FastCGIDaemon package:

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

## Server Configuration

### NGINX

With NGINX, you need to use a process manager such as [supervisord](http://supervisord.org/) to manage instances of your application. Have a look at [AstroSplash](http://astrosplash.com/) for an [example supervisord configuration](https://github.com/AndrewCarterUK/AstroSplash/blob/master/supervisord.conf).

Below is an example of the modification that you would make to the [Symfony NGINX configuration](https://www.nginx.com/resources/wiki/start/topics/recipes/symfony/). The core principle is to replace the PHP-FPM reference with one to a cluster of workers.

{% highlight conf %}
# This shows the modifications that you would make to the Symfony NGINX configuration
# https://www.nginx.com/resources/wiki/start/topics/recipes/symfony/

upstream workers {
    server localhost:5000;
    server localhost:5001;
    server localhost:5002;
    server localhost:5003;
}

server {
    # ...

    location ~ ^/app\.php(/|$) {
        # ...
        fastcgi_pass workers;
        # ...
    }

    # ...
}
{% endhighlight %}

### Apache

If you wish to configure your FastCGI application to work with the apache web server, you can use the apache FastCGI module to process manage your application.

This can be done by creating a FCGI script that launches your application and inserting a FastCgiServer directive into your virtual host configuration.

Here is an example `script.fcgi`:

{% highlight bash %}
#!/bin/bash
php /path/to/application.php run
{% endhighlight %}

Or with Symfony:

{% highlight bash %}
#!/bin/bash
php /path/to/bin/console speedfony:run --env=prod
{% endhighlight %}

In your configuration, you can use the [FastCgiServer](https://web.archive.org/web/20150913190020/http://www.fastcgi.com/mod_fastcgi/docs/mod_fastcgi.html#FastCgiServer) directive to inform Apache of your application.

{% highlight conf %}
FastCgiServer /path/to/script.fcgi
{% endhighlight %}

By default, the daemon will listen on FCGI_LISTENSOCK_FILENO, but it can also be configured to listen on a TCP address. For example:


{% highlight bash %}
#!/bin/bash
php /path/to/application.php run --port=5000 --host=localhost
{% endhighlight %}

Or with Symfony:

{% highlight bash %}
#!/bin/bash
php /path/to/bin/console speedfony:run --env=prod --port=5000 --host=localhost
{% endhighlight %}
