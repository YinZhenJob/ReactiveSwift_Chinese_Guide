# 基本操作符

该文档解释了一些在ReactiveSwift中比较常用的操作符，并包含简单的demon介绍如何使用。

注意操作符，在本文中，涉及到信号和信号发生器转化的函数，并不是自定义Swift的操作符。换另外一句话，ReactiveCocoa为事件流提供这些基本组成。

这篇文档在处理`信号`和`信号发生器`时将使用『事件流』这个概念。当需要做出区别这两者时，它们的命名就是它们的类型。

[TOC]

## 演示事件流的副作用

### 观察者(Observation)

`信号`可以被`观察`函数监听：

```
signal.observe { event in
    switch event {
    case let .value(value):
        print("Value: \(value)")
    case let .failed(error):
        print("Failed: \(error)")
    case .completed:
        print("Completed")
    case .interrupted:
        print("Interrupted")
    }
}
```

并且，当一个相应的事件出现时提供回调给`数据值`、`失败`、`完成`和`中断`事件。

```
signal.observeValues { value in
    print("Value: \(value)")
}

signal.observeFailed { error in
    print("Failed: \(error)")
}

signal.observeCompleted {
    print("Completed")
}

signal.observeInterrupted {
    print("Interrupted")
}
```

### 引入影响(Injecting effects)

副作用能够注入`on`操作给一个事件流，并且不用显式声明它。

```
let producer = signalProducer
    .on(starting: { 
        print("Starting")
    }, started: { 
        print("Started")
    }, event: { event in
        print("Event: \(event)")
    }, value: { value in
        print("Value: \(value)")
    }, failed: { error in
        print("Failed: \(error)")
    }, completed: { 
        print("Completed")
    }, interrupted: { 
        print("Interrupted")
    }, terminated: { 
        print("Terminated")
    }, disposed: { 
        print("Disposed")
    })
```

请注意，这不需要提供所有的参数——所有的这些都是可选的，只需要提供你所关注的回调事件。

再次注意，直到`发生器`开启（可能在其他地方）这并不会打印任何东西。

### 操作符结合(Operator composition)

#### 调用(Lifting)

