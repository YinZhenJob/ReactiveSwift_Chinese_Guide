# ReactiveSwift 中文介绍

因为想学习RSwift但发现中文相关介绍比较少，鉴于方便国内同好所以打算将相关资料翻译。目前正在做准备工作，希望有共同兴趣的朋友一起来加入这个行列。

​														—— 2017.03.08

小伙伴时间被导师挤没了，所以……这真是个浩大的工程~我已经泪奔了~~~，离职了，还得找工作,这段时间不像前两年，市场饱和了~每天还是挤出点时间来弄弄了，谁知道有什么意外效果。

如果在这个项目上感兴趣的小伙伴，请联系我：

Email： zohar_zeng@163.com

​													  	——2017-03-16 

## 目录

##### [让我们开始吧](https://github.com/YinZhenJob/ReactiveSwift_Chinese_Guide/blob/master/Getting%20Started/Introduction.md) Getting Started

+ [基本运算符（未完）](https://github.com/YinZhenJob/ReactiveSwift_Chinese_Guide/blob/master/Getting%20Started/Base%20Operators.md)	(Basic Operators)
+ 框架概述         (Framework Overview)
+ 调试技巧         (Debugging Techniques)
+ 设计指南         (DesignGuidenlines)

##### 信号  Signal

+ 事件		(Event)
+ 事件协议         (EventProtocol)
+ 观察者             (Observer)
+ 观察者协议     (ObserverProtocol)
+ 信号                (Signal)
+ 信号发生器     (SignalProtocol)
+ 扁平策略         (FlattenStrategy)

##### 信号发生器  SignalProducer

+ 信号发生器 	(SignalProducer)
+ 信号发生器协议 (SignalProducerProtocol)
+ 时间  timer(interval: on:)
+ 时间  timer(interval: on: leeway:)

##### 属性 Property

+ 属性		(Property)
+ 属性协议        (PropertyProtocol)
+ 可变属性        (MutableProperty)
+ 可变属性协议 (MutablePropertyProtocol)

##### 行为操作  Action

+ 行为操作 	(Action)
+ 行为操作错误 (ActionError)
+ 行为操作协议 (ActionProtocol)

##### 绑定  Bindings

+ 源绑定协议	(BindingSourceProtocol)
+ 目标绑定	        (BindingTarget)
+ 目标绑定协议 (BindingTargetProtocol)

##### 任务计划  Schedulers

+ 日期任务协议	(DateSchedulerProtocol)
+ 即刻任务         (ImmediateScheduler)
+ 任务池             (QueueScheduler)
+ 任务协议         (schedulerProtocol)
+ 测试任务         (TestScheduler)
+ UI任务             (UIScheduler)

##### 响应拓展  Reactive Extensions

+ 响应		    (Reactive)
+ 响应拓展提供者 (ReactiveExtensionsProvider)

##### 生命周期  LifeTime

+ 生命周期  LiftTime
+ 令牌         (- Token)

##### 弃用  Disposable  

+ Any弃用		(AnyDisposable)
+ 行为弃用      （ActionDisposable）
+ 弃用                (Disposable)
+ 结合弃用      （CompositeDisposable）
+ 弃用操作      （DisposableHandle）
+ 范围弃用      （ScopedDisposable）
+ 连续弃用      （SerialDisposable）
+ 简单弃用        (SimpleDisposable)
+ +=(\_:\_:)

##### 实用工具 Utilities

+ 原子		(Atomic)
+ 原子协议        (AtomicProtocol)
+ 包                 （Bag）
+ 包连接器        (BagIterator)
+ 迁移令牌        (RemovalToken)
+ 事件记录器    (EventLogger)
+ 记录事件     （LoggingEvent）
+ 可选               (Optional)
+ 可选协议       (OptionalProtocol)