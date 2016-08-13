##RxJava系列五（组合操作符）
> 转载请注明出处：[]()

[RxJava系列1(简介)](http://www.jianshu.com/p/ec9849f2e510)  
[RxJava系列2(基本概念及使用介绍)](http://www.jianshu.com/p/ba61c047c230)  
[RxJava系列3(转换操作符)](http://www.jianshu.com/p/5970280703b9)  
[RxJava系列4(过滤操作符)](http://www.jianshu.com/p/3a188b995daa)  
[RxJava系列5(组合操作符)]()     
<u>RxJava系列6(源码分析)</u>    
<u>RxJava系列7(最佳实践)</u> 

***
这一章我们接着介绍组合操作符，这类operators可以同时处理多个Observable来创建我们所需要的Observable。组合操作符主要包含： **`Merge`** **`Zip`** **`Join`** **`CombineLatest`** **`And/Then/When`** **`SwitchOnNext`** **`StartWith`**等等。

###Merge
**`merge(Observable, Observable)`**将亮个Observable发射的事件序列组合并成一个事件序列，就像是一个Observable发射的一样。你可以简单的将它理解为两个Obsrvable合并成了一个Observable。
![merge(Observable, Observable)](MergeOperator.png)

**`merge(Observable[])`**将多个Observable发射的事件序列组合并成一个事件序列，就像是一个Observable发射的一样。
![merge(Observable[])](MergeIOOperator.png)

###Zip
**`zip(Observable, Observable, Func2)`**用来合并两个Observable发射的数据项，根据Func2函数生成一个新的值并发射出去。
![zip(Observable, Observable, Func2)](ZipOperator.png)

###Join
**`join(Observable, Func1, Func1, Func2)`**
我们先介绍下join操作符的4个参数：

* Observable:源Observable需要组合的Observable
* Func1:
* Func1:
* Func2:

![join(Observable, Func1, Func1, Func2)](JoinOperator.png)



###CombineLatest
**`comnineLatest(Observable, Observable, Func2)`**用于将两个Observale最近发射的数据已经Func2函数的规则进展组合。下面是官方提供的原理图：
![comnineLatest(Observable, Observable, Func2)](CombineLatestOperator.png)

下面这张图应该更容易理解：
![comnineLatest(Observable, Observable, Func2)](combineLatest.png)

###And/Then/When

![]()

###SwitchOnNext
**`switchOnNext(Observable<? extends Observable<? extends T>>`**用来将一个发射多个小Observable的源Observable转化为一个Observable，然后发射这多个小Observable所发射的数据。如果一个小的Observable正在发射数据的时候，源Observable又发射出一个新的小Observable，则前一个Observable发射的数据会被抛弃，直接发射新
的小Observable所发射的数据。

结合下面的原理图大家应该很容易理解，我们可以看到下图中的黄色圆圈就被丢弃了。
![switchOnNext(Observable<? extends Observable<? extends T>>)](SwitchOnNextOperator.png)

###StartWith
**`startWith(T)`**用于在源Observable发射的数据前插入数据。使用**`startWith(Iterable<T>)`**我们还可以在源Observable发射的数据前插入Iterable。官方示意图：
![startWith(T) startWith(Iterable<T>)](StartWithOperator.png)

**`startWith(Observable<T>)`**用于在源Observable发射的数据前插入另一个Observable发射的数据（这些数据会被插入到
源Observable发射数据的前面）。官方示意图：
![](StartWithOOperator.png)





