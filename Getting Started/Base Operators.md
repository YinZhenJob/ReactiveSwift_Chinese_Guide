# 基本操作符

该文档解释了一些在ReactiveSwift中比较常用的操作符，并包含简单的demon介绍如何使用。

注意操作符，在本文中，涉及到信号和信号发生器转化的函数，并不是自定义Swift的操作符。换另外一句话，ReactiveCocoa为事件流提供这些基本组成。

这篇文档在处理`信号Signal`和`信号发生器SignalProducer`时将使用『事件流』这个概念。当需要做出区别这两者时，它们的命名就是它们的类型。

[TOC]

## 演示事件流的副作用

### 观察者(Observation)

`信号Signal`可以被`观察observe`函数监听：

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

并且，当一个相应的事件出现时提供回调给`数据值value`、`失败failed`、`完成completed`和`中断interrupted`事件。

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

再次注意，直到`发生器producer`开启（可能在其他地方）这并不会打印任何东西。

## 操作符结合(Operator composition)

#### 调用(Lifting)

`信号(Signal)`操作符能够调用`信号发生器(SignalProducer)`调用的一切方法。

这将会创建一个新的`信号发生器(SignalProducer)`,它将应用操作符给每一个`信号(Signal)`的创建，就如操作符已经应用于每一个独立`信号(Signal)`的生成。

## 转化事件流(Transforming event streams)

这些操作符转化一个事件流到一个新的流。

#### 映射(Mapping)

`映射map`操作符被用来转换值到一个事件流，创建一个新的流来作为它的结果。

```
let (signal, observer) = Signal<String, NoError>.pipe()

signal
    .map { string in string.uppercased() }
    .observeValues { value in print(value) }

observer.send(value: "a")     // Prints A
observer.send(value: "b")     // Prints B
observer.send(value: "c")     // Prints C
```

[`map`操作符的可视化交互](http://neilpa.me/rac-marbles/#map) 

#### 滤除(Filtering)

`滤除Filtering`操作符用于仅包括满足谓词事件流中的值。

```
let (signal, observer) = Signal<Int, NoError>.pipe()

signal
    .filter { number in number % 2 == 0 }
    .observeValues { value in print(value) }

observer.send(value: 1)     // Not printed
observer.send(value: 2)     // Prints 2
observer.send(value: 3)     // Not printed
observer.send(value: 4)     // prints 4
```

[`filter`操作符的可视化交互](http://neilpa.me/rac-marbles/#filter)

### 聚合(Aggregating)

`简化reduce`操作符被用来聚合事件流为单个组合值。注意，最终值仅在输入流完成后才发送。

```
let (signal, observer) = Signal<Int, NoError>.pipe()

signal
    .reduce(1) { $0 * $1 }
    .observeValues { value in print(value) }

observer.send(value: 1)     // nothing printed
observer.send(value: 2)     // nothing printed
observer.send(value: 3)     // nothing printed
observer.sendCompleted()   // prints 6
```

`集合Collect`操作符被用来聚合事件流为单个数组值。注意，最终值仅在输入流完成后才发送。

```
let (signal, observer) = Signal<Int, NoError>.pipe()

signal
    .collect()
    .observeValues { value in print(value) }

observer.send(value: 1)     // nothing printed
observer.send(value: 2)     // nothing printed
observer.send(value: 3)     // nothing printed
observer.sendCompleted()   // prints [1, 2, 3]
```

[`reduce`操作符的可视化交互.](http://neilpa.me/rac-marbles/#reduce)

## 组合事件流(Combining event streams)

这些操作符组合多个事件流中的数据从而变成一个新的统一的流。

#### 组合最新数据(Combining latest values)

`最新组合combineLatest`函数组合最新的两个或多个事件流。

返回流将仅在最初的数据到之后的每一个输入的数据都已发送完成时才发送。至此，从任何输入的新数据将会返回一个新的值到输出端。

```
let (numbersSignal, numbersObserver) = Signal<Int, NoError>.pipe()
let (lettersSignal, lettersObserver) = Signal<String, NoError>.pipe()

let signal = Signal.combineLatest(numbersSignal, lettersSignal)
signal.observeValues { next in print("Next: \(next)") }
signal.observeCompleted { print("Completed") }

numbersObserver.send(value: 0)      // nothing printed
numbersObserver.send(value: 1)      // nothing printed
lettersObserver.send(value: "A")    // prints (1, A)
numbersObserver.send(value: 2)      // prints (2, A)
numbersObserver.sendCompleted()  // nothing printed
lettersObserver.send(value: "B")    // prints (2, B)
lettersObserver.send(value: "C")    // prints (2, C)
lettersObserver.sendCompleted()  // prints "Completed"
```

`combineLatest(with:)` 操作符工作在同样的方法上。

[`combineLatest`操作符的可视化交互](http://neilpa.me/rac-marbles/#combineLatest)

### 压缩(Zipping)

`压缩zip`函数成对连接两个或多个数据流中的值。任何第N个元组的元素对应于输入流的第N个元素。

这就意味着第N个输出流的数值不能被发送，直到每一个输入端发送至少N个数据之后。

```
let (numbersSignal, numbersObserver) = Signal<Int, NoError>.pipe()
let (lettersSignal, lettersObserver) = Signal<String, NoError>.pipe()

let signal = Signal.zip(numbersSignal, lettersSignal)
signal.observeValues { next in print("Next: \(next)") }
signal.observeCompleted { print("Completed") }

numbersObserver.send(value: 0)      // nothing printed
numbersObserver.send(value: 1)      // nothing printed
lettersObserver.send(value: "A")    // prints (0, A)
numbersObserver.send(value: 2)      // nothing printed
numbersObserver.sendCompleted()  // nothing printed
lettersObserver.send(value: "B")    // prints (1, B)
lettersObserver.send(value: "C")    // prints (2, C) & "Completed"
```

 `zipWith` 操作符以同样的方式工作。

[`.zip`操作符的可视化交互](http://neilpa.me/rac-marbles/#zip)

## 扁平化事件流(Flattening event streams)

`扁平化flatten`操作符转变流中的流（a stream-of-streams）成为一个单独的流，根据提供的`扁平化策略FlattenStrategy`从内流转发数据。扁平化的结果会变成输出流的类型，就像一个信号发生器的信号发生器，或者信号的信号发生器变得扁平化成为一个信号发生器；同样的一个信号发生器的信号，或者信号的信号变得扁平化成为一个信号。

为了明白为什么会有不同的策略和它们之间的区别，看一下以下的示例并将列偏移量设为时间：

```
let values = [
// imagine column offset as time
[1,2,3],
[4,5,6],
[7,8],
]

let merge =
[1,4,2,7,5,3,8,6]

let concat = 
[1,2,3,4,5,6,7,8]

let latest =
[1,4,7,8]
```

请注意，数值如何交错 和 哪些数值是被包含结果数组中的。

### 合并(Merging)

`.merge` 属性方法可以立即将输入事件流每个值转到内部事件流。所有外部事件流和内部事件流的失败提示被立即发送到扁平化事件流中并终结。

```
let (lettersSignal, lettersObserver) = Signal<String, NoError>.pipe()
let (numbersSignal, numbersObserver) = Signal<String, NoError>.pipe()
let (signal, observer) = Signal<Signal<String, NoError>, NoError>.pipe()

signal.flatten(.merge).observeValues { print($0) }

observer.send(value: lettersSignal)
observer.send(value: numbersSignal)
observer.sendCompleted()

lettersObserver.send(value: "a")    // prints "a"
numbersObserver.send(value: "1")    // prints "1"
lettersObserver.send(value: "b")    // prints "b"
numbersObserver.send(value: "2")    // prints "2"
lettersObserver.send(value: "c")    // prints "c"
numbersObserver.send(value: "3")    // prints "3"
```

[ `flatten(.merge)` 操作符的可视化交互.](http://neilpa.me/rac-marbles/#merge)

### 连接(Concatenating)

`.concat`属性方法常被用来序列化内部事件流中连续事件。外部事件流可从开始被观测。直到前一个进度条已经完成，随后的每一个事件流才能观测。失败提示会立刻被转发到扁平事件流中。

```
let (lettersSignal, lettersObserver) = Signal<String, NoError>.pipe()
let (numbersSignal, numbersObserver) = Signal<String, NoError>.pipe()
let (signal, observer) = Signal<Signal<String, NoError>, NoError>.pipe()

signal.flatten(.concat).observeValues { print($0) }

observer.send(value: lettersSignal)
observer.send(value: numbersSignal)
observer.sendCompleted()

numbersObserver.send(value: "1")    // nothing printed
lettersObserver.send(value: "a")    // prints "a"
lettersObserver.send(value: "b")    // prints "b"
numbersObserver.send(value: "2")    // nothing printed
lettersObserver.send(value: "c")    // prints "c"
lettersObserver.sendCompleted()     // nothing printed
numbersObserver.send(value: "3")    // prints "3"
numbersObserver.sendCompleted()     // nothing printed
```

[ `flatten(.concat)`操作符的可视化交互](http://neilpa.me/rac-marbles/#concat)

### 选择最新(Switching to the latest)

`.latest`属性方法今后选择只有最新输入端事件流中新值或者失败提示。

```
let (lettersSignal, lettersObserver) = Signal<String, NoError>.pipe()
let (numbersSignal, numbersObserver) = Signal<String, NoError>.pipe()
let (signal, observer) = Signal<Signal<String, NoError>, NoError>.pipe()

signal.flatten(.latest).observeValues { print($0) }

observer.send(value: lettersSignal) // nothing printed
numbersObserver.send(value: "1")    // nothing printed
lettersObserver.send(value: "a")    // prints "a"
lettersObserver.send(value: "b")    // prints "b"
numbersObserver.send(value: "2")    // nothing printed
observer.send(value: numbersSignal) // nothing printed
lettersObserver.send(value: "c")    // nothing printed
numbersObserver.send(value: "3")    // prints "3"
```



## 与错误相伴(Working with errors)

这些操作符常被用来操作那些可能发生在事件流中的失败，或者在事件流中可能失败的执行操作。

### 获取失败(Catching failures)

`flatMapError`操作可获取所有发生在输入端事件流中的失败提示，然后开启一个新的`SignalProducer` 在同一个地方。

```
let (signal, observer) = Signal<String, NSError>.pipe()
let producer = SignalProducer(signal: signal)

let error = NSError(domain: "domain", code: 0, userInfo: nil)

producer
    .flatMapError { _ in SignalProducer<String, NoError>(value: "Default") }
    .startWithValues { print($0) }


observer.send(value: "First")     // prints "First"
observer.send(value: "Second")    // prints "Second"
observer.send(error: error)       // prints "Default"
```

### 失败转换(Failable transformations)

`SignalProducer.attempt(_:)` 允许你把一个可失败操作转变成一个事件流。`attempt(_:)` 和 `attemptMap(_:)` 操作符允许你执行可失败操作或者转换成一个事件流。

```
let dictionaryPath = URL(fileURLWithPath: "/usr/share/dict/words")

// Create a `SignalProducer` that lazily attempts the closure
// whenever it is started
let data = SignalProducer.attempt { try Data(contentsOf: dictionaryPath) }

// Lazily apply a failable transformation
let json = data.attemptMap { try JSONSerialization.jsonObject(with: $0) }

json.startWithResult { result in
    switch result {
    case let .success(words):
        print("Dictionary as JSON:")
        print(words)
    case let .failure(error):
        print("Couldn't parse dictionary as JSON: \(error)")
    }
}
```

### 重试(Retrying)

 `retry` 操作符将会数次重启最初 `SignalProducer`，直到失败次数达到。

```
var tries = 0
let limit = 2
let error = NSError(domain: "domain", code: 0, userInfo: nil)
let producer = SignalProducer<String, NSError> { (observer, _) in
    tries += 1
    if tries <= limit {
        observer.send(error: error)
    } else {
        observer.send(value: "Success")
        observer.sendCompleted()
    }
}

producer
    .on(failed: {e in print("Failure")})    // prints "Failure" twice
    .retry(upTo: 2)
    .start { event in
        switch event {
        case let .value(next):
            print(next)                     // prints "Success"
        case let .failed(error):
            print("Failed: \(error)")
        case .completed:
            print("Completed")
        case .interrupted:
            print("Interrupted")
        }
}
```

如果`SignalProducer` 在达到尝试的次数后没有成功，`SignalProducer` 的结果将会失败。例如：在上例使用`retry(1)`而不是 `retry(2)`，将会打印 `"Failed: Error Domain=domain Code=0 "(null)""` 而不是 `"Success"`。

### 映射错误(Mapping errors)

 `mapError` 操作符转换所有在一个事件流中的失败错误到一个新的失败消息。

```
enum CustomError: String, Error {
    case foo = "Foo Error"
    case bar = "Bar Error"
    case other = "Other Error"
}

let (signal, observer) = Signal<String, NSError>.pipe()

signal
    .mapError { (error: NSError) -> CustomError in
        switch error.domain {
        case "com.example.foo":
            return .foo
        case "com.example.bar":
            return .bar
        default:
            return .other
        }
    }
    .observeFailed { error in
        print(error.rawValue)
}

observer.send(error: NSError(domain: "com.example.foo", code: 42, userInfo: nil))    // prints
```

### 提升(Promote)

 `promoteErrors` 操作符促使一个通常不可失败事件流为可失败事件流。

```
let (numbersSignal, numbersObserver) = Signal<Int, NoError>.pipe()
let (lettersSignal, lettersObserver) = Signal<String, NSError>.pipe()

numbersSignal
    .promoteErrors(NSError.self)
    .combineLatest(with: lettersSignal)
```

这提供的流依然是不可失败的，但这变得非常有用。因为一些操作符组合流要求输入已经有匹配的失败类型。