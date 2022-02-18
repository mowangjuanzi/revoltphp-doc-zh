---
title: Revolt - The rock-solid event loop for PHP
permalink: /
layout: base
---
# Revolt 是什么？

Revolt is a rock-solid event loop for concurrent PHP applications.
The usual PHP application spends most of its time waiting for I/O.
While PHP is single threaded, [cooperative multitasking](https://en.wikipedia.org/wiki/Cooperative_multitasking) can be used to allow for concurrency by using the waiting time to do different things.

PHP's traditional synchronous execution flow is easy to understand. Doing one thing at a time.
If you query a database, you send the query and wait for a response from the database server.
Once you have the response, you can start doing the next thing.

Amp, ReactPHP, and other libraries have offered cooperative multitasking in PHP for a long time.
However, their event-driven nature was incompatible to many existing interfaces and required a different thinking model.
PHP 8.1 ships with fibers built-in, which offers cooperative multi-threading.
Calls can be asynchronous without promises or callbacks, while still allowing for non-blocking I/O.

Every application making use of cooperative multitasking needs a single scheduler (also called event loop), which this package provides.
Revolt is the result of combining years of experience of Amp's and ReactPHP's event loop implementations.
However, it is not a full-blown framework for writing concurrent PHP applications, but only provides what's necessary as a common base.
Different (strongly) opinionated libraries can be built on top of it and both Amp and ReactPHP will continue to co-exist.

## 安装

It may surprise people to learn that the PHP standard library already has everything we need to write event-driven and non-blocking applications.
This package can be installed as a [Composer](https://getcomposer.org/) dependency on PHP 8 and later.
PHP 8.1 ships with fibers built-in, but users on PHP 8.0 can install [`ext-fiber`](https://github.com/amphp/ext-fiber) with almost identical behavior.

```bash
composer require revolt/event-loop
```

{:.small-hint}
Applications with many concurrent file descriptors require one of the [extensions](/extensions).

→&nbsp;&nbsp;[开始](/fundamentals)
