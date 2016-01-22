
###Java8新特性第3章(Stream API)
> 转载请注明出处：[http://www.jianshu.com/p/e3ba9a0b7d72](http://www.jianshu.com/p/e3ba9a0b7d72)

***

Java Lambda表达式出现后，Java集合API显得有点过时了，所以Java8中对集合框架的类库进行了相关的扩展：

* 添加新的类库Stream(java.util.stream.Stream)和Collector(java.util.stream.Collector)；
* 为现有接口(Collection,List等)扩展新功能；
* 改造现有的功能使之可以提供流视图。

假设我们需要把一个集合中的所有形状设置成红色，那么我们可以这样写

	for (Shape shape : shapes){
		shape.setColor(RED)
	}
	
如果使用Java8扩展后的集合框架则可以这样写：

	shapes.foreach(s -> s.setColor(RED));

__第一种__写法我们叫外部迭代，for-each调用`shapes`的`iterator()`依次遍历集合中的元素。这种外部迭代有一些问题：

* for循环是串行的，而且必须按照集合中元素的顺序依次进行；
* 集合框架无法对控制流进行优化，例如通过排序、并行、短路求值以及惰性求值改善性能。
> 上面这两个问题我们会在后面的文章中逐步解答。
	
__第二种__写法我们叫内部迭代，两段代码虽然看起来只是语法上的区别，但实际上他们内部的区别其实非常大。用户把对操作的控制权交还给类库，从而允许类库进行各种各样的优化（例如乱序执行、惰性求值和并行等等）。总的来说，内部迭代使得外部迭代中不可能实现的优化成为可能。

外部迭代同时承担了做什么（把形状设为红色）和怎么做（得到Iterator实例然后依次遍历），而内部迭代只负责做什么，而把怎么做留给类库。这样代码会变得更加清晰，而集合类库则可以在内部进行各种优化。


####1.Stream
Java8中新增了Stream(java.util.stream.Stream)，java.util.stream中包含了若干xxxStream(IntStream,LongStream等)。Stream提供了强大的数据集合操作功能，并被深入整合到现有的集合类和其它的JDK类型中。

流的操作可以被组合成流水线（Pipeline）。拿前面的例子来说，如果我只想把蓝色改成红色：

	shapes.stream()
      	  .filter(s -> s.getColor() == BLUE)
      	  .forEach(s -> s.setColor(RED));

在`Collection`上调用`stream()`会生成该集合元素的流，接下来`filter()`操作会产生只包含蓝色形状的流，最后，这些蓝色形状会被`forEach`操作设为红色。

如果我们想把蓝色的形状提取到新的List里，则可以：

	List<Shape> blue = shapes.stream()
							  .filter(s -> s.getColor() == BLUE)
							  .collect(Collectors.toList());

`collect()`操作会把其接收的元素聚集到一起（这里是List），`collect()`方法的参数则被用来指定如何进行聚集操作。在这里我们使用`toList()`以把元素输出到List中。

如果每个形状都被保存在`Box`里，然后我们想知道哪个盒子至少包含一个蓝色形状，我们可以这么写：

	Set<Box> hasBlueShape = shapes.stream()
								   .filter(s -> s.getColor() == BLUE)
                                  .map(s -> s.getContainingBox())
                                  .collect(Collectors.toSet());
                                  
`map()`操作通过映射函数（这里的映射函数接收一个形状，然后返回包含它的盒子）对输入流里面的元素进行依次转换，然后产生新流。

如果我们需要得到蓝色物体的总重量，我们可以这样表达：

	int sum = shapes.stream()
                    .filter(s -> s.getColor() == BLUE)
                    .mapToInt(s -> s.getWeight())
                    .sum();
               

####2.Stream vs Collection
流（Stream）和集合（Collection）的区别：

* Collection主要用来对元素进行管理和访问；
* Stream并不支持对其元素进行直接操作和直接访问，而只支持通过声明式操作在其之上进行运算后得到结果；
* Stream不存储值
* 对Stream的操作会产生一个结果，但是Stream并不会改变数据源；
* 大多数Stream的操作(filter,map,sort等)都是以惰性的方式实现的。这使得我们可以使用一次遍历完成整个流水线操作,并可以用短路操作提供更高效的实现。

####3.惰性求值 vs 急性求值
`filter()`和`map()`这样的操作既可以被急性求值（以`filter()`为例，急性求值需要在方法返回前完成对所有元素的过滤），也可以被惰性求值（用`Stream`代表过滤结果，当且仅当需要时才进行过滤操作）在实际中进行惰性运算可以带来很多好处。比如说，如果我们进行惰性过滤，我们就可以把过滤和流水线里的其它操作混合在一起，从而不需要对数据进行多遍遍历。相类似的，如果我们在一个大型集合里搜索第一个满足某个条件的元素，我们可以在找到后直接停止，而不是继续处理整个集合。（这一点对无限数据源是很重要，惰性求值对于有限数据源起到的是优化作用，但对无限数据源起到的是决定作用，没有惰性求值，对无限数据源的操作将无法终止）

对于`filter()`和`map()`这样的操作，我们很自然的会把它当成是惰性求值操作，不过它们是否真的是惰性取决于它们的具体实现。另外，像`sum()`这样生成值的操作和`forEach()`这样产生副作用的操作都是__天然急性求值__，因为它们必须要产生具体的结果。

我们拿下面这段代码举例：

	int sum = shapes.stream()
                    .filter(s -> s.getColor() == BLUE)
                    .mapToInt(s -> s.getWeight())
                    .sum();
                    
这里的`filter()`和`map()`都是惰性的，这就意味着在调用`sum()`之前不会从数据源中提取任何元素。在`sum()`操作之后才会把`filter()`、`map()`和`sum()`放在对数据源一次遍历中。这样可以大大减少维持中间结果所带来的开销。

<!--####6.流水线(Pipeline)的并行操作
流水线可以是串行的也可以是并行的，串行和并行是流的属性。默认情况下数据源返回的都是串行流，但是我们可以通过`parallel()`将串行流转换为并行流,就像下面这样：

	int sum = shapes.parallelStream()
                .filter(s -> s.getColor = BLUE)
                .mapToInt(s -> s.getWeight())
                .sum();
那么，串行流和并行流有什么区别呢？

流的数据源可能是一个可变集合，如果当我们在遍历流时数据源被改变了，那么就会产生干扰。所以在进行流操作的时候，数据源应该保持不变。如果在单线程模型下，我们只需要保证lambda表达式不修改流的数据源就OK了；但如果是多线程环境，lambda在执行时可能会同时运行在多个线程上-->

####4.举个栗子🌰
前面长篇大论的介绍概念实在太枯燥，为了方便大家理解我们用Streams API来实现一个具体的业务场景。

假设我们有一个房源库项目，这个房源库中有一系列的小区，每个小区都有小区名和房源列表，每套房子又有价格、面积等属性。现在我们需要筛选出含有100平米以上房源的小区，并按照小区名排序。

我们先来看看不用Streams API如何实现：

	List<Community> result = new ArrayList<>();
    for (Community community : communities) {
            for (House house : community.houses) {
                if (house.area > 100) {
                    result.add(community);
                    break;
                }
            }
        }
        Collections.sort(result, new Comparator<Community>() {
            @Override
            public int compare(Community c1, Community c2) {
                return c1.name.compareTo(c2.name);
            }
        });
        return result;
        
        
        
如果使用Streams API:

	return communities.stream()
	                  .filter(c -> c.houses.stream().anyMatch(h -> h.area>100))
                      .sorted(Comparator.comparing(c -> c.name))
                      .collect(Collectors.toList());


