---
layout: post
title:  "Things to consider when developing an application with PHPFastCGI"
date:   2015-08-21 09:00:00
categories: general
---

## Introduction 

Let me start with something very important.

**Not every application is a suitable candidate to become a FastCGI application.**

In conventional PHP applications, HTTP requests are nicely separated from each other. Web servers handle each HTTP request as a different execution of an application. This provides a degree of **safety** and **security**. If you choose to run your PHP application as a FastCGI application, you forefit this for an increase in performance.

## Fatal Errors and Exceptions

A properly written application should not respond to clients with uncaught errors and exceptions. The consequences, however, of doing this in a FastCGI application are much steeper.

A fatal error will cause your application to terminate and stop listening for requests. Something will need to restart your application otherwise it will stay down.

The best practice is to make sure that all errors and exceptions are appropriately caught and logged without terminating the application. You should also make sure that your application is being managed by a suitable process manager so that if it does crash, it will be promptly restarted.

Apache's [mod_fastcgi][mod_fastcgi] includes a process manager that is very easy to configure. If you are using a web server such as NGINX, you may need to use a process manager such as [supervisord][supervisord].

## Code Updates and Deployment

Usually when we make changes to our application, these are recognised immediately as PHP rereads our file every time it executes it. In some scenarios, we may have to flush caches but that is about as tricky as it usually gets. When working with long running processes, we have to make sure that we shutdown and reload our application when we update its code.

This can be an easy step to forget! Many times I have found myself staring blankly at error messages before remembering that I needed to reload the application whilst developing this very project!

I am currently working on features to ensure that shutdown signals cause PHPFastCGI applications to exit cleanly after they have finished handling and responding to all open requests.

When running multiple instances of your FastCGI application, you should desire a configuration that allows you to shutdown and restart instances individually to prevent down time. Running multiple instances of your application is also recommended for performance reasons.

If you are using apache's [mod_fastcgi][mod_fastcgi] to process manage your application, a graceful reload of the web server may be your best option.

## Memory Segregation

Static memory and global variables are well documented enemies. If you intend to create a FastCGI application, you should avoid these like the plague (that includes $_SESSION, $_POST, $_GET, etc...).

Any scenario where a HTTP request could influence, interfere or read memory that is shared by a later (or earlier) HTTP request is an opportunity for an attacker.

## Memory Leaks

Although this is changing, PHP applications and frameworks are not usually designed to run for long periods of time. Anyone who has ever worked with a low level language (such as C or C++) will understand that the traditional PHP developer approach to memory allocation is incredibly lacking in any form of principal or organization.

This is an approach that we cannot afford to take when developing long running applications. PHP itself is now very good at cleaning up memory automatically once it is no longer required. Nowadays, most memory leaks in our PHP applications can be traced to dangling references either inside the application or within its dependencies.

For example, Doctrine's SQL logger (enabled by default in the Symfony development environment) can very quickly cause a resource issue. If you are using the Doctrine Entity Manager, you must also take care to use the EntityManager::clear() method provided and remove references to entities that you no longer require it to manage.

One potential approach for handling memory leaks is to restart application instances after they have been operating for a certain amount of time or handled a certain number of requests. This feature is scheduled for integration into the FastCGIDaemon core package and will be available before the first stable release. However, it is still considered best practice to try and develop applications without memory leaks.

## Connection Timeouts

One of the advantages of using PHPFastCGI to keep your application alive is not having to reconnect to databases, cache layers and message queues for every HTTP request. However, should any of these connections timeout or otherwise disconnect, your application must contain logic to restablish these. Failure to do so will leave you with 5XX error codes and possibly useless instances of your application running.

This is a particularly nasty bug, as you may not notice it until you have deployed to your production server and gone though a quiet patch in the early hours of the morning.

## Conclusion

There is lots to be gained and lots to be lost by daemonizing your PHP applications. It is very important that you are aware of the potential issues you could face and how to mitigate these. However, with a properly designed and carefully considered application - you can reach response speeds well beyond the dreaming limits of conventional PHP applications. 

[mod_fastcgi]: http://www.fastcgi.com/mod_fastcgi/docs/mod_fastcgi.html "Apache module: mod_fastcgi"
[supervisord]: http://supervisord.org/                                  "Supervisor: A process control system:"
