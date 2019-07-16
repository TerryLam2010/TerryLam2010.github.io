## spring-aop组件详解——Pointcut切点[转]

<https://my.oschina.net/lixin91/blog/686660> 

Pointcut（切点）是面向切面编程中的一个非常重要的概念，此概念由spring框架定义。Pointcut的唯一作用就是筛选要拦截的目标方法，因此，有很多人会把Pointcut直接理解成——要拦截的方法，其实不然，Pointcut只是一种筛选规则（或者叫过滤器）。

​        Pointcut由ClassFilter（类过滤器）和MethodMatcher（方法匹配器）两个组件组成。ClassFilter检查当前筛选规则与目标类是否匹配，MethodMatcher检查当前筛选规则与目标方法是否匹配。两个组件的共同作用，可以筛选出一个符合既定规则的方法的集合。通过Advisor（通知器）和Advice（通知）和Pointcut（切点）组合起来，就可以把指定的通知应用到指定的方法集合上。

​        Pointcut定义源码如下：

```
public interface Pointcut {

	/**
	 * 返回当前切点持有的类过滤器，不能返回null值。
	 * @return
	 */
	ClassFilter getClassFilter();

	/**
	 * 返回当前切点持有的方法匹配器，不能返回null值
	 * @return
	 */
	MethodMatcher getMethodMatcher();

	/**
	 * 一个最简单切点的实现。
	 * 这个切点由ClassFilter.TRUE和MethodMatcher.TRUE两个组件组成
	 * 它匹配所有的目标类和目标方法。
	 */
	Pointcut TRUE = TruePointcut.INSTANCE;

}
```

​        知道了Pointcut的基本作用，接下来我们需要了解一下spring到底有哪些具体的实现，分别头什么用。首先看下组件体系图：![img](http://static.oschina.net/uploads/space/2016/0603/141435_bGUm_2624635.png) 

由图可知，spring对Pointcut接口作了一些基本的简单实现。除此之外，还从Pointcut接口扩展出了一类特殊且重要的切点（或者说筛选方式）——ExpressionPointcut，也就是通过String类型的表达式来描述切点筛选规则。注意：spring只是规定了使用表达式的方式进行方法筛选，但是并没有为表达式定义语法，而其子类AspectJExpressionPointcut才具体限定了表达式的语法，而语法的规则由AspectJ框架定义（具体的语法规则不是本文要关注的重点，感兴趣的读者可自行研究）。

​        下面重点介绍几个常用的切点：

1. AnnotationMatchingPointcut：
   注解匹配切点。根据类上或方法上是否存在指定的注解判断切点的匹配性，如果没有显示指定注解，则匹配所有。
2. DynamicMethodMatcherPointcut：
   动态方法匹配器切点。它本质上是一个方法匹配器，但同时具有了切点的功能。
3. ComposablePointcut：
   可组合的切点。这种切点可以与或逻辑，任意组合其他的Pointcut、ClassFilter和MethodMatcher。其本质是通过ClassFilters和MethodMatchers两个工具类进行Pointcut内部组件的组合。
4. JdkRegexpMethodPointcut：
   JDK正则表达式切点，即使用正则表达式描述方法的拦截规则和排除规则。
5. AspectJExpressionPointcut：
   AspectJ切点表达式切点。顾名思义，使用AspectJ的切点表达式描述筛选规则。表达式基本语法如下（非完整语法）：

```
execution(<方法修饰符>? <方法返回值类型> <包名>.<类名>.<方法名>(<参数类型>) [throws <异常类型>]?)
```

1. 其中，‘*’代表0个或多个任意字符，包名中的..（两个点）代表当前包及其子包，参数列表中的..代表任意个参数。
   举个例子，如：execution(public static * *..*.*(..) throws *)，此表达式匹配所有方法。

​        最后，我们看一下如何使用XML配置文件配置切点，以及配置的切点最终是Pointcut接口的哪个具体实现类型。XML配置文件如下（我们只关注对切点的配置）：

```
<aop:config>
	<aop:pointcut expression=""/>
	<aop:advisor pointcut="" pointcut-ref=""/>
	<aop:aspect>
		<aop:pointcut expression=""/>
	</aop:aspect>
</aop:config>
```

 <pointcut>元素可以直接配置一个切点，其expression属性值是一个AspectJ切点表达式，元素最终生成的bean的实际类型为AspectJExpressionPointcut。

​        <advisor>元素在配置一个通知器，通知器由一个Advice和一个Pointcut组成，此元素的pointcut属性和pointcut-ref属性都可以配置切点，但是两个属性只能同时配置一个。pointcut属性值依然是AspectJ表达式，实际bean类型也为AspectJExpressionPointcut，且它是一个没有beanName的bean对象，只供当前通知器使用（这相当于配置bean属性时<property>元素嵌套<bean>子元素）。但是pointcut-ref属性值不同，它是一个beanName，也就是运行时对容器中bean的引用，它引用的bean对象可以是Pointcut接口的任意实现类型。也就是说，我们可以通过<bean>元素定义一个自定义切点bean，再通过<advisor>元素让通知器引用这个bean即可。

​        <aspect>配置的是切面bean，也就是自定义的拦截逻辑。其<pointcut>子元素与<config>的<pointcut>子元素作用完全相同。

​        现在，我们可以自有定制自己的方法筛选逻辑了。