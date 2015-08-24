---
layout: post
title:  "PHPFastCGI Benchmarks - Symfony, Silex and Slim"
date:   2015-08-24 00:25:00
categories: general
---

## Introduction

I have conducted a series of benchmark tests to illustrate the performance increases achievable when using PHPFastCGI. All of these tests have been conducted in good faith.

My purpose is not to advise you which is the best framework to use, or even to convince you that using PHPFastCGI in your application would be a good idea. For many PHP developers, the number of requests per second that their application can handle is not important. My purpose is only to prove that if you wish to develop high performance PHP applications, PHPFastCGI (and the technique it uses) can help.

## Benchmarking System

Ubuntu 64-bit Server 15.04 run inside VMWare Fusion, NGINX, 2GB RAM and 4 processor cores.

For the control tests, the PHP FastCGI Process Manager (PHP-FPM) was used to interface the application being benchmarked with NGINX. For the PHPFastCGI tests, the appropriate framework adapter was used.

Whilst PHP-FPM and PHPFastCGI sound similar, there is a very important difference:

- PHP-FPM uses FastCGI to keep the PHP process alive between HTTP requests.
- PHPFastCGI uses FastCGI to keep the PHP process **and application** alive between HTTP requests.

Because of this difference, **applications using PHPFastCGI have to be developed very carefully**. Read this article on [things to consider when using PHPFastCGI with your application]({% post_url 2015-08-21-things-to-consider-using-phpfastcgi %}) if you wish to know more.

Six worker daemons were set up for the PHPFastCGI tests.

The 'ab' benchmarking tool was used to conduct the tests. It was configured for 5000 requests at a concurrency level of 20.

## The Tests

The first three tables show the benchmark results for simple 'Hello, World!' applications. The number of requests per second is a pretty useless figure here; what is more important is the reduction in the time taken to handle each request, as this indicates the amount of time shaved off the bootstrapping of the framework.


| Symfony            | Requests Per Second | Time Per Request (ms) |
|--------------------|---------------------|-----------------------|
| NGINX + PHP-FPM    | 584.88              | 1.71                  |
| NGINX + PHPFastCGI | 1701.97             | 0.59                  |

| Silex              | Requests Per Second | Time Per Request (ms) |
|--------------------|---------------------|-----------------------|
| NGINX + PHP-FPM    | 1252.08             | 0.799                 |
| NGINX + PHPFastCGI | 2701.78             | 0.37                  |

| Slim (v3)          | Requests Per Second | Time Per Request (ms) |
|--------------------|---------------------|-----------------------|
| NGINX + PHP-FPM    | 1654.26             | 0.605                 |
| NGINX + PHPFastCGI | 3740.75             | 0.267                 |

For a more realistic benchmark, I created a small 500 page website application using the Symfony framework. This has a single route configured which selects a random page from the database and renders it using Twig. What is interesting to note here is that the application is only marginally slower than the 'Hello, World!' Symfony application when using PHPFastCGI. I am planning on doing more investigation as to why this is, but I suspect that the framework is caching quite a lot in memory, though to what extent, I am unsure.

| 500 Page Website   | Requests Per Second | Time Per Request (ms) |
|--------------------|---------------------|-----------------------|
| NGINX + PHP-FPM    | 262.52              | 3.809                 |
| NGINX + PHPFastCGI | 1695.75             | 0.59                  |

## References

- [Speedfony Bundle](https://github.com/PHPFastCGI/SpeedfonyBundle) for Symfony
- [Speedex Package](https://github.com/PHPFastCGI/Speedex) for Silex
- [Slimmer Package](https://github.com/PHPFastCGI/Slimmer) for Slim v3
- [Speedfony Benchmarking App](https://github.com/PHPFastCGI/SpeedfonyBenchmarkingApp) (500 page website)

You may also be interested in the core [FastCGIDaemon Package](https://github.com/PHPFastCGI/FastCGIDaemon) which can be used to set up your web application with a different framework.
