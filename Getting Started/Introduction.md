[![ReactiveSwift](http://reactivecocoa.io/reactiveswift/docs/latest/Logo/PNG/logo-Swift.png)](https://github.com/ReactiveCocoa/ReactiveSwift/)

泛时数据流~为Swift量身定做

[原英文文档](http://reactivecocoa.io/reactiveswift/docs/latest/index.html) *如果大家发现翻译有错漏的地方请告知~在此感激不敬*

☕️ [查看Cocoa的拓展?](https://github.com/ReactiveCocoa/ReactiveCocoa/#readme) 🎉 [准备开始](http://reactivecocoa.io/reactiveswift/docs/latest/#getting-started) ⚠️ [仍然在用Swift 2.x?](https://github.com/ReactiveCocoa/ReactiveCocoa/tree/v4.0.0) 🚄 [发布历程](http://reactivecocoa.io/reactiveswift/docs/latest/#release-roadmap)

## 什么是 ReactiveSwift ？

ReactiveSwift 提供了组合的、声明的和灵活的基本方法，去构建易于接入的**泛时数据流概念(streams of values over time)**。

这些基本方法可以被用来一致代表常规的Cocoa和一般的程序模式设计，从根本上诸如观察者、代理模式、回调闭包、通知、操作事件、响应链事件、[futures/promises](https://www.gitbook.com/book/yinzhenjob/reactiveswift/#) 和KVO模式。

因为这些所有不同的机制可以被相同的方式表示，所以着很容易将它们进行声明组合，只需要少量的胶水代码将接口粘合。



## 响应单元的核心

#### 信号(Signal)：一个单向的事件流。

信号的拥有者对事件流拥有单面的控制权。观察者随时可以对感兴趣的将要发生的事件进行注册，但是观察不到流或拥有者的**副作用(side effect)**。

这就像看直播电视节目——你可以对内容进行观看和做出评价，但是你不会对直播电视产生任何的副作用。

`````
let channel : Signal<Program, NoError> = tvStation.channelOne

channel.observeValues { program in ... }

`````

#### 事件(Event)：一个事件流的基础传递单位

一个`信号`可以是携带值的任意数量事件，随后由观察者而被观察。(A `Signal` may have any arbitrary number of events carrying a value, **following by an eventual terminal event of a specific reason**.)

这就像实时信号源中的一帧——海洋般的数据帧承载着视频和音频的信息，但这些帧最终会以标识为**结束流**的一个特别帧来到最后的地方而结束。

#### 信号发生器(SignalProducer)： 创建有值数据流的延迟工作

`信号发生器`延迟工作——输出有值的数据流，直到它自己启动。对于每一次调用开启信号发生器的方法，一个新的`信号`会被创建然后延迟工作随后也会被调用。

这就像一个流媒体点播服务——尽管片段只是电视直播的一帧，你仍然可以选择你想看什么、什么时候开始看和中断。

```
let frames: SignalProducer<VideoFrame, ConnectionError> = vidStreamer.streamAsset(id: tvShowId)
let interrupter = frames.start { frame in ... }
interrupter.dispose()
```

#### 属性(Property): 一个始终保持有值的可观察入口

`属性`是一个值变化则立即能观测到的变量。换另外一种说法，这是一个比`信号`更有强力保障的数据流——它能保证最新的数据总是可用，从来没有失败过。

这就像一个当前时间偏移连续更新的视屏录播——这个录播任何时候都会在正确的时间偏移上，只要播放继续它就会被自己的录播逻辑更新。

```
let currentTime: Property<TimeInterval> = video.currentTime
print("Current time offset: \(currentTime.value)")
currentTime.observeValues { timeBar.timeLabel.text = "\($0)" }
```
#### 行为动作(Action): 一系列有序的预置操作

当被输入端调用，`行为动作`使用输入的最新状态进行预置操作，然后输出给所有对该行为动作感兴趣的相关对象。

这就像一个自动售卖的机器——当你选择一个选项插入硬币后，机器就会处理订单，最终输出你想要的那个东西。注意的是整个流程是独立的，你不能让一台机器同时服务两个顾客。

```
// Purchase from the vending machine with a specific option.
vendingMachine.purchase
    .apply(snackId)
    .startWithResult { result
        switch result {
        case let .success(snack):
            print("Snack: \(snack)")

        case let .failure(error):
            // Out of stock? Insufficient fund?
            print("Transaction aborted: \(error)")
        }
    }

// The vending machine.
class VendingMachine {
    let purchase: Action<Int, Snack, VendingMachineError>
    let coins: MutableProperty<Int>

    // The vending machine is connected with a sales recorder.
    init(_ salesRecorder: SalesRecorder) {
        coins = MutableProperty(0)
        purchase = Action(state: coins, enabledIf: { $0 > 0 }) { coins, snackId in
            return SignalProducer { observer, _ in
                // The sales magic happens here.
                // Fetch a snack based on its id
            }
        }

        // The sales recorders are notified for any successful sales.
        purchase.values.observeValues(salesRecorder.record)
    }
}
```
#### 参考

更多有关概念和基本单元的详情在ReactiveSwift，查看这些文档在：

+ 框架概述(Framework Overview)   

  一份概述关于行为、建议使用ReactiveSwift基本元素和工具的示例文档。

+ 基本操作(Basic Operators)

  一份概述关于结合和转换的基本操作。

+ 设计参考(Design Guidelines)

  包含了ReactiveSwift的基本元素，最适合练习ReactiveSwift，和指导自定义操作的实现。

  ​

## 例子：在线搜索

假设有一个输入框，用户可以随时输入东西进去，然后你需要请求网络去搜索输入进去的东西。

*请注意该例子是用[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa/#readme)描述的Cocoa拓展。*

#### 观察文字编辑

第一步是观察输入框的编辑，使用RAC给`UITextFile`进行拓展是我们的主要目的：

`let searchStrings = textField.reactive.continuosTextValues`

这将会给我们发送一个`String?`类型值的信号。

#### 创建网络请求

我们想对每一个字符发起网络请求。ReactiveSwift提供了一个`URLSession`的拓展去做这件事：

```
let searchResults = searchStrings
    .flatMap(.latest) { (query: String?) -> SignalProducer<(Data, URLResponse), AnyError> in
        let request = self.makeSearchRequest(escapedQuery: query)
        return URLSession.shared.reactive.data(with: request)
    }
    .map { (data, response) -> [SearchResult] in
        let string = String(data: data, encoding: .utf8)!
        return self.searchResults(fromJSONString: string)
    }
    .observe(on: UIScheduler())
```
这个把我们数组中的字符转化成包含搜索内容的结果，将传递到主线程中。

此外，`flatMap(.latest)`确保只有最新的搜索被允许执行。用户在网络请求的时候如果键入其它字符它将仍然执行，当前搜索会被取消然后重新开始。仅仅想想我们需要亲手写下多少代码去完成这件事！

#### 接受返回的结果

一旦**搜索字符串**这就变成了一个`信号`并且是一个热信号，这将用于自动计算转化为任何时候从**搜索字符串**发出的新值。

因此，我们能简单地用`Signal.observe(_:)`观察信号:

```
searchResults.observe { event in
    switch event {
    case let .value(results):
        print("Search results: \(results)")

    case let .failed(error):
        print("Search error: \(error)")

    case .completed, .interrupted:
        break
    }
}
```
这里，我们查看包含结果的数据值、事件，然后把这些记录在控制台。这样能更容易地去做些其它事，像更新一个在视图上的列表或者标签。

#### 操作失败

目前在这个例子里，所有的网络错误将会产生一个`失败`事件，这将让一个终止事件流。不幸地，这意味着将来的查询甚至不会去尝试。

为了补救这个，我们需要决定怎么去处理失败发生。最快的解决办法是打印它们，然后忽略它们：

```
    .flatMap(.latest) { (query: String) -> SignalProducer<(Data, URLResponse), AnyError> in
        let request = self.makeSearchRequest(escapedQuery: query)

        return URLSession.shared.reactive
            .data(with: request)
            .flatMapError { error in
                print("Network error occurred: \(error)")
                return SignalProducer.empty
            }
    }
```
为了用空白事件流去代替错误，我们最有效的可能就是忽略它们。

然而，在将要放弃之前进行重试多次可能更加合适。方便的是，有一个`retry`操作符完全地这样去做！

我们改进的`搜索结果`发生器看起来也许是这样：

```
let searchResults = searchStrings
    .flatMap(.latest) { (query: String) -> SignalProducer<(Data, URLResponse), AnyError> in
        let request = self.makeSearchRequest(escapedQuery: query)

        return URLSession.shared.reactive
            .data(with: request)
            .retry(upTo: 2)
            .flatMapError { error in
                print("Network error occurred: \(error)")
                return SignalProducer.empty
            }
    }
    .map { (data, response) -> [SearchResult] in
        let string = String(data: data, encoding: .utf8)!
        return self.searchResults(fromJSONString: string)
    }
    .observe(on: UIScheduler())
```
#### 限制请求

现在，假设你实际上仅仅想要定期地运行搜索，为了减少流量。

ReactiveCocoa有一个声明为`限制(throttle)`的操作符，那样我们可以运用在搜索文字上：

```
let searchStrings = textField.reactive.continuousTextValues
    .throttle(0.5, on: QueueScheduler.main)
```
这防止了被发送的数据值不少与0.5秒。

手动地去做这些事将是奇怪的情况，这最终变得更加难以理解！使用ReactiveCocoa，我们只需使用一个操作符去把时间管理混合到我们的事件流里。

#### 调试事件流

追溯本质，一个事件流的堆栈追踪可能有几十帧，这是经常存在的，这样调试是非常让人沮丧的行为。一个天真的调试想法是，给数据流注入副作用，比如：

```
let searchString = textField.reactive.continuousTextValues
    .throttle(0.5, on: QueueScheduler.main)
    .on(event: { print ($0) }) // the side effect
```

这将会打印数据流的事件，并且保留原始数据流行为。信号发生器和信号这两者都提供了`事件日记(logEvents)`的操作符，这将会替你自动完成这些。

```
let searchString = textField.reactive.continuousTextValues
    .throttle(0.5, on: QueueScheduler.main)
    .logEvents()
```

更多信息和使用方法，关注**调试技巧**文档。



## ReactiveSwift 和 RxSwift的有什么关联？

RxSwift是一个Swift关于 [ReactiveX](https://reactivex.io/) (Rx)接口的实现。ReactiveCocoa很大程度上受到RX的启发，ReactiveSwift是函数响应式编程中有主见的实现，**并且故意没有直接接口如RXSwift 一样 (and *intentionally* not a direct port like [RxSwift](https://github.com/ReactiveX/RxSwift/#readme).)。**

ReactiveSwift和RxSwift/ReactiveX的差别有以下这些：

+ 通过更加简洁的接口返回结果
+ 常见来源地址混淆（Addresses common sources of confusion）
+ 更加接近Swift和Cocoa的一些约定

以下是些重要的差异，它们有自己的原理。

## 信号和信号发生器（*热* 和 *冷* 观察）

对于RX最为让人困惑的一个方面就是『热』、『冷』和『温』观察事件流。

简短一点，用C#方法或函数来描述这件事：

```
IObservable<string> Search(string query)
```

……这是不可能去告诉订阅者是否去观察`IObservable`将要调用副作用。如果它没有调用副作用，这也不可能去告诉每一个订阅者是否有一个副作用，活着只有起初的那个是这样做的。

这个例子是人为的，但是示范了**一个真实、普遍的问题**，即那样会变得极其难以理解RX的代码（和 3.0版本前的ReactiveCocoa代码）一眼看下去来说。

**ReactiveSwift**表示通过分开的`信号`和`信号发生器`类型区别副作用。尽管这意味着你将要多学一个类型，但能够提高代码的清晰和帮助传达意图会更好。

换一句话说，**ReactiveSwift这儿变得更简单**，这是不容易的。

#### 错误类型

在ReactiveSwift中，当信号和信号发生器被允许失败，各种各样的错误必须指定在该错误类型系统里。例如，`Signal<Int, AnyError>`是一个整型值的信号，它如果失败则会发出一个`AnyError`的错误。

更加重要的是，RAC允许`NoError`这个特殊类型用于代替一个不被允许发送失败的静态保障(statically guarantees)事件流。**这个消除了很多由意外失败事件的错误。**

在RX系统类型里，事件流仅仅规定它们值的类型——并没有指名它们的错误类型——所以这分类的保障是不可靠的。

#### 命名

在RX的多个版本里，流在任何时候都是可观察的，这个和*.NET*中的枚举有相似之处。此外，大多数在RX和.NET的操作符都是借用 [LINQ](https://msdn.microsoft.com/en-us/library/bb397926.aspx)的命名，它用于关系型数据库的条件查询(terms reminiscent )，像`Select`和`Where`。

**ReactiveSwift**，在另外一方面，关注Swift原生社区是首要和重要的，并且遵守 [Swift API Guidelines](https://swift.org/documentation/api-design-guidelines/)。其它命名的类型代替很大程度上受[Haskell](https://www.haskell.org) 和[Elm](http://elm-lang.org) （『信号』术语的主要来源）的激发。

#### 界面设计

RX在很多时候不知道怎么去使用。尽管界面设计对RX来说是非常普遍的，这个仅有少数特性学习在特殊的例子里。

ReactiveSwift从[ReactiveUI](http://reactiveui.net/)中受到很多启发，包括基础的行为动作([Actions](http://reactivecocoa.io/reactiveswift/docs/latest/Documentation/FrameworkOverview.md#actions))。

不像ReactiveUI，不幸地是不能直接更改RX使在界面设计上更加友好。**ReactiveSwift明确为了这个目的已经改进很多次了**——甚至意味着与RX进一步分歧。



## 开始准备

ReactiveSwift支持macOS 10.9+, iOS 8.0+, watchOS 2.0+, tvOS 9.0+ and Linux.

#### Carthage

如果你是用 [Carthage](https://github.com/Carthage/Carthage/#readme) 去管理你的依赖包，把ReactiveSwift添加到你的`Cartfile`是方便的：

```
github "ReactiveCocoa/ReactiveSwift" ~> 1.0
```

如果你使用Carthage去构建你的依赖包，请确保已经添加了`ReactiveSwift.framework`和 `Result.framework` 在target里的_Linked Frameworks and Libraries_ 栏目中，以及在Carthage库里复制生成阶段中已经引用它们。

#### Cocoapods

如果你使用[CocoaPods](https://cocoapods.org/)去管理你的依赖包，把ReactiveSwift添加到你的 `Podfile`是方便的：

```
pod 'ReactiveSwift', '~> 1.0.0'
```
#### Swift Package Manager

如果你使用Swift Package Manager，把ReactiveSwift如同依赖包添加到你的`Package.swift`里：

```
.Package(url: "https://github.com/ReactiveCocoa/ReactiveSwift.git", majorVersion: 1)
```

#### Git子库

1. 添加ReactiveSwift仓库像子库一样添加到你的应用仓库。
2. 到包含ReactiveCocoa的文件夹下运行`git submodule update --init --recursive`。
3. 拖拽 `ReactiveSwift.xcodeproj` 和`Carthage/Checkouts/Result/Result.xcodeproj`到你应用Xcode工程里。
4. 在你应用target设置的 “General”表头里，添加 `ReactiveSwift.framework`，和`Result.framework`到“Embedded Binaries”栏目下。
5. 如果你的应用项目一点没有包含Swift代码，你应该在*build setting*设置`EMBEDDED_CONTENT_CONTAINS_SWIFT` 为“Yes”。

## Playground

我们同样提供了Playground，所以你可以使用ReactiveCocoa的操作符。为了开始使用它：

1. 克隆ReactiveSwift仓库。
2. 重新检索你工程的依赖包从ReactiveSwift 工程的根目录，使用以下其中一个终端命令：
   - `git submodule update --init --recursive` 或者，如果你使用[Carthage](https://github.com/Carthage/Carthage/#readme)安装
   - `carthage checkout`
3. 打开`ReactiveSwift.xcworkspace`
4. 编译`Result-Mac` 方案
5. 编译 `ReactiveSwift-macOS` 方案
6. 最后打开`ReactiveSwift.playground`
7. 选择 `View > Show Debug Area`

## 发布版记录

**当前稳定的发布版：**

[![GitHub release](https://img.shields.io/github/release/ReactiveCocoa/ReactiveSwift.svg)](https://github.com/ReactiveCocoa/ReactiveSwift/releases)

### 记录版本

#### ReactiveSwift 2.0

把Swift 3.1.x.作为目标，方案估计得到2017年春。

这个发布版有了完全改变。但是并没有让使用这个的普通群众感到很期待，因为只有增加一些特殊的使用样例。

ReactiveSwift 2.0的主要目标是移除信号实现的协议，例如`SignalProtocol`, `SignalProducerProtocol`，作用于要求相同类型混合的工作组一样。

ReactiveSwift 2.0 可能包含其它推荐的突破改变。

为了兼容准备执行的Swift 4.0，拥有清晰、稳定的API作为开始是很重要的。这很值得期待**在ReactiveSwift 2.0里去把API清理和回顾作为总结**，在我们迁移到ReactiveSwift 3.0和Swift 4.0前。所有帮助实现这个目标的贡献都是欢迎的。

#### ReactiveSwift 3.0

把Swift 4.0.x作为目标，这个方案估计得要2017年末。

这个发布版包含突破性的改变，主要取决于Swift 4.0的特性有什么样的实现。

ReactiveSwift 3.0将会关注以下两个主要目标：

1. Swift 4.0兼容
2. 适应新的特性引用在Swift 4.0的第二阶段