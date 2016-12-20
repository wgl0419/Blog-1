# 神兵利器Dagger2

Dagger-匕首，鼎鼎大名的Square公司旗下又一把利刃（没错！还有一把黄油刀，唤作ButterKnife）；故此给本篇取名神兵利器Dagger2。

Dagger2起源于Dagger，是一款基于Java注解来实现的完全在编译阶段完成依赖注入的开源库，主要用于模块间解耦、提高代码的健壮性和可维护性。Dagger2在编译阶段通过apt利用Java注解自动生成Java代码，然后结合手写的代码来自动帮我们完成依赖注入的工作。

起初Square公司受到Guice的启发而开发了Dagger，但是Dagger这种半静态半运行时的框架还是有些性能问题（虽说依赖注入是完全静态的，但是其有向无环图(Directed Acyclic Graph)还是基于反射来生成的，这无论在大型的服务端应用还是在Android应用上都不是最优方案）。因此Google工程师Fork了Dagger项目，对它进行了改造。于是变演变出了今天我们要讨论的Dagger2，所以说Dagger2其实就是高配版的Dagger。

## 依赖注入（Dependency Injection）

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

#### 1、构造注入：通过给构造函数传参给依赖的成员变量赋值，从而实现注入。

```java
public class Car{

	private Engine engine;
	
	public Car(Engine engine){
		this.engine = engine;
	}
}
```

#### 2、接口注入：实现接口方法，同样以传参的方式实现注入。

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

#### 3、注解注入：使用Java注解在编译阶段生成代码实现注入或者是在运行阶段通过反射实现注入。

```java
public class Car{

	@Inject
	Engine engine;
	
	public Car(){}
}
```

前两种注入方式需要我们编写大量的模板代码，而机智的Dagger2则是通过Java注解在编译期来实现依赖注入的。

## 为什么需要依赖注入

我们之所是要依赖注入，最重要的就是为了解耦，达到高内聚低耦合的目的，保证代码的健壮性、灵活性和可维护性。

下面我们看看同一个业务的两种实现方案：

#### 1、方案A

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

#### 2、方案B

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

方案A：由于没有依赖注入，因此需要我们自己是在Car的构造函数中创建Engine和Wheel对象。

方案B：我们手动以构造函数的方式注入依赖，将engine和wheels作为参数传入而不是在Car的构造函数中去显示的创建。

方案A明显丧失了灵活性，一切依赖都是在Car类的内部创建，Car与Engine和Wheel严重耦合。一旦Engine或者Wheel的创建方式发生了改变，我们就必须要去修改Car类的构造函数（比如说现在创建Wheel实例的构造函数改变了，需要传入Rubber（橡胶）了）；另外我们也没办法替换动态的替换依赖实例（比如我们想把Car的Wheel（轮胎）从邓禄普（轮胎品牌）换成米其林（轮胎品牌）的）。这类问题在大型的商业项目中则更加严重，往往A依赖B、B依赖C、C依赖D、D依赖E；一旦稍有改动便牵一发而动全身，想想都可怕！而依赖注入则很好的帮我们解决了这一问题。

## 为什么是Dagger2

无论是构造函数注入还是接口注入，都避免不了要编写大量的模板代码。机智的猿猿们当然不开心做这些重复性的工作，于是各种依赖注入框架应用而生。但是这么多的依赖注入框架为什么我们却偏爱Dagger2呢？我们先从Spring中的控制反转（IOC）说起。

谈起依赖注入，做过J2EE开发的同学一定会想起Spring IOC，那通过迷之XML来配置依赖的方式真的很让人讨厌；而且XML与Java代码分离也导致代码链难以追踪。之后更加先进的Guice（Android端也有个RoboGuice）出现了，我们不再需要通过XML来配置依赖，但其运行时实现注入的方式让我们在追踪和定位错误的时候却又万分痛苦。开篇提到过Dagger就是受Guice的启发而开发出来的；Dagger继承了前辈的思想，在性能又碾压了它的前辈Guice，可谓是长江后浪推前浪，前浪死在沙滩上。

又如开篇我在简介中说到的，Dagger是一种半静态半运行时的DI框架，虽说依赖注入是完全静态的，但是生成有向无环图(DAG)还是基于反射来实现，这无论在大型的服务端应用还是在Android应用上都不是最优方案。升级版的Dagger2解决了这一问题，从半静态变为完全静态，从Map式的API变成申明式API（@Module），生成的代码更优雅高效；而且一旦出错我们在编译期间就能发现。所以Dagger2对开发者的更加友好了，当然Dagger2也因此丧失了一些灵活性，但总体来说利还是远远大于弊的。

前面提到这种A B C D E连续依赖的问题，一旦E的创建方式发生了改变就会引发连锁反应，可能会导致A B C D都需要做针对性的修改；但是骚年，你以为为这仅仅是工作量的问题吗？更可怕的是我们创建A时需要按顺序先创建E D C B四个对象，而且必须保证顺序上是正确的。Dagger2就很好的解决了这一问题（不只是Dagger2，在其他DI框架中开发者同样不需要关注这些问题）。

## Dagger2注解

开篇我们就提到Dagger2是基于Java注解来实现依赖注入的，那么在正式使用之前我们需要先了解下Dagger2中的注解。Dagger2使用过程中我们通常接触到的注解主要包括：@Inject, @Module, @Provides, @Component, @Qulifier, @Scope, @Singleten。

* @Inject：@Inject有两个作用，一是用来标记需要依赖的变量，以此告诉Dagger2为它提供依赖；二是用来标记构造函数，Dagger2通过@Inject注解可以在需要这个类实例的时候来找到这个构造函数并把相关实例构造出来，以此来为被@Inject标记了的变量提供依赖；

* @Module：@Module用于标注提供依赖的类。你可能会有点困惑，上面不是提到用@Inject标记构造函数就可以提供依赖了么，为什么还需要@Module？很多时候我们需要提供依赖的构造函数是第三方库的，我们没法给它加上@Inject注解，又比如说提供以来的构造函数是带参数的，如果我们之所简单的使用@Inject标记它，那么他的参数又怎么来呢？@Module正是帮我们解决这些问题的。

* @Provides：@Provides用于标注Module所标注的类中的方法，该方法在需要提供依赖时被调用，从而把预先提供好的对象当做依赖给标注了@Inject的变量赋值；

* @Component：@Component用于标注接口，是依赖需求方和依赖提供方之间的桥梁。被Component标注的接口在编译时会生成该接口的实现类（如果@Component标注的接口为CarComponent，则编译期生成的实现类为DaggerCarComponent）,我们通过调用这个实现类的方法完成注入； 

* @Qulifier：@Qulifier用于自定义注解，也就是说@Qulifier就如同Java提供的几种基本元注解一样用来标记注解类。我们在使用@Module来标注提供依赖的方法时，方法名我们是可以随便定义的（虽然我们定义方法名一般以provide开头，但这并不是强制的，只是为了增加可读性而已）。那么Dagger2怎么知道这个方法是为谁提供依赖呢？答案就是返回值的类型，Dagger2根据返回值的类型来决定为哪个被@Inject标记了的变量赋值。但是问题来了，一旦有多个一样的返回类型Dagger2就懵逼了。@Qulifier的存在正式为了解决这个问题，我们使用@Qulifier来定义自己的注解，然后通过自定义的注解去标注提供依赖的方法和依赖需求方（也就是被@Inject标注的变量），这样Dagger2就知道为谁提供依赖了。----一个更为精简的定义：当类型不足以鉴别一个依赖的时候，我们就可以使用这个注解标示；

* @Scope：@Scope同样用于自定义注解，我能可以通过@Scope自定义的注解来限定注解作用域，实现局部的单例；

* @Singleton：@Singleton其实就是一个通过@Scope定义的注解，我们一般通过它来实现全局单例。但实际上它并不能提前全局单例，是否能提供全局单例还要取决于对应的Component是否为一个全局对象。

我们提到@Inject和@Module都可以提供依赖，那如果我们即在构造函数上通过标记@Inject提供依赖，有通过@Module提供依赖Dagger2会如何选择呢？具体规则如下：

* 步骤1：首先查找@Module标注的类中是否存在提供依赖的方法。
* 步骤2：若存在提供依赖的方法，查看该方法是否存在参数。
	* a：若存在参数，则按从步骤1开始依次初始化每个参数；
	* b：若不存在，则直接初始化该类实例，完成一次依赖注入。
* 步骤3：若不存在提供依赖的方法，则查找@Inject标注的构造函数，看构造函数是否存在参数。
	* a：若存在参数，则从步骤1开始依次初始化每一个参数
	* b：若不存在，则直接初始化该类实例，完成一次依赖注入。

## Dagger2使用入门

前面长篇大论的基本都在介绍概念，下面我们看看Dagger2的基本应用。关于Dagger2的依赖配置就不在这里占用篇幅去描述了，大家可以到它的github主页下去查看官方教程[https://github.com/google/dagger](https://github.com/google/dagger)。接下来我们还是拿前面的Car和Engine来举例。

#### 1、案例A

Car类是需求依赖方，依赖了Engine类；因此我们需要在类变量Engine上添加@Inject来告诉Dagger2来为自己提供依赖。

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

Engine类是依赖提供方，因此我们需要在它的构造函数上添加@Inject

```Java
public class Engine {
    
    @Inject
    Engine(){}
    
    public void run(){
        System.out.println("引擎转起来了~~~");
    }
}
```

接下来我们需要创建一个用@Component标注的接口CarComponent，这个CarComponent其实就是一个注入器，这里用来将Engine注入到Car中。

```Java
@Component
public interface CarComponent {
    void inject(Car car);
}
```

完成这些之后我们需要Build下项目，让Dagger2帮我们生成相关的Java类。接着我们就可以在Car的构造函数中调用Dagger2生成的DaggerCarComponent来实现注入（这其实在前面Car类的代码中已经有了体现）

```Java
public Car() {
    DaggerCarComponent.builder().build().inject(this);
}
```

#### 2、案例B

**如果创建Engine的构造函数是带参数的呢？比如说制造一台引擎是需要齿轮(Gear)的。或者Eggine类是我们无法修改的呢？这时候就需要@Module和@Provide上场了。**

同样我们需要在Car类的成员变量Engine上加上@Inject表示自己需要Dagger2为自己提供依赖；Engine类的构造函数上的@Inject也需要去掉，应为现在不需要通过构造函数上的@Inject来提供依赖了。

```Java
public class Car {

    @Inject
    Engine engine;

    public Car() {
        DaggerCarComponent.builder().markCarModule(new MarkCarModule())
                .build().inject(this);
    }

    public Engine getEngine() {
        return this.engine;
    }
}
```

接着我们需要一个Module类来生成依赖对象。前面介绍的@Module就是用来标准这个类的，而@Provide则是用来标注具体提供依赖对象的方法（这里有个不成文的规定，被@Provide标注的方法命名我们一般以provide开头，这并不是强制的但有益于提升代码的可读性）。

```Java
@Module
public class MarkCarModule {

    public MarkCarModule(){ }

    @Provides Engine provideEngine(){
        return new Engine("gear");
    }
}
```

接下来我们还需要对CarComponent进行一点点修改，之前的@Component注解是不带参数的，现在我们需要加上`modules = {MarkCarModule.class}`，用来告诉Dagger2提供依赖的是`MarkCarModule`这个类。

```Java
@Component(modules = {MarkCarModule.class})
public interface CarComponent {
    void inject(Car car);
}
```

Car类的构造函数我们也需要修改，相比之前多了个`markCarModule(new MarkCarModule())`方法，这就相当于告诉了注入器`DaggerCarComponent`把`MarkCarModule`提供的依赖注入到了Car类中。

```Java
public Car() {
   DaggerCarComponent.builder()
           .markCarModule(new MarkCarModule())
           .build().inject(this);
}
```

这样一个最最基本的依赖注入就完成了，接下来我们测试下我们的代码。

```Java
public static void main(String[] args){
    Car car = new Car();
    car.getEngine().run();
}
```

输出

```Text
引擎转起来了~~~
```

#### 3、案例C

那么如果一台汽车有两个引擎（也就是说Car类中有两个Engine变量）怎么办呢？没关系，我们还有@Qulifier！首先我们需要使用Qulifier定义两个注解：

```Java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface QualifierA { }
```

```Java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface QualifierB { }
```

同时我们需要对依赖提供方做出修改

```Java
@Module
public class MarkCarModule {

    public MarkCarModule(){ }

    @QualifierA
    @Provides
    Engine provideEngineA(){
        return new Engine("gearA");
    }

    @QualifierB
    @Provides
    Engine provideEngineB(){
        return new Engine("gearB");
    }
}
```

接下来依赖需求方Car类同样需要修改

```Java
public class Car {

    @QualifierA @Inject Engine engineA;
    @QualifierB @Inject Engine engineB;

    public Car() {
        DaggerCarComponent.builder().markCarModule(new MarkCarModule())
                .build().inject(this);
    }

    public Engine getEngineA() {
        return this.engineA;
    }

    public Engine getEngineB() {
        return this.engineB;
    }
}
```

最后我们再对Engine类做些调整方便测试

```Java	
public class Engine {

    private String gear;
    
    public Engine(String gear){
        this.gear = gear;
    }
    
    public void printGearName(){
        System.out.println("GearName:" + gear);
    }
}
```

测试代码

```Java
public static void main(String[] args) {
    Car car = new Car();
    car.getEngineA().printGearName();
    car.getEngineB().printGearName();
}
```

执行结果：

```Text
GearName:gearA
GearName:gearB
```

#### 4、案例D

接下来我们看看@Scope是如何限定作用域，实现局部单例的。

首先我们需要通过@Scope定义一个CarScope注解：

```Java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface CarScope {
}
```

接着我们需要用这个@CarScope去标记依赖提供方MarkCarModule。

```Java
@Module
public class MarkCarModule {

    public MarkCarModule() {
    }

    @Provides
    @CarScope
    Engine provideEngine() {
        return new Engine("gear");
    }
}
```

同时还需要使用@Scope去标注注入器Compoent

```Java
@CarScope
@Component(modules = {MarkCarModule.class})
public interface CarComponent {
    void inject(Car car);
}
```

为了便于测试我们对Car和Engine类做了一些改造：

```Java
public class Car {

    @Inject Engine engineA;
    @Inject Engine engineB;

    public Car() {
        DaggerCarComponent.builder()
                .markCarModule(new MarkCarModule())
                .build().inject(this);
    }
}
```

```Java
public class Engine {

    private String gear;
    
    public Engine(String gear){
        System.out.println("Create Engine");
        this.gear = gear;
    }
}
```

如果我们不适用@Scope,上面的代码会实例化两次Engine类，因此会有两次\"Create Engine\"输出。现在我们在有@Scope的情况测试下劳动成果：

```Java
public static void main(String[] args) {
    Car car = new Car();

    System.out.println(car.engineA.hashCode());
    System.out.println(car.engineB.hashCode());
}
```

输出

```Java
Create Engine
```

bingo！我们确实通过@Scope实现了局部的单例。

## Dagger2原理分析

前面啰里啰嗦的介绍了Dagger2的基本使用，接下来我们再分析分析实现原理。这里不会分析Dagger2根据注解生成各种代码的原理，关于Java注解以后有机会再写一篇文章来介绍。后面主要分析的是Dagger2生成的各种类如何帮我们实现依赖注入，为了便于理解我这里选了前面相对简单的**案例B**来做分析。

Dagger2编译期生成的代码位于`build/generated/source/apt/debug/your package name/`下面:
![Generated Code](generated_code.png)

首先我们看看Dagger2依据依赖提供方`MarkCarModule`生成的对应工厂类`MarkCarModule_ProvideEngineFactory`。为了方便大家理解对比，后面我一律会把自己写的类和Dagger2生成的类一并放出来。

```Java
/**
* 我们自己的类
*/
@Module
public class MarkCarModule {

    public MarkCarModule(){ }

    @Provides Engine provideEngine(){
        return new Engine("gear");
    }
}

/**
* Dagger2生成的工厂类
*/
public final class MarkCarModule_ProvideEngineFactory implements Factory<Engine> {
  private final MarkCarModule module;

  public MarkCarModule_ProvideEngineFactory(MarkCarModule module) {
    assert module != null;
    this.module = module;
  }

  @Override
  public Engine get() {
    return Preconditions.checkNotNull(
        module.provideEngine(), "Cannot return null from a non-@Nullable @Provides method");
  }

  public static Factory<Engine> create(MarkCarModule module) {
    return new MarkCarModule_ProvideEngineFactory(module);
  }

  /** Proxies {@link MarkCarModule#provideEngine()}. */
  public static Engine proxyProvideEngine(MarkCarModule instance) {
    return instance.provideEngine();
  }
}
```

我们可以看到`MarkCarModule_ProvideEngineFactory`中的get()调用了`MarkCarModule`的`provideEngine()`方法来获取我们需要的依赖`Engine`，`MarkCarModule_ProvideEngineFactory`的实例化有`crate()`创建，并且`MarkCarModule`的实例也是通过`create()`方法传进来的。那么这个`create()`一定会在哪里调用的，我们接着往下看。

前面提到@Component是依赖提供方(MarkCarModule)和依赖需求方(Car)之前的桥梁，那我看看Dagger2是如何通过CarComponent将两者联系起来的。

```Java
/**
* 我们自己的类
*/
@Component(modules = {MarkCarModule.class})
public interface CarComponent {

    void inject(Car car);
}

/**
* Dagger2生成的CarComponent实现类
*/
public final class DaggerCarComponent implements CarComponent {
  private Provider<Engine> provideEngineProvider;

  private MembersInjector<Car> carMembersInjector;

  private DaggerCarComponent(Builder builder) {
    assert builder != null;
    initialize(builder);
  }

  public static Builder builder() {
    return new Builder();
  }

  public static CarComponent create() {
    return builder().build();
  }

  @SuppressWarnings("unchecked")
  private void initialize(final Builder builder) {

    this.provideEngineProvider = MarkCarModule_ProvideEngineFactory.create(builder.markCarModule);

    this.carMembersInjector = Car_MembersInjector.create(provideEngineProvider);
  }

  @Override
  public void inject(Car car) {
    carMembersInjector.injectMembers(car);
  }

  public static final class Builder {
    private MarkCarModule markCarModule;

    private Builder() {}

    public CarComponent build() {
      if (markCarModule == null) {
        this.markCarModule = new MarkCarModule();
      }
      return new DaggerCarComponent(this);
    }

    public Builder markCarModule(MarkCarModule markCarModule) {
      this.markCarModule = Preconditions.checkNotNull(markCarModule);
      return this;
    }
  }
}
```

通过上面的代码我们看到Dagger2依据`CarComponent`接口生成了实现类`DaggerCarComponent`（没错这正是我们在Car的构造函数中使用DaggerCarComponent）。`DaggerCarComponent`在build的时候实例化了`DaggerCarComponent`对象，并首先调用`MarkCarModule_ProvideEngineFactory.create(builder.markCarModule)`始化了`provideEngineProvider`变量，接着调用`Car_MembersInjector.create(provideEngineProvider)`初始化了`carMembersInjector`变量。当我们手动在Car类的构造函数中调用`inject(Car car)`方法时会执行`carMembersInjector.injectMembers(car)`。所以接下来我们要看看`Car_MembersInjector`的实现。

```Java
public final class Car_MembersInjector implements MembersInjector<Car> {
  private final Provider<Engine> engineProvider;

  public Car_MembersInjector(Provider<Engine> engineProvider) {
    assert engineProvider != null;
    this.engineProvider = engineProvider;
  }

  public static MembersInjector<Car> create(Provider<Engine> engineProvider) {
    return new Car_MembersInjector(engineProvider);
  }

  @Override
  public void injectMembers(Car instance) {
    if (instance == null) {
      throw new NullPointerException("Cannot inject members into a null reference");
    }
    instance.engine = engineProvider.get();
  }

  public static void injectEngine(Car instance, Provider<Engine> engineProvider) {
    instance.engine = engineProvider.get();
  }
}
```

`Car_MembersInjector`中的`create()`用于实例化自己，这个方法前面我们看到是在`DaggerCarComponent`中调用的。`injectMembers(Car instance)`将`engineProvider.get()`的返回值赋给了依赖需求方Car的engine变量，而`engineProvider.get()`正是本节一开始我们提到的`MarkCarModule_ProvideEngineFactory`中的`get()`方法。至此整个依赖注入的流程就完成了。更复杂的应用场景会生成更加复杂的代码，但原理都和前面分析的大同小异。

## 总结

这篇文章只是通过一些简单的例子介绍了Dagger2的相关概念及使用，实际项目中的应用远比这里的例子要复杂。关于Dagger2在实际项目中的应用可以参照这个开源项目 [https://github.com/BaronZ88/MinimalistWeather](https://github.com/BaronZ88/MinimalistWeather)（项目采用MVP架构，其中View层和Presenter层的解耦就是通过Dagger2来实现的）。

> [MinimalistWeather](https://github.com/BaronZ88/MinimalistWeather)是一款开源天气App，开发此项目主要是为展示各种开源库的使用方式以及Android项目的架构方案，并作为团队开发规范的一部分。项目中每一个字母、每一个命名、每一行代码都是经过仔细考究的；但是由于时间精力有限，项目UI未做严格要求。本着精益求精、提供更好开源项目和更美天气应用的原则，因此期望有兴趣的开发和UED同学可以一起来完成这个项目。



