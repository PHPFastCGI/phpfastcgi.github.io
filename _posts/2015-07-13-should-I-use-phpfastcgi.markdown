---
layout: post
title:  "Should I use PHPFastCGI with my application?"
date:   2015-07-13 20:00:00
categories: general
---


## Introduction 

Let me start with something very important.

**Not every application is a suitable candidate to become a FastCGI application.**

In conventional PHP applications, HTTP requests are nicely separated from each other. Web servers handle each HTTP request as a different execution of an application. This provies a degree of **safety** and **security**. If you choose to run your PHP application as a FastCGI application, you forefit this separation for an increase in performance.

## Fatal Errors and Exceptions

A properly written application should not respond to clients with uncaught errors and exceptions. The consequences, however, of doing this in a FastCGI application are much steeper.

A fatal error will cause your application to terminate and stop listening for requests. Something will need to restart your application otherwise it will stay down.

The best practice is to make sure that all errors and exceptions are appropriately caught and logged without terminating the application. You should also make sure that your application is being managed by a suitable process manager so that if it does crash, it will be promptly restarted.

Apache's [mod_fastcgi][mod_fastcgi] includes a process manager that is very easy to configure. If you are using a web server such as NGINX, you may need to use a process manager such as [supervisord][supervisord].

## Memory Segregation

Static memory and global variables are well documented enemies. If you intend to create a FastCGI application, you should avoid these like the plague.

Any scenario where a HTTP request could influence, interfere or read memory that is shared by a later (or earlier) HTTP request is an opportunity for an attacker.

[mod_fastcgi]: http://www.fastcgi.com/mod_fastcgi/docs/mod_fastcgi.html "Apache module: mod_fastcgi"
[supervisord]: http://supervisord.org/                                  "Supervisor: A process control system:"