---
title: Autorelease Pool
date: 2018-04-25
categories: iOS
tags: iOS
---
#> 综述

NSAutoreleasePool 是一个支持Cocoa引用计数内存管理系统的对象。
当Autorelease pool被drained的时候， pool会向存储在其中的对象发送release消息。

```c++
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

Application Kit在每一个事件循环开始时，就会在主线程上创建一个Autorelease pool，


