# 为 Retrofit2 提供的 FastJson 转换库

> 为 Retrofit2 提供的 FastJson 转换器（Retrofit2-FastJson-Converter）

## 前言

Retrofit 是 Android 和 Java 平台上一款优秀且使用广泛的 Http 客户端，GitHub 上 21K+ 的 Star 和 4.3K+ 的 Fork 充分证明了 Retrofit 的风靡程度。

Retrofit 在 Android 平台如此流行与它及其简介的调用方式和优秀的可扩展、可配置性是分不开的。我们客户端在和服务端交互的时候通常采用 Json 格式来传递数据，客户端拿到服务端传递过来的 Json 格式的数据后需要对它进行解析；Json 解析库有很多：Gson、Jackson、FastJson等等。Retrofit 优秀的可配置性可以让我们客户端程序员随意选择心怡的 Json 解析库，Retrofit 针对 Gson 和 Jackson 都提供相应的 Converter；可能由于 FastJson 是国内程序员开发的原因，Retrofit 对于 FastJson 并没有提供对应的 Converter ，这对于使用 FastJson 的开发者是不友好的。

## 使用

好在 Retrofit 提供了接口来让开发者实现自己的 Json Converter 。实现 Converter 虽然简单，但每次使用 Retrofit2 + FastJson 组合时都实现一套显然是没必要的，于是我使用 FastJson 实现了一个 Converter: [Retrofit2-FastJson-Converter](https://github.com/BaronZ88/Retrofit2-FastJson-Converter) 。有同样需求的同学只需要使用我这个 Converter 库就好啦，不行再去自定义。使用方式如下：

### 1、添加依赖配置

**Step 1**. 由于 [Retrofit2-FastJson-Converter](https://github.com/BaronZ88/Retrofit2-FastJson-Converter)  是发布到 JitPack 的，因此首先需要在项目根目录的 build.gradle 中加入 JitPack 的仓库地址，具体配置如下：

```groovy
allprojects {
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}
```
	
**Step 2**. 在具体使用 [Retrofit2-FastJson-Converter](https://github.com/BaronZ88/Retrofit2-FastJson-Converter) 的 module 中加入依赖配置：

```groovy
dependencies {
	compile 'com.github.BaronZ88:Retrofit2-FastJson-Converter:lastVersion'
}
```

### 2、配置 Retrofit Converter

在 Retrofit.Builder 的 addConverterFactory 方法中传入 `FastJsonConverterFactory.create()` ：

```java
Retrofit retrofit = new Retrofit.Builder()
      .baseUrl(baseUrl)
      .addConverterFactory(FastJsonConverterFactory.create())
      .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
      .client(client)
      .build();
```

最后贴上 Retrofit2-FastJson-Converter 源码地址 ：[https://github.com/BaronZ88/Retrofit2-FastJson-Converter](https://github.com/BaronZ88/Retrofit2-FastJson-Converter)

## 其他

**一直不满意各博客平台上的阅读体验，排版糟糕、布局混乱、字体丑陋、各种广告及杂七杂八的组件分散了读者宝贵的注意力；最最重要的是这年头竟然找不到一个优雅、简介、有美感的博客平台！！！我不能忍！真的不能忍！！为了赏脸阅读我文章的读者！为了我这仅剩的一点点审美！我必须采用 GitHub Pages + Hexo + NexT 来搭建一个优美简介的个人博客 [http://baronzhang.com](http://baronzhang.com) ，不过身为拖延症晚期患者的我，直到最近才将博客系统的各项功能陆续完善起来。之前的文章均已同步，之后所有的文章也会第一时间在我的个人博客上发布，追求更好阅读体验的同学都来访问 [baronzhang.com](http://baronzhang.com) 吧。**

> 如果你喜欢我的文章，就关注下我的**知乎专栏**或者在 GitHub 上添个 Star 吧！
>   
> * 知乎专栏：[https://zhuanlan.zhihu.com/baron](https://zhuanlan.zhihu.com/baron)  
> * GitHub：[https://github.com/BaronZ88](https://github.com/BaronZ88)
> * 个人博客：[http://baronzhang.com](http://baronzhang.com)



