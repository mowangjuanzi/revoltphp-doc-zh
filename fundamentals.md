---
title: 原理
permalink: /fundamentals
layout: base
---
# 原理

每个使用协作式多任务的应用程序都只能有一个调度程序。同时运行两个事件循环没有意义，因为它们需要以繁忙等待的方式相互调度，浪费
CPU 周期。

Revolt 在 `Revolt\EventLoop` 上的方法提供了对调度程序的全局访问。第一次使用该类时，会自动创建[最佳自定义驱动](/extensions)。`Revolt\EventLoop::setDriver()` 
可用于设置定义驱动。

没有办法重置调度程序。
While this might seem useful for test isolation, it would break as soon as test frameworks start to make use of Revolt themselves.
Test frameworks and test code need to share the same scheduler, as there can only be one.

Cooperative multitasking works by telling the scheduler which events we're interested in.
The scheduler will invoke a callback once the event happened.
Revolt 支持以下事件：

 - [**Defer**](/timers)
 
   {:.small-hint .mt-n2 .mb-1}
   回调在事件循环的下一次迭代中执行。 If there are defers scheduled, the event loop won't wait between iterations.
 - [**Delay**](/timers)
 
   {:.small-hint .mt-n2 .mb-1} 
   回调在指定秒数后执行。 Fractions of a second may be expressed as floating point numbers.
 - [**Repeat**](/timers)
 
   {:.small-hint .mt-n2 .mb-1}
   回调重复在指定秒数后执行。 Fractions of a second may be expressed as floating point numbers.
 - [**Stream readable**](/streams)
 
   {:.small-hint .mt-n2 .mb-1}
   The callback is executed when there's data on the stream to be read, or the connection closed.
 - [**Stream writable**](/streams)
 
   {:.small-hint .mt-n2 .mb-1}
   The callback is executed when there's enough space in the write buffer to accept new data to be written.
 - [**Signal**](/signals)

   {:.small-hint .mt-n2 .mb-1}
   The callback is executed when the process received a specific signal from the OS.

We'll give up control to the scheduler until the events we're interested in happened.
调度程序每运行一个循环，则在迭代中执行一些操作：

 - 检查任何延迟回调
 - 检查可操作的计时器/流/信号事件
 - Wait for stream activity until the next timer callback expires (unless there are `defer` events to be executed)

The event loop controls the program flow as long as it runs.
Once we tell the event loop to run it will maintain control until it is suspended, the application errors out, has nothing left to do, or is explicitly stopped.

## 示例

思考一下这个非常简单的例子：

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use Revolt\EventLoop;

$suspension = EventLoop::createSuspension();

$repeatId = EventLoop::repeat(1, function (): void {
    print '++ Executing callback created by EventLoop::repeat()' . PHP_EOL;
});

EventLoop::delay(5, function () use ($suspension, $repeatId): void {
    print '++ Executing callback created by EventLoop::delay()' . PHP_EOL;

    EventLoop::cancel($repeatId);
    $suspension->resume(null);

    print '++ Suspension::resume() is async!' . PHP_EOL;
});

print '++ Suspending to event loop...' . PHP_EOL;

$suspension->suspend();

print '++ Script end' . PHP_EOL;
```

执行上述示例后，应该会看到以下输出：

```plain
++ Suspending to event loop...
++ Executing callback created by EventLoop::repeat()
++ Executing callback created by EventLoop::repeat()
++ Executing callback created by EventLoop::repeat()
++ Executing callback created by EventLoop::repeat()
++ Executing callback created by EventLoop::delay()
++ Suspension::resume() is async!
++ Script end
```

This output demonstrates that what happens inside the event loop is like its own separate program.
Your script will not continue past the point of `$suspension->suspend()` unless the suspension point is resumed with `$suspension->resume()` or `$suspension->throw()`.

While an application can and often does take place almost entirely inside the confines of the event loop, we can also use the event loop to do things like the following example which imposes a short-lived timeout for interactive console input:

```php
<?php

require __DIR__ . '/vendor/autoload.php';

use Revolt\EventLoop;

if (\stream_set_blocking(STDIN, false) !== true) {
    \fwrite(STDERR, "Unable to set STDIN to non-blocking" . PHP_EOL);
    exit(1);
}

print "Write something and hit enter" . PHP_EOL;

$suspension = EventLoop::createSuspension();

$readableId = EventLoop::onReadable(STDIN, function ($id, $stream) use ($suspension): void {
    EventLoop::cancel($id);

    $chunk = \fread($stream, 8192);

    print "Read " . \strlen($chunk) . " bytes" . PHP_EOL;

    $suspension->resume(null);
});

$timeoutId = EventLoop::delay(5, function () use ($readableId, $suspension) {
    EventLoop::cancel($readableId);
    
    print "Timeout reached" . PHP_EOL;

    $suspension->resume(null);
});

$suspension->suspend();

EventLoop::cancel($readableId);
EventLoop::cancel($timeoutId);
```

Obviously we could have simply used `fgets(STDIN)` synchronously in this example.
We're just demonstrating that it's possible to move in and out of the event loop to mix synchronous tasks with non-blocking tasks as needed.

## 事件

Event callbacks are registered using the methods on `Revolt\EventLoop` and are invoked using the following standardized parameter order:

| Method                                | Callback Signature                      |
| ------------------------------------- | --------------------------------------- |
| [`EventLoop::defer()`](/timers)       | `function(string $callbackId)`          |
| [`EventLoop::delay()`](/timers)       | `function(string $callbackId)`          |
| [`EventLoop::repeat()`](/timers)      | `function(string $callbackId)`          |
| [`EventLoop::onReadable()`](/streams) | `function(string $callbackId, $stream)` |
| [`EventLoop::onWritable()`](/streams) | `function(string $callbackId, $stream)` |
| [`EventLoop::onSignal()`](/signals)   | `function(string $callbackId, $signal)` |

### 暂停、恢复和取消回调 

All event callbacks, regardless of type, can be temporarily disabled and enabled in addition to being cancelled via `EventLoop::cancel()`.
This allows for advanced capabilities such as disabling the acceptance of new socket clients in server applications when simultaneity limits are reached.
In general, the performance characteristics of event callback reuse via `enable()`/`disable()` are favorable by comparison to repeatedly canceling and re-registering callbacks.

#### 暂停回调

一个简单的禁用示例：

```php
<?php

use Revolt\EventLoop;

// 注册将禁用的回调
$callbackIdToDisable = EventLoop::delay(1, function (): void {
    echo "I'll never execute in one second because: disable()\n";
});

// 注册回调执行 disable() 操作
EventLoop::delay(0.5, function () use ($callbackIdToDisable) {
    echo "Disabling callback: ", $callbackIdToDisable, "\n";
    EventLoop::disable($callbackIdToDisable);
});

EventLoop::run();
```

After our second event callback executes, the event loop exits because there are no longer any enabled event callbacks registered.

#### 恢复回调

`enable()` is the diametric analog of the `disable()` example demonstrated above:

```php
<?php

use Revolt\EventLoop;

// Register a repeating timer callback
$callbackId = EventLoop::repeat(1, function(): void {
    echo "tick\n";
});

// Disable the callback
EventLoop::disable($callbackId);

EventLoop::defer(function () use ($callbackId): void {
    // Immediately enable the callback when the event loop starts
    EventLoop::enable($callbackId);
    // Now that it's enabled we'll see tick output in our console every second.
});

EventLoop::run();
```

#### 取消回调

It's important to *always* cancel persistent event callbacks once you're finished with them, or you'll create memory leaks in  your application.
This functionality works in exactly the same way as the above `enable` / `disable` examples:

```php
<?php

use Revolt\EventLoop;

$callbackId = EventLoop::repeat(1, function (): void {
    echo "tick\n";
});

// Cancel $callbackId in five seconds and exit the event loop
EventLoop::delay(5, function () use ($callbackId): void {
    EventLoop::cancel($callbackId);
});

EventLoop::run();
```

#### Cancellation Safety

It is always safe to cancel a callback from within itself. For example:

```php
<?php

use Revolt\EventLoop;

$increment = 0;

EventLoop::repeat(0.1, function ($callbackId) use (&$increment): void {
    echo "tick\n";
    if (++$increment >= 3) {
        EventLoop::cancel($callbackId); // <-- cancel myself!
    }
});

EventLoop::run();
```

It is also always safe to cancel a callback from multiple places. A double-cancel will simply be ignored.

### Referencing Callbacks

Callbacks can either be referenced or unreferenced. An unreferenced callback doesn't keep the event loop alive.
All callbacks are referenced by default.

One example to use unreferenced callbacks is when using signal callbacks.
Generally, if all callbacks are gone and only the signal callback still exists, you want to exit the event loop unless you're actively waiting for that event to happen.

#### Referencing Callbacks

`reference()` marks a callback as referenced. Takes the `$callbackId` as first and only argument.

#### Unreferencing Callbacks

`unreference()` marks a callback as unreferenced. Takes the `$callbackId` as first and only argument.
