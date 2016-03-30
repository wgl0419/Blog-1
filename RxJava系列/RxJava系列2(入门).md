##RxJava系列二（入门）
> 转载请注明出处：[http://www.jianshu.com/p/ba61c047c230](http://www.jianshu.com/p/ba61c047c230)


[RxJava系列1(简介)](http://www.jianshu.com/p/ec9849f2e510)  
[RxJava系列2(入门)](http://www.jianshu.com/p/ba61c047c230)  
<u>RxJava系列3(敬请期待)</u>  
<u>RxJava系列4(敬请期待)</u>  


***
###前言
上一篇的示例代码中大家一定发现了Observable这个类。从纯Java的观点看，Observable类源自于经典的观察者模式。RxJava的异步实现正是基于观察者模式来实现的，而且是一种扩展的观察者模式。

###观察者模式
观察者模式基于Subject这个概念，Subject是一种特殊对象，又叫做**主题**或者**被观察者**。当它改变时那些由它保存的一系列对象将会得到通知，而这一系列对象被称作Observer(**观察者**)。它们会对外暴漏了一个通知方法(比方说update之类的)，当Subject状态发生变化时会调用的这个方法。

观察者模式很适合下面这些场景中的任何一个：

1. 当你的架构有两个实体类，一个依赖另一个，你想让它们互不影响或者是独立复用它们时。
2. 当一个变化的对象通知那些与它自身变化相关联的未知数量的对象时。
3. 当一个变化的对象通知那些无需推断具体类型的对象时。

通常一个观察者模式的类图是这样的：

![Observer](../DesignPatterns/ObserverPattern/Observer.png)

如果你对观察者模式很了解，那么强烈建议你先去学习下。关于观察者模式的详细介绍可以参考我之前的文章：[设计模式之观察者模式](http://www.jianshu.com/p/d55ee6e83d66)

###扩展的观察者模式以及RxJava如何使用

在RxJava中主要有4个角色：

* Observable
* Subject
* Observer
* Subscriber

Observable和Subject是两个“生产”实体，Observer和Subscriber是两个“消费”实体。说直白点`Observable`对应于观察者模式中的**被观察者**，而`Observer`和`Subscriber`对应于观察者模式中的**观察者**。`Subscriber`其实是一个实现了`Observer`的抽象类，后面我们分析源码的时候也会介绍到。`Subject`比较复杂，以后再分析。

上一篇文章中我们说到RxJava中有个关键概念：**事件**。观察者`Observer`和被观察者`Observable`通过`subscribe()`方法实现订阅关系。从而`Observable` 可以在需要的时候发出**事件**来通知`Observer`。

我自己在学习一种新技术的时候通常喜欢先去了解它是怎么用的，掌握了使用方法后再去深挖其原理。那么我们现在就来说说RxJava到底该怎么用。

**第一步：创建观察者Observer**

	Observer<Object> observer = new Observer<Object>() {
	
        @Override
        public void onCompleted() {

        }

        @Override
        public void onError(Throwable e) {

        }

        @Override
        public void onNext(Object s) {

        }
     };
     
这么简单，一个观察者Observer创建了!
        
大兄弟你等等...，你之前那篇[观察者模式](http://www.jianshu.com/p/d55ee6e83d66)中不是说观察者只提供一个update方法的吗？这特么怎么有三个？！！

少年勿急，且听我慢慢道来。在普通的观察者模式中观察者一般只会提供一个update()方法用于被观察者的状态发生变化时，用于提供给被观察者调用。而在RxJava中的观察者Observer提供了:`onNext()`、 `onCompleted()`和`onError()`三个方法。还记得吗？开篇我们讲过RxJava是基于一种扩展的观察这模式实现，这里多出的onCompleted和onError正是对观察者模式的扩展。*ps:onNext就相当于普通观察者模式中的update*

RxJava中添加了普通观察者模式缺失的三个功能：

1. RxJava中规定当不再有新的事件发出时，可以调用onCompleted()方法作为标示；
2. 当事件处理出现异常时框架自动触发onError()方法；
3. 同时Observables支持链式调用，从而避免了回调嵌套的问题。
 

**第二步：创建被观察者Observable**

`Observable.create()`方法可以创建一个Observable，使用`crate()`创建Observable需要一个OnSubscribe对象，这个对象继承Action1。当观察者订阅我们的Observable时，它作为一个参数传入并执行`call()`函数。 
	
	Observable<Object> observable = Observable.create(new 				Observable.OnSubscribe<Object>() {
        @Override
        public void call(Subscriber<? super Object> subscriber) {

        }
    });

除了create()，just()和from()同样可以创建Observable。看看下面两个例子：

`just(T...)`将传入的参数依次发送

	Observable observable = Observable.just("One", "Two", "Three");
	//上面这行代码会依次调用
	//onNext("One");
	//onNext("Two");
	//onNext("Three");
	//onCompleted();

`from(T[])/from(Iterable<? extends T>)`将传入的数组或者Iterable拆分成Java对象依次发送

	String[] parameters = {"One", "Two", "Three"};
	Observable observable = Observable.from(parameters);
	//上面这行代码会依次调用
	//onNext("One");
	//onNext("Two");
	//onNext("Three");
	//onCompleted();

**第三步：被观察者Observable订阅观察者Observable**（*ps:你没看错，不同于普通的观察者模式，这里是被观察者订阅观察者*）
	
有了观察者和被观察者，Wimbledon就可以调用subscribe()订阅事件了，就像这样：
	
	observable.subscribe(observer);
	
连在一起写就是这样：

    Observable.create(new Observable.OnSubscribe<Integer>() {

        @Override
        public void call(Subscriber<? super Integer> subscriber) {
            for (int i = 0; i < 5; i++) {
                subscriber.onNext(i);
            }
            subscriber.onCompleted();
        }

    }).subscribe(new Observer<Integer>() {

        @Override
        public void onCompleted() {
            System.out.println("onCompleted");
        }

        @Override
        public void onError(Throwable e) {
            System.out.println("onError");
        }

        @Override
        public void onNext(Integer item) {
            System.out.println("Item is " + item);
        }
    });
至此一个完整的RxJava调用就完成了。

兄台，你叨逼叨叨逼叨的说了一大堆，可是我没搞定你特么到底在干啥啊？！！不急，我现在就来告诉你们到底发生了什么。

首先我们使用Observable.create()创建了一个新的Observable<Integer>，并为`create()`方法传入了一个OnSubscribe，OnSubscribe中包含一个`call()`方法，一旦我们调用`subscribe()`订阅后就会自动触发call()方法。call()方法中的参数Subscriber其实就是subscribe()方法中的观察者Observer。我们在`call()`方法中调用了5次`onNext()`和1次`onCompleted()`方法。一套流程周下来以后输出结果就是下面这样的：

	Item is 0
	Item is 1
	Item is 2
	Item is 3
	Item is 4
	onCompleted
	
看到这里可能你又要说了，大兄弟你别唬我啊！OnSubscribe的call()方法中的参数Subscriber怎么就变成了subscribe()方法中的观察者Observer？！！！这俩儿货明明看起来就是两个不同的类啊。

我们先看看Subscriber这个类：

	public abstract class Subscriber<T> implements Observer<T>, Subscription {
		
		...
	}
从源码中我们可以看到，Subscriber是Observer的一个抽象实现类，所以我首先可以肯定的是Subscriber和Observer类型是一致的。接着往下我们看看subscribe()这个方法：

    public final Subscription subscribe(final Observer<? super T> observer) {
    
    	//这里的if判断对于我们要分享的问题没有关联，可以先无视
        if (observer instanceof Subscriber) {
            return subscribe((Subscriber<? super T>)observer);
        }
        return subscribe(new Subscriber<T>() {

            @Override
            public void onCompleted() {
                observer.onCompleted();
            }

            @Override
            public void onError(Throwable e) {
                observer.onError(e);
            }

            @Override
            public void onNext(T t) {
                observer.onNext(t);
            }

        });
    }
我们看到subscribe()方法内部首先将传进来的Observer做了一层代理，将它转换成了Subscriber。我们再看看这个方法内部的subscribe()方法：

    public final Subscription subscribe(Subscriber<? super T> subscriber) {
        return Observable.subscribe(subscriber, this);
    }
进一步往下追踪看看return后面这段代码到底做了什么。精简掉其他无关代码后的subscribe(subscriber, this)方法是这样的：

    private static <T> Subscription subscribe(Subscriber<? super T> subscriber, Observable<T> observable) {

        subscriber.onStart();
        try {
            hook.onSubscribeStart(observable, observable.onSubscribe).call(subscriber);
            return hook.onSubscribeReturn(subscriber);
        } catch (Throwable e) {
            return Subscriptions.unsubscribed();
        }
    }

我们重点看看hook.onSubscribeStart(observable, observable.onSubscribe).call(subscriber),前面这个hook.onSubscribeStart(observable, observable.onSubscribe)返回的是它自己括号内的第二个参数observable.onSubscribe,然后调用了它的call方法。而这个observable.onSubscribe正是create()方法中的Subscriber，这样整个流程就理顺了。看到这里是不是对RxJava的执行流程清晰了一点呢？这里也建议大家在学习新技术的时候多去翻一翻源码，知其然还要能知其所以然不是吗。

###异步
上一篇文章中开篇就讲到RxJava就是来处理异步任务的。但是默认情况下我们在哪个线程调用subscribe()就在哪个线程生产事件，在哪个线程生产事件就在哪个线程消费事件。那怎么做到异步呢？RxJava为我们提供Scheduler用来做线程调度，我们来看看RxJava提供了哪些Scheduler。


| Schedulers                    | 作用           |
| ----------------------------- | ------------- |
| Schedulers.immediate()        | 默认的Scheduler，直接在当前线程运行。|
| Schedulers.newThread()        | 总是启用一个新线程来运行。|
| Schedulers.io()               | 用来做io操作的Scheduler，例如文件、DB和网络操作等。它的行为模式和newThread()差不多，区别在于io()的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。| 
| Schedulers.computation()      | 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。 | 
| AndroidSchedulers.mainThread()| RxAndroid中新增的Scheduler，表示在Android主线程中运行。 |

同时RxJava还为我们提供了`subscribeOn()`和`observeOn()`两个方法来指定Observable和Observer运行的线程。

    Observable.from(folders)
            .flatMap(file -> Observable.from(file.listFiles()))
            .filter(file -> file.getName().endsWith(".png"))
            .map(this::getBitmapFromFile)
            .subscribeOn(Schedulers.io())//指定获取图片的任务在IO线程运行
            .observeOn(AndroidSchedulers.mainThread())//指定显示图片的任务在UI线程运行
            .subscribe(bitmap -> imageViewGroup.addImage(bitmap));

上面这段代码大家应该有印象吧，没错正是我们上一篇文章中的例子。`subscribeOn(Schedulers.io())`指定了获取png图片的一系列事件都是在IO线程中运行，`observeOn(AndroidSchedulers.mainThread())`指定了显示图片的操作在UI线程执行。这就做到了在子线程获取图片，主线程显示图片。

好了，RxJava系列的入门内容我们就聊到这。下一篇我们再继续介绍更多的API以及它们内部的原理。

