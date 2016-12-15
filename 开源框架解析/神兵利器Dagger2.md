# 神兵利器Dagger2

> 这可能是东半球最好的Dagger2入门教程了

## 一、简介

Dagger-匕首，鼎鼎大名的Square公司旗下又一把利刃（没错！还有一把黄油刀，唤作ButterKnife）；故此给本篇取名神兵利器Dagger2。

Dagger2起源于Dagger，是一款基于Java注解来实现的完全在编译阶段完成依赖注入的开源库，主要用于模块间解耦、提高代码的健壮性和可维护性。

起初Square公司受到Guice的启发而开发了Dagger，但是Dagger这种半静态半运行时的框架还是有些性能问题（很多时候依赖注入都是通过反射来实现，这无论在大型的服务端应用还是在Android应用上都不是最优方案）。因此Google工程师Fork了Dagger项目，对它进行了改造。于是变演变出了今天我们要讨论的Dagger2，所以说Dagger2其实就是高配版的Dagger。

## 二、依赖注入（Dependency Injection）

那么什么是依赖注入呢？在解释这个概念前我们先看一小段代码：

```java
public class Car{

	private Engine engine;
	
	public Car(){
		engine = new Engine();
	}
}
```

这段Java代码中Car类持有了对Engine实例的引用，我们称之为Car类对Engine类有一个依赖。而依赖注入则是指通过注入的方式实现类与类之间的依赖，下面是常见的三种依赖注入的方式：

1、构造注入：

```java
public class Car{

	private Engine engine;
	
	public Car(Engine engine){
		this.engine = engine;
	}
}
```

2、接口注入：

```java
public interface Injection<T>{

	void inject(T t);
}

public class Car implements Injection<Engine>{

	private Engine engine;
	
	public Car(){}

	public void inject(Engine engine){
		this.engine = engine;
	}

}
```

3、注解注入：

```java
public class Car{

	@Inject
	Engine engine;
	
	public Car(){}
}
```

前两种注入方式需要我们编写大量的模板代码，而机智的Dagger2则是通过Java注解在编译期来实现依赖注入的。

## 三、为什么需要依赖注入

我们之所是要依赖注入，最重要的就是为了解耦，达到高内聚低耦合的目的，保证代码的健壮性、灵活性和可维护性。

下面我们看看同一个业务的两种实现方案：

方案一

```java
public class Car{

	private Engine engine;
	private List<Wheel> wheels;

	public Car(){
		engine = new Engine();
		wheels = new ArrayList<>();
		for(int i = 0; i < 4; i++){
			wheels.add(new Wheel());
		}
	}

	public void start{
		System.out.println("启动汽车");
	}
}

public class CarTest{

	public static void main(String[] args){
		Car car = new Car();
		car.start();
	}
} 
```

方案二

```java
public class Car{

	private Engine engine;
	private List<Wheel> wheels;

	public Car(Engine engine, List<Wheel> wheels){
		this.engine = engine;
		this.wheels = wheels;
	}

	public void start{
		System.out.println("启动汽车");
	}
}

public class CarTest{

	public static void main(String[] args){

		Engine engine = new Engine();
		List<Wheel> wheels = new ArrayList<>();
		for(int i = 0; i < 4; i++){
			wheels.add(new Wheel());
		}
		Car car = new Car(engine, wheels);
		car.start();
	}
}
```

方案一：由于没有依赖注入，因此需要我们自己是在Car的构造函数中创建Engine和Wheel对象。

方案二：我们手动以构造函数的方式注入依赖，将engine和wheels作为参数传入而不是在Car的构造函数中去显示的创建。

方案一明显丧失了灵活性，一切依赖都是在Car类的内部创建，Car与Engine和Wheel严重耦合。一旦Engine或者Wheel的创建方式发生了改变，我们就必须要去修改Car类的构造函数（比如说现在创建Wheel实例的构造函数改变了，需要传入Rubber（橡胶）了）；另外我们也没办法替换动态的替换依赖实例（比如我们想把Car的Wheel（轮胎）从邓禄普（轮胎品牌）换成米其林（轮胎品牌）的）。这类问题在大型的商业项目中则更加严重，往往A依赖B、B依赖C、C依赖D、D依赖E；一旦稍有改动便牵一发而动全身，想想都可怕！而依赖注入则帮我们解决了这一问题。

## 四、为什么是Dagger2

无论是构造函数注入还是接口注入，都避免不了要编写大量的模板代码。机智的猿猿们当然不开心做这些重复性的工作，于是各种依赖注入框架应用而生。但是这么多的依赖注入框架为什么我们却偏爱Dagger2呢？我们先从Spring中的控制反转（IOC）说起。

谈起依赖注入，做过J2EE开发的同学一定会想起Spring IOC，那通过迷之XML来配置依赖的方式让我厌恶至今；而且XML与Java代码分离也导致代码链难以追踪。之后更加先进的Guice（Android端也有个RoboGuice）出现了，我们不再需要通过XML来配置依赖，但其运行时实现注入的方式让我们在追踪和定位错误的时候却又万分痛苦。开篇提到过Dagger就是受Guice的启发而开发出来的；Dagger继承了前辈的思想，在性能又碾压了它的前辈Guice，可谓是长江后浪推前浪，前浪死在沙滩上。

又如开篇我在简介中说到的，Dagger是一种半静态半运行时的DI框架。很多时候它依赖注入需要通过反射来实现，这无论在大型的服务端应用还是在Android应用上都不是最优方案。升级版的Dagger2解决了这一问题，从半静态变为完全静态，从Map式的API变成申明式API（@Module），生成的代码更优雅高效；而且一旦出错我们在编译期间就能发现。所以Dagger2对开发者的更加友好了，当然Dagger2也因此丧失了一些灵活性，但总体来说利还是远远大于弊的。

前面提到这种A B C D E连续依赖的问题，一旦E的创建方式发生了改变就会引发连锁反应，可能会导致A B C D都需要做针对性的修改；但是骚年，你以为为这仅仅是工作量的问题吗？更可怕的是我们创建A时需要按顺序先创建E D C B四个对象，而且必须保证顺序上是正确的。Dagger2就很好的解决了这一问题（不只是Dagger2，在其他DI框架中开发者同样不需要关注这些问题）。

## 五、Dagger2注解

开篇我们就提到Dagger2是基于Java注解来实现依赖注入的，那么在正式使用之前我们需要先了解下Dagger2中的注解：

* @Inject：@Inject有两个作用，一是用来标记需要依赖的变量，以此告诉Dagger2为它提供依赖；二是用来标记构造函数，Dagger2通过Inject注解可以在需要这个类实例的时候来找到这个构造函数并把相关实例构造出来，以此来为被Inject标记了的变量提供依赖；

* @Module：@Module用于标注专门用来提供依赖的类。有的人可能有些疑惑，看了上面的@Inject，需要在构造函数上标记才能提供依赖，那么如果我们需要提供的类构造函数无法修改怎么办，比如一些jar包里的类，我们无法修改源码。这时候就需要使用Module了。Module可以给不能修改源码的类提供依赖，当然，能用Inject标注的通过Module也可以提供依赖；

* @Provide：@Provide用于标注Module所标注的类中的一个方法，该方法在需要提供依赖时被调用，从而把预先提供好的对象当做依赖给标注了@Injection的变量赋值；

* @Component：@Component用于标注接口，是依赖需求方和依赖提供方之间的桥梁。被Component标注的接口在编译时会生成该接口的实现类（如果@Component标注的接口为CarComponent，则编译期生成的实现类为DaggerCarComponent）,我们通过调用这个实现类的方法完成注入； 

* @Qulifier：@Qulifier用于自定义注解，也就是说@Qulifier就如同Java提供的几种基本元注解一样用来标记注解类。我们在使用@Module来标注提供依赖的方法时，方法名我们是可以随便定义的（虽然我们定义方法名一般以provide开头，但这并不是强制的，只是为了增加可读性而已）。那么Dagger2怎么知道这个方法是为谁提供依赖呢？答案就是返回值的类型，Dagger2根据返回值的类型来决定为哪个被@Inject标记了的变量赋值。但是问题来了，一旦有多个一样的返回类型Dagger2就懵逼了。@Qulifier的存在正式为了解决这个问题，我们使用@Qulifier来定义自己的注解，然后通过自定义的注解去标注提供依赖的方法和依赖需求方（也就是被@Inject标注的变量）；这样Dagger2就知道为谁提供依赖了。

* @Scope：

## 六、Dagger2的简单使用

关于Dagger2的依赖配置就不在这里占用篇幅去描述了，大家可以到它的github主页下去查看官方教程[https://github.com/google/dagger](https://github.com/google/dagger)

前面我们详(luo)细(suo)的介绍了什么是依赖注入以及怎样实现依赖注入；下面来看看如何使用Dagger2来实现依赖注入。我们还是拿前面的Car和Engine来举例。

```Java
public class Car {

    @Inject
    Engine engine;

    public Car() {
        DaggerCarComponent.builder().build().inject(this);
    }

    public Engine getEngine() {
        return this.engine;
    }
}
```

```Java
public class Engine {
    
    @Inject
    Engine(){}
    
    public void run(){
        System.out.println("引擎转起来了~~~");
    }
}
```

```Java
@Component
public interface CarComponent {
    void inject(Car car);
}
```

如果创建Engine的构造函数需要参数呢？比如说制造一台引擎是需要齿轮(Gear)的。

```Java
public class Car {

    @Inject
    Engine engine;

    public Car() {
        DaggerCarComponent.builder()
                .markCarModule(new MarkCarModule())
                .build().inject(this);
    }

    public Engine getEngine() {
        return this.engine;
    }
}
```

```Java
public class Engine {
    
    public Engine(String gear){ }
    
    public void run(){
        System.out.println("引擎转起来了~~~");
    }
}
```

```Java
@Module
public class MarkCarModule {

    public MarkCarModule(){ }

    @Provides
    Engine provideEngine(){
        return new Engine("gear");
    }
}
```

```Java
@Component(modules = {MarkCarModule.class})
public interface CarComponent {
    void inject(Car car);
}
```

## 七、Dagger2原理分析

## 八、总结

关于Dagger2在实际项目中的应用可以参照这个开源项目 [https://github.com/BaronZ88/MinimalistWeather](https://github.com/BaronZ88/MinimalistWeather)



