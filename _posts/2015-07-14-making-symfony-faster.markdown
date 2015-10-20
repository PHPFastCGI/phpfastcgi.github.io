---
layout: post
title:  "How to make Symfony faster"
date:   2015-07-14 10:00:00
categories: general
---

By turning your Symfony application into a FastCGI application, you can keep the application in memory between request cycles.

Before doing this, please read the post on [things to consider when using PHPFastCGI with your application]({% post_url 2015-08-21-things-to-consider-using-phpfastcgi %}).

To do this, open the terminal in your project directory and run the following command:

{% highlight bash %}
php composer.phar require "phpfastcgi/speedfony-bundle:~0.7"
{% endhighlight %}

This command adds the Speedfony Bundle to your composer dependencies.

Now you need to register the bundle in your AppKernel.php file:
{% highlight php startinline=true %}
// app/AppKernel.php

// ...
class AppKernel extends Kernel
{
  public function registerBundles()
  {
    $bundles = array(
      // ...
      new PHPFastCGI\SpeedfonyBundle\PHPFastCGISpeedfonyBundle(),
    );

    // ...
  }
// ...
{% endhighlight %}

Your application can now be run as a FastCGI application listening on FCGI_LISTENSOCK_FILENO (stdin) using this command:

{% highlight bash %}
php app/console speedfony:run --env="prod"
{% endhighlight %}

You can also configure the FastCGI application to listen on a TCP address using the 'port' and 'host' options:

{% highlight bash %}
php app/console speedfony:run --env="prod" --port 5000
{% endhighlight %}
