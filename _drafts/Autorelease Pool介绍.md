---
title: Autorelease Pool
date: 2018-04-25
categories: iOS
tags: iOS, 翻译
---
# NSAutoreleasepool翻译 

## 综述

NSAutoreleasePool 是一个支持Cocoa引用计数内存管理系统的对象。
当Autorelease pool被drained的时候， pool会向存储在其中的对象发送release消息。

```objc
注意

如果使用ARC，就不能直接使用autorelease pool。可以使用@autoreleasepool代替。例如：

你可以把
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
// Code benefitting from a local autorelease pool.
[pool release];
写成
@autoreleasepool {
    // Code benefitting from a local autorelease pool.
}

@autoreleasepool比直接使用NSAutoreleasePool实例更高效；你也可以在非ARC环境下使用它。
```

在引用计数环境下（与垃圾回收机制对照），存储在NSAutoreleasePool中的对象会收到一个autorelease消息，
当NSAutoreleasePool对象被释放的时候，它会自动发送一个release消息给存储在其中的每一个对象。因此，
向一个对象发送autorelease消息相比于发送release消息会扩展该对象的生命周期直到pool被释放。一个对象
可以被多次放入同一个pool中，在这种情况下，每次把对象放入pool中是该对象都会收到一个release消息。

在引用计数环境下，Cocoa对象总是可以获得一个autorelease pool。如果无法获得pool，Autoreleased
对象就无法release，这样就会导致内存leak。在这种情况下，程序就会打印相应的警告消息。

Application Kit在每一个事件循环开始时，就会在主线程上创建一个Autorelease pool，在事件循环结束时释放pool，从而释放处理事件所产生的autorelease对象。因此，如果你使用Application Kit，那么通常不需要自己创建pool。如果在应用的事件循环中，你创建了许多临时autoreleased对象，那么创建局域autolease pool将有助于节约内存（minimize the peak memory footprint）。

通过向NSAutoreleasePool发送alloc和init消息创建pool对象，发送drain（或release）消息销毁pool。既然不能retain一个autolease pool，那么drain一个pool的最终效果就是释放这个pool。

每个线程拥有它自己的NSAutoreleasePool对象栈。当新的pool被创建时，新的pool就会被添加到栈顶。随着pool被释放，它就从栈中移除。Autoreleased对象被放在当前线程的栈顶pool中。当一个线程结束时，它会自动drain所有与之相关的pool。

## 线程

如果你打算在Application Kit的主线程之外调用Cocoa，例如你打算创建一个只基于Foundation的应用或者开启一个线程，那么就需要创建自己的autorelease pool。

如果你的应用或者线程的生命周期很长，并且有可能产生许多autoreleased对象，那么你就应该周期性的释放和创建autolease pool（就像Application Kit在主线程上做的那样）；否则，autoreleased对象积累起来会导致内存增长。如果你创建的线程不调用Cocoa，就不需要创建autorelease pool。

# Using Autorelease Pool Blocks翻译

## 简介

Autorelease pool block(APB) 提供一种机制，这种机制能让你即便不拥有一个对象，也不必担心会立马释放掉这个对象。通常不需要自己创建APB，但是有些情况下需要你创建APB。

## 关于APB

APB用`@autoreleasepool`标记，比如下面这个例子
``` objc
@autoreleasepool {
    // Code that creates autoreleased objects.
}
```
在APB的结尾，block内收到一个`autorelease`消息的对象会被发送一个`release`消息。

Cocoa总是希望在APB中执行，否则autoreleased对象就不会被释放，从而导致内存leak。AppKit和UIKit框架会在APB中处理每一个事件循环。因此，通常不用自己创建APB。不过有三种情况你需要创建自己的APB：
* 如果你显得程序不是基于UI框架的，比如命令行工具
* 如果你写的循环创建了许多临时对象
* 如果你创建了另外线程

## 使用龃龉APB减少内存峰值

许多程序会创建临时的autoreleased对象。直到block结束之前，这些对象都会占用内存。在大多数情况下，直到当前事件循环结束前临时对象的累积不会导致内存过度开销；然而有一些情况，你可能会创建大量的临时对象，从而大幅度占用内存，你会想快速释放这些临时对象。这时你就要创建自己的APB了。在block结束时，临时对象会被释放，从而减少内存占用。

下面这个例子展示了在for循环中使用局域的APB。
```objc
NSArray *urls = <# An array of file URLs #>;
for (NSURL *url in urls) {
    @autoreleasepool {
        NSError *error;
        NSString *fileContents = [NSString stringWithContentsOfURL:url encoding:NSUTF8StringEncoding error:&error];
        /* Process the string, creating and autoreleasing more objects. */
    }
}
```
for循环一次处理一个文件。接收到autorelease消息的对象会在block结束时被释放。


# 参考文章

1. [NSAutoreleasepool](https://developer.apple.com/documentation/foundation/nsautoreleasepool)
2. [Using Autorelease Pool Blocks](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html#//apple_ref/doc/uid/20000047-1041876)

