---
title: Revolt - The rock-solid event loop for PHP
permalink: /
layout: base
---
# Revolt 是什么？

Revolt 是用于 PHP 并发应用程序的健壮事件循环。通常 PHP
应用程序大部分时间都在等待 I/O。虽然 PHP
是单线程，[协作式多任务](https://zh.wikipedia.org/wiki/%E5%8D%8F%E4%BD%9C%E5%BC%8F%E5%A4%9A%E4%BB%BB%E5%8A%A1)用于通过使用等待时间允许并发做不同的事情。

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
