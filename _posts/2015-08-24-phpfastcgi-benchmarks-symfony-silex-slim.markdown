---
layout: post
title:  "PHPFastCGI Benchmarks - Symfony, Silex and Slim"
date:   2015-08-24 00:25:00
categories: general
---

## Introduction

I have conducted a series of benchmark tests to illustrate the performance increases achievable when using the PHPFastCGI project. All of these tests have been conducted in good faith.

My purpose is not to advise you which is the best framework to use, or even to convince you that using PHPFastCGI in your application would be a good idea. **For many PHP developers, the number of requests per second that their application can handle is not important**. My purpose is only to prove that if you wish to develop high performance PHP applications, PHPFastCGI (and the technique it uses) can help.

## Benchmarking System

Running inside VMWare Fusion: Ubuntu 64-bit Server 15.04, NGINX, PHP 5.6.4, 2GB RAM and 4 cores (Intel Core i7, 3.4 GHz).

For each of the frameworks, the application was initially benchmarked using the PHP FastCGI Process Manager (PHP-FPM) with the OPcache enabled.

For the PHPFastCGI benchmarks, a command line application was created for each of the frameworks using the appropriate adapter. Six instances of each application were then launched and NGINX's inbuilt load balancer was used in round-robin mode. The benchmarks for the PHPFastCGI project were run twice, firstly using the PHP implementation of the FastCGI protocol and then using the php5-fastcgi extension.

Whilst PHP-FPM and PHPFastCGI sound similar, there is a very important difference:

- PHP-FPM uses FastCGI to keep the PHP process alive between HTTP requests.
- PHPFastCGI uses FastCGI to keep the PHP process **and application** alive between HTTP requests.

Because of this difference, **applications using PHPFastCGI have to be developed very carefully**. Read this article on [things to consider when using PHPFastCGI with your application]({% post_url 2015-08-21-things-to-consider-using-phpfastcgi %}) if you wish to know more.

The 'ab' benchmarking tool was used to conduct the tests. It was configured for 50000 requests at a concurrency level of 20.

## The Tests

The first three tables show the benchmark results for simple 'Hello, World!' applications. The number of requests per second is a pretty useless figure here; what is more important is the **reduction in the time taken to handle each request**, as this indicates the amount of time shaved off the bootstrapping of the framework.


| Symfony                  | Requests Per Second | Time Per Request (ms) |
|--------------------------|---------------------|-----------------------|
| NGINX + PHP-FPM          | 551.59              | 1.813                 |
| NGINX + PHPFastCGI       | 1739.16             | 0.575                 |
| NGINX + PHPFastCGI (ext) | 2469.36             | 0.405                 |

| Silex                    | Requests Per Second | Time Per Request (ms) |
|--------------------------|---------------------|-----------------------|
| NGINX + PHP-FPM          | 1424.30             | 0.702                 |
| NGINX + PHPFastCGI       | 2332.22             | 0.429                 |
| NGINX + PHPFastCGI (ext) | 4148.08             | 0.241                 |

| Slim (v3)                | Requests Per Second | Time Per Request (ms) |
|--------------------------|---------------------|-----------------------|
| NGINX + PHP-FPM          | 2058.91             | 0.486                 |
| NGINX + PHPFastCGI       | 2734.21             | 0.366                 |
| NGINX + PHPFastCGI (ext) | 5562.47             | 0.180                 |

In general, **if the application is doing more there are greater potential time savings to be had**. This is why the increase in performance of the micro-frameworks is less significant than that of the full stack Symfony framework. This is also why 'Hello, World!' benchmarks are actually the **least impressive** benchmarks for the PHPFastCGI project.

For a more realistic benchmark, I created a small 500 page website application using the Symfony framework. This has a single route configured that selects a random page from the database (MySQL) and renders it using Twig. The Doctrine entity repository is cleared after each request, so the framework is not caching the results of the queries. A much greater performance can be achieved by not doing this, but the purpose of this benchmark is to test when the application has actual work to do - so I have not cheated.

| 500 Page Website         | Requests Per Second | Time Per Request (ms) |
|--------------------------|---------------------|-----------------------|
| NGINX + PHP-FPM          | 282.55              | 3.539                 |
| NGINX + PHPFastCGI       | 1334.64             | 0.749                 |
| NGINX + PHPFastCGI (ext) | 1769.94             | 0.565                 |

## Author Notes

Whilst I hope you agree that the result of these benchmarks is impressive, there is more work to be done! This project is still not stable, and I welcome any contributions that you may have. In particular, the [php5-fastcgi extension](https://github.com/PHPFastCGI/php5-fastcgi) would benefit from a review by someone with a knowledge of PHP internals.

Also, if you are attending ConFoo and would like to hear more about this project then [please vote for my talk proposal](http://confoo.ca/en/call-for-papers/speaker/andrew-carter).

## References

- [Speedfony Bundle](https://github.com/PHPFastCGI/SpeedfonyBundle) for Symfony
- [Silex Adapter](https://github.com/PHPFastCGI/SilexAdapter)
- [Slim Adapter](https://github.com/PHPFastCGI/SlimAdapter)
- [Speedfony Benchmarking App](https://github.com/PHPFastCGI/SpeedfonyBenchmarkingApp) (500 page website)

You may also be interested in the core [FastCGIDaemon Package](https://github.com/PHPFastCGI/FastCGIDaemon), which can be used to create FastCGI applications without a framework (or with a different one).

The [php5-fastcgi extension](https://github.com/PHPFastCGI/php5-fastcgi) is still being developed and tested. Thus, there have been no releases of this software yet and the integration of this extension into the FastCGIDaemon core can only be found on the [php5-fastcgi-ext branch](https://github.com/PHPFastCGI/FastCGIDaemon/tree/php5-fastcgi-ext) of the repository.

_This article was revised on 29th August 2015 to contain more accurate benchmarks. Changes include turning on the opcache and significant changes to the FastCGIDaemon core._

_This article was revised again on 9th September 2015 to include benchmarks using the php5-fastcgi extension currently under development._
