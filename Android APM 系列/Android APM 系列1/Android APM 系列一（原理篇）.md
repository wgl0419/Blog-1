# Android 性能监控系列一（原理篇）

![题图来自 https://unsplash.com](http://ocjtywvav.bkt.clouddn.com/blog/framework/android/apm/apm1/header.jpg)

> 欢迎关注微信公众号：**BaronTalk**，获取更多精彩好文！

## 一. 前言

性能问题是导致 App 用户流失的罪魁祸首之一，如果用户在使用我们 App 的时候遇到诸如页面卡顿、响应速度慢、发热严重、流量电量消耗大等问题的时候，很可能就会卸载掉我们的 App。而往往获取用户的成本是高昂的，因此因为性能问题导致用户流失的情况是我们要极力避免的，做不好这一点是我们开发人员的失职。

去年我们团队完成了整个项目架构方面的重构（有兴趣的同学可以参考我之前的文章[安居客 Android 项目架构演进 ](https://mp.weixin.qq.com/s?__biz=MzU4ODM2MjczNA==&mid=2247483731&idx=1&sn=76bd5612ba723171b6ebac69aaf039f8&chksm=fddca7d2caab2ec4eec8736cf4005615c401984e2218a0cfc71dddfe3a204495c4e8a7312b4a&scene=38#wechat_redirect)与[Android 模块化探索与实践 ](https://mp.weixin.qq.com/s?__biz=MzU4ODM2MjczNA==&mid=2247483732&idx=1&sn=b7ee1151b2c8ad2e997b8db39adf3267&chksm=fddca7d5caab2ec33905cc3350f31c0c98794774b0d04a01845565e3989b1f20205c7f432cb9&scene=38#wechat_redirect) ），目前已经能够很好的支撑我们的业务，并对团队的开发效率也有了一定的提升、项目质量也有了大幅的进步。

但是项目上线后，到底有没有性能问题？用户体验到底怎么样？在用户的使用场景中到底会遇到哪些性能问题？我们项目的性能短板又在哪里？这些问题的答案我们都不得而知，因此开发一套完善的性能监控体系势在必行。我们团队在今年开始着手开发自己的性能监控组件 APM，希望通过它来采集线上性能数据，找到性能短板，针对性的优化用户体验。

> APM 全称 Application Performance Management & Monitoring (应用性能管理/监控)  

后面我会通过一系列的文章来介绍 APM 的原理、框架设计与实现等等。本篇就是这个系列的第一篇，主要从实现原理方面来介绍 APM。按照目前的计划，这个系列大致会从如下几个方面来展开：

* **原理篇**：主要介绍 APM 的实现原理；
* **设计篇**：介绍整个 APM 框架设计；
* **实现篇-Gradle Plugin**：介绍 Gradle 插件在 APM 项目中的应用，以及如何开发一个 Gradle Plugin；
* **实现篇-Javassist/ASM**：Javassist、ASM 等字节码操作库的介绍，以及如何使用它们在编译时插入代码来采集各项性能数据；
* **实现篇-数据存储及上报**：介绍 APM 框架的存储上报机制及实现过程；
* **发布集成**：最后会介绍如何将库发布到 jCenter() 以及如何在生产项目中集成。

这里要向大家交代一点是，之前的文章为了极力做到将复杂的问题用通俗易懂的方式解释清楚，又要面面俱到，往往篇幅过长；诸如之前写过的[RxJava系列6(从微观角度解读RxJava源码) ](https://mp.weixin.qq.com/s?__biz=MzU4ODM2MjczNA==&mid=2247483727&idx=3&sn=f3d0ccfee85c26d8e49cfd22fbccb7f6&chksm=fddca7cecaab2ed8be6e34288bcec23a34289f09fd22e5a06ecc547020a3474978b73366d94b&scene=38#wechat_redirect)、[神兵利器Dagger2 ](https://mp.weixin.qq.com/s?__biz=MzU4ODM2MjczNA==&mid=2247483730&idx=1&sn=d308f7073182f8c48dd41e17f2fb7682&chksm=fddca7d3caab2ec5c77884a3f8414d2668cd40627c9b3674dcbd3774deb31400ff34d5e6d11a&scene=38#wechat_redirect) 、[安居客 Android 项目架构演进 ](https://mp.weixin.qq.com/s?__biz=MzU4ODM2MjczNA==&mid=2247483731&idx=1&sn=76bd5612ba723171b6ebac69aaf039f8&chksm=fddca7d2caab2ec4eec8736cf4005615c401984e2218a0cfc71dddfe3a204495c4e8a7312b4a&scene=38#wechat_redirect)、[Android 模块化探索与实践 ](https://mp.weixin.qq.com/s?__biz=MzU4ODM2MjczNA==&mid=2247483732&idx=1&sn=b7ee1151b2c8ad2e997b8db39adf3267&chksm=fddca7d5caab2ec33905cc3350f31c0c98794774b0d04a01845565e3989b1f20205c7f432cb9&scene=38#wechat_redirect) 、[写给 Android 应用工程师的 Binder 原理剖析](https://mp.weixin.qq.com/s?__biz=MzU4ODM2MjczNA==&mid=2247483735&idx=1&sn=202bcc94cc581d91e77728afe674fdfe&chksm=fddca7d6caab2ec0c1a08e25dc1acf5aaf257365f64d5c50c37aae4ead919fdbc11062575f9f&scene=38#wechat_redirect)等文章，篇幅通常都在 8000~10000字以上，通篇阅读下来可能需要近半个小时的时间，不太符合当下碎片化阅读的需求；因此在后面的写作上会控制篇幅，尽量控制在 10 分钟以内的长度。

这也是我为什么会将 APM 作为一个系列来介绍的原因，同时这也能保证后面在介绍 APM 的时候能够深入到实现细节，避免泛泛而谈。

## 二. Android APM 的基本原理

市场上有很多商业化的 APM 平台，比如著名的 NewRelic，还有国内的 听云、OneAPM 等等。这些平台的工作流程基本都是一致的：

1. 首先在客户端（Android、iOS、Web等）采集数据；
2. 接着将采集到的数据整理上报到服务器；
3. 服务器接收到数据后建模、存储、挖掘分析，让后将数据可视化，供用户使用。

如下图：
![APM 工作流程](http://ocjtywvav.bkt.clouddn.com/blog/framework/android/apm/apm.png)

我们介绍的 Android APM 框架其实就是在 Android 平台上应用的一个数据采集上报 SDK。主要包含三大模块：

1. 数据采集
2. 数据存储
3. 数据上报

其中数据采集是整个 APM 框架的核心。

数据采集我们可以通过手动埋点的方式，但这种方式工作量巨大、不灵活，而且无法覆盖到所有场景；因此只能通过自动化的方式来采集数据。在应用构建期间，通过修改字节码的方式来进行字节码插桩就是实现自动化的方案之一。

## 三. Android 打包流程及字节码插桩原理

在谈字节码插桩的原理之前，首先我们看看 Android 的打包流程，如下图：
![Android 打包流程](http://ocjtywvav.bkt.clouddn.com/blog/framework/android/apm/apk-build.png)

从上面这张打包流程图我们可以看到，一个 App 的所有 class 文件，包括第三方的 class 文件都会经过 dex 的过程打包成一个或者多个 dex 文件。

这其中涉及到两个很关键的环节：
1. **javac**：将 .java 格式的源代码文件编译成 class 文件；
2. **dex**: 将 class 格式的文件打包汇总，组成一个或者多个 dex 文件。

我们想要对字节码进行修改，只需要在 javac 之后 dex 之前遍历所有的字节码文件，并按照一定的规则过滤修改就好了，这里便是字节码插桩的入口。

那么我们到底如何介入打包过程，在 class 转换为 dex 文件的时候实现对字节码的修改呢？

答案是 **transform api**

Android Gradle Plugin 1.5.0 及以上版本，Google 官方提供了 transform api 作为字节码插桩的入口。我们只需要实现一个自定义的 Gradle Plugin，然后在编译阶段去修改字节码文件。对于 Gradle Plugin 的具体实现后面的文章再做详细讲解。

## 四. 修改字节码

找到了插桩入口，接下来就要对字节码进行修改。对于字节码的修改，比较常用的框架有 Javassist 和 ASM。

1. **Javassist** 是一个开源的分析、编辑和创建 Java 字节码的类库,它提供了源码级别的 API 以及字节码级别的 API，源码级别的 API，直接使用 Java 编码的形式，而不需要深入了解虚拟机指令，就能动态改变类的结构或者动态生成类。

2. **ASM** 是一个 Java 字节码操控框架。它能被用来动态生成类或者增强既有类的功能。ASM 可以直接产生二进制 class 文件，也可以在类被加载入 Java 虚拟机之前动态改变类行为。

ASM 和 Javassit 相比，API 贴近底层，比较难使用，需要对 Java 字节码和虚拟机方面有一定程度的了解。ASM 的优点就在于性能上的优势，且更加灵活；Javassist 的实现中大量使用的反射，所以性能偏低。

简单的说就是 ASM 虽然难以使用，但是功能强大效率高。是很多无痕埋点、APM框架的首选方案。

ASM 的具体时候我们放到这个系列后面的文章介绍。

## 五. 总结

Android APM 的原理其实非常简单，用一句话总结就是：

依据打包原理，在 class 转换为 dex 的过程中，调用 gradle transform api 遍历 class 文件，借助 Javassist、ASM 等框架修改字节码，插入我们自己的代码实现性能数据的统计。

以上所有过程都是在编译期完成的。

其实 Android 上的无痕埋点也是同样的原理，区别只不过是我们 hook 的点不同，采集的数据不同，因此掌握了 APM 的实现原理同样可以实现无痕埋点系统。

原理很简单，难的是实现细节。比如如何插桩采集到页面帧率、流量、耗电量等等。这些具体细节我们放到后面一一介绍。至于为什么放到后面……因为很多东西自己没做过我也不知道啊……🤣

> 如果你喜欢我的文章，就关注下我的公众号 **BaronTalk** 、 [**知乎专栏**](https://zhuanlan.zhihu.com/baron) 或者在 [**GitHub**](https://github.com/BaronZ88) 上添个 Star 吧！
>   
> * 微信公众号：**BaronTalk**
> * 知乎专栏：[https://zhuanlan.zhihu.com/baron](https://zhuanlan.zhihu.com/baron)  
> * GitHub：[https://github.com/BaronZ88](https://github.com/BaronZ88)
> * 个人博客：[http://baronzhang.com](http://baronzhang.com)

![](http://ocjtywvav.bkt.clouddn.com/blog/common/qrcode1.png)

