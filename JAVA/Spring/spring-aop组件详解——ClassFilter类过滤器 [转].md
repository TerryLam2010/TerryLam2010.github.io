## spring-aop组件详解——ClassFilter类过滤器 [转]

原网址：<https://my.oschina.net/lixin91/blog/684918> 

术语定义：

1. ClassFilter：类过滤器
2. Advisor：通知器
3. targetClass：目标类，或称被代理的原始类
4. Advice：通知，或称拦截器，也就是要增强的代码逻辑
5. MethodMatcher：方法匹配器
6. Pointcut：切点，由ClassFilter和MethodMatcher组成

​        ClassFilter，用于约束一个Advisor，与指定的targetClass是否匹配。只有匹配的前提下，Advisor才能使用其内部持有的Advice对targetClass进行增强。

​        Advisor分两大类：IntroductionAdvisor（引介通知器）和PointcutAdvisor（切点通知器）。两类Advisor都是为了增强targetClass，但是作用不一样。IntroductionAdvisor主要为了给targetClass追加接口（或者说追加更多的方法），这种增强属于类级别的增强；而PointcutAdvisor主要为了拦截方法，这种增强属于方法级别的增强。

​        正是由于两类Advisor的增强级别不同，而导致了对ClassFilter的使用方式不同。IntroductionAdvisor进行类级别增强，因此只需要直接持有ClassFilter即可；而PointcutAdvisor进行方法级别增强，因此需要同时使用ClassFilter和MethodMatcher（方法匹配器）。PointcutAdvisor内部持有一个Pointcut，而Pointcut就是由ClassFilter和MethodMatcher组成的。

​        ClassFilter只定义了唯一的方法，代码如下：

```
	/**
	 * 当前类过滤器是否与指定的类型匹配，
	 * 如果匹配，则需要把自身内部持有的Advice应用的目标类上。
	 * 
	 * @param clazz 候选目标类
	 * @return Advice通知是否应该应用到指定的目标类
	 */
	public abstract boolean matches(Class<?> clazz);
```

​        ClassFilter组件的体系图如下：

![img](http://static.oschina.net/uploads/space/2016/0531/162342_a5Oq_2624635.png)

 

从图中可以看出，ClassFilter有4中简单方式的实现：

1. TypePatternClassFilter：基于AspectJ的类型匹配实现
2. AnnotationClassFilter：通过检查目标类是否存在指定的注解，决定是否匹配
3. RootClassFilter：通过判断目标类是否是指定类型（或其子类型），决定是否匹配
4. TrueClassFilter：这是最简单实现，matches方法总会返回true。此类设计使用了单例模式，且其对象引用直接被在ClassFilter接口中声明成了常量。

​        除此之外，还有两种特殊方式的实现：

1. DefaultIntroductionAdvisor：默认的引介通知器，它是一种通知器，但同时兼具了类过滤器的功能，且matches总返回true。它的作用是给所有bean追加指定接口。
2. AspectJExpressionPointcut：AspectJ表达式切点（通过解析XML配置文件中的<aop:pointcut>元素生成的就是此类型的bean）。它是一种切点，但与一般的切点不同。一般的切点需要持有单独的ClassFilter和MethodMatcher。但是AspectJ表达式切点本身就兼具了这两个组件的功能。因为切点表达式，就是用来描述要代理的目标类和目标方法的。

​        另外，针对ClassFilter，还有一个工具类——ClassFilters。ClassFilters内部定义了两个私有的静态内部类：IntersectionClassFilter和UnionClassFilter，分别支持以与的逻辑和或的逻辑组合多个ClassFilter。此工具类同时对外提供了组合ClassFilter的API。

![img](http://static.oschina.net/uploads/space/2016/0531/164343_WIRy_2624635.png)

​    通过以上介绍，我们已经可以通过注解匹配、类型匹配等方式进行目标类的选取了，还可以通过复杂的与或逻辑进一步精确的选取目标类。当然，我们也可以通过实现ClassFilter接口，自定义自己的匹配逻辑。但是，通常不建议这么做，因为使用AspectJ切点表达式选取目标类已经足够强大了。