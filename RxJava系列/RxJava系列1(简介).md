##RxJava系列一（简介）

> 转载请注明出处：[http://www.jianshu.com/p/ec9849f2e510](http://www.jianshu.com/p/ec9849f2e510)


[RxJava系列1(简介)](http://www.jianshu.com/p/ec9849f2e510)  
[RxJava系列2(入门)](http://www.jianshu.com/p/ba61c047c230)  
<u>RxJava系列3(敬请期待)</u>  
<u>RxJava系列4(敬请期待)</u>  

***
###前言
提升开发效率，降低维护成本一直是开发团队永恒不变的宗旨。近一年来国内的技术圈子中越来越多的开始提及Rx，经过一段时间的学习和探索之后我也深深的感受到了RxJava的魅力。它能帮助我们简化代码逻辑，提升代码可读性。这对于开发效率的提升、后期维护成本的降低帮助都是巨大的。个人预测RxJava一定是2016年的一个大趋势，所以也有打算将它引入到公司现有的项目中来，写这一系列的文章主要也是为了团队内部做技术分享。

> 由于我本人是个Android程序猿，因此这一系列文章中的场景都是基于Android平台的。如果你是个Java Web工程师或者是其它方向的那也没关系，我会尽量用通俗的语言将问题描述清楚。

###响应式编程
在介绍RxJava前，我们先聊聊响应式编程。那么什么是响应式编程呢？响应式编程是一种基于异步数据流概念的编程模式。数据流就像一条河：它可以被观测，被过滤，被操作，或者为新的消费者与另外一条流合并为一条新的流。

响应式编程的一个关键概念是事件。事件可以被等待，可以触发过程，也可以触发其它事件。事件是唯一的以合适的方式将我们的现实世界映射到我们的软件中：如果屋里太热了我们就打开一扇窗户。同样的，当我们的天气app从服务端获取到新的天气数据后，我们需要更新app上展示天气信息的UI；汽车上的车道偏移系统探测到车辆偏移了正常路线就会提醒驾驶者纠正，就是是响应事件。

今天，响应式编程最通用的一个场景是UI：我们的移动App必须做出对网络调用、用户触摸输入和系统弹框的响应。在这个世界上，软件之所以是事件驱动并响应的是因为现实生活也是如此。

> 本章节中部分概念摘自《RxJava Essentials》一书

###RxJava的来历
Rx是微软.Net的一个响应式扩展，Rx借助可观测的序列提供一种简单的方式来创建异步的，基于事件驱动的程序。2012年Netflix为了应对不断增长的业务需求开始将.NET Rx迁移到JVM上面。并于13年二月份正式向外展示了RxJava。
从语义的角度来看，RxJava就是.NET Rx。从语法的角度来看，Netflix考虑到了对应每个Rx方法,保留了Java代码规范和基本的模式。

###什么是RxJava
叨逼叨的说了一大堆，你特么倒是告诉RxJava是个什么货啊！！！

那么到底什么是RxJava呢？我对它的定义是：**RxJava本质上是一个异步操作库，是一个能让你用极其简洁的逻辑去处理繁琐复杂任务的异步事件库。**（注意，我说的是简洁的逻辑而不是简洁的代码！）

###RxJava好在哪
Android平台上为已经开发者提供了AsyncTask,Handler等用来做异步操作的类库，那我们为什么还要选择RxJava呢？答案是简洁！RxJava可以用非常简洁的代码逻辑来解决复杂问题；而且即使业务逻辑的越来越复杂，它依然能够保持简洁！再配合上Lambda用简单的几行代码分分钟就解决你负责的业务问题。简直逼格爆表，拿它装逼那是极好的！

多说无益，上代码！

假设有这样一个需求：我们要读取Android手机SD卡中目录数组 `File[] folders`中每个目录下的所有png图片,展示在我们的自定义控件imageViewGroup上;imageViewGroup有个`addImage(Bitmap bitmap)`方法用来向控件上添加图片。由于从SD卡上读取文件比较耗时，故需要在后台运行；图片显示又需要在UI线程上执行，通常我们可以这样实现：

	new Thread() {
    	@Override
        public void run() {
        	super.run();
            for (File folder : folders) {
            	File[] files = folder.listFiles();
            	for (File file : files) {
                	if(file.getName().endsWith(".png")){
                		Bitmap bitmap = getBitmapFromFile(file);
                   		runOnUiThread(new Runnable() {
                   			@Override
                   			public void run() {
                       			imageViewGroup.addImage(bitmap);
                      		}
                 		});
              		}
          		}
        	}
    	}
    }.start();

使用RxJava的写法是这样的：

    Observable.from(folders)
            .flatMap(new Func1<File, Observable<File>>() {
                @Override
                public Observable<File> call(File file) {
                    return Observable.from(file.listFiles());
                }
            })
            .filter(new Func1<File, Boolean>() {
                @Override
                public Boolean call(File file) {
                    return file.getName().endsWith(".png");
                }
            })
            .map(new Func1<File, Bitmap>() {
                @Override
                public Bitmap call(File file) {
                    return getBitmapFromFile(file);
                }
            })
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new Action1<Bitmap>() {
                @Override
                public void call(Bitmap bitmap) {
                    imageViewGroup.addImage(bitmap);
                }
            });
            
从上面这段代码我们可以看到：虽然代码量看起来变复杂了，但是RxJava的实现是一条链式调用，没有任何的嵌套；整个实现逻辑看起来异常简洁清晰，这对我们的编程实现和后期维护是有巨大帮助的。特别是对于那些回调嵌套的场景。配合Lambda表达式还可以简化成这样：
  
      Observable.from(folders)
            .flatMap(file -> Observable.from(file.listFiles()))
            .filter(file -> file.getName().endsWith(".png"))
            .map(this::getBitmapFromFile)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(bitmap -> imageViewGroup.addImage(bitmap));

看到这样的代码我想用用两个词来形容它们：优雅！完美！！！

看完这篇文章大家应该能够理解RxJava为什么会越来越火了。它能极大的提高我们的开发效率和代码的可读性！当然了RxJava的学习曲线也是比较陡的，在后面的文章我会对主要的知识点做详细的介绍，敬请关注！