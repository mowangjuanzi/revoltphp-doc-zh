---
title: Revolt - The rock-solid event loop for PHP
permalink: /
layout: base
---
# Revolt 是什么？

Revolt 是健壮的事件循环，用于并发 PHP 应用程序。通常 PHP
应用程序大部分时间都在等待 I/O。虽然 PHP
是单线程，[协作式多任务](https://zh.wikipedia.org/wiki/%E5%8D%8F%E4%BD%9C%E5%BC%8F%E5%A4%9A%E4%BB%BB%E5%8A%A1)用于通过使用等待时间允许并发做不同的事情。

PHP 的传统同步执行流程很容易理解。一次只做一件事。如果查询数据库，则发送查询并等待数据库服务器的响应。一旦得到响应，就可以开始做下一件事了。

Amp，ReactPHP 和其他库长期以来一直在 PHP 中提供协作式多任务处理。但是，本质上它们的事件驱动跟许多现有接口不兼容，需要有不同的思维模式。PHP 8.1
内置纤程（Fiber），提供了多线程协作。调用可以是异步的，无需 promises 或者 callback，同时仍允许非堵塞 I/O。

每个使用协作式多任务的应用程序都需要一个单独的调度器（也称为事件循环），这个包提供了。Revolt 
是 Amp 和 ReactPHP 多年事件循环实现经验相结合的结果。然而，它并不是用于编写并发 PHP 
应用程序的成熟框架，而只是提供了必要的公共基础。可以基于它构建强有力的库，并且 Amp
和 ReactPHP 还将继续共存。

## 安装

这会让很多人惊喜的发现 PHP 标准库已经拥有了所需要的编写事件驱动和非堵塞应用程序所需要的一切。使用 PHP 8 和更高版本的 PHP 通过
[Composer](https://getcomposer.org/) 安装该包。
PHP 8.1 内置纤程，但 PHP 8.0 的用户可以安装 [`ext-fiber`](https://github.com/amphp/ext-fiber)，其行为几乎相同。

```bash
composer require revolt/event-loop
```

{:.small-hint}
存在许多并发文件描述符的应用程序需要一个[扩展](/extensions)。

→&nbsp;&nbsp;[开始](/fundamentals)
