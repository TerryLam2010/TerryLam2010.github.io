## spring-aop组件详解——Advice通知[转]

<https://my.oschina.net/lixin91/blog/685532> 

​       Advice（通知）是面向切面编程中的一个非常重要的概念。我们都知道，AOP的目的在于对目标类或目标方法的逻辑增强（如：日志逻辑、统计逻辑、访问控制逻辑等），那么Advice就代表要增强的具体逻辑。Advice接口由AOP联盟（aopalliance.org）定义，它只是一个标记接口，用来强调概念，没有定义任何功能（或者说没有定义增强方式或增强内容）。

Advice体系图如下：

![img](http://static.oschina.net/uploads/space/2016/0601/154124_VXys_2624635.png)

​        AOP联盟在Advice的基础上扩展定义了子接口——Interceptor（拦截器）。拦截器定义了通知的增强方式，也就是通过对Joinpoint（连接点）的拦截。AOP联盟的原话是这样的：

> A generic interceptor can intercept runtime events that occur within a base program. Those events are materialized by (reified in) joinpoints. Runtime joinpoints can be invocations, field access, exceptions...
>
> 以下是我的翻译：
>
> 一个通用的拦截器可以拦截发生在基础程序中的运行时事件。这些事件被连接点具体化。运行时连接点可以是一次方法调用、字段访问、异常产生等等。

​        很明显，Interceptor接口也在强调概念而非功能，也是一个标记接口。 由Interceptor扩展出的ConstructorInterceptor和MethodInterceptor两个子接口，才具体定义了拦截方式。它们一个用于拦截构造方法，一个用于拦截普通方法。代码如下：

```
public interface ConstructorInterceptor extends Interceptor {

	Object construct(ConstructorInvocation invocation) throws Throwable;
	
}
public interface MethodInterceptor extends Interceptor {
	
    Object invoke(MethodInvocation invocation) throws Throwable;
   
}
```

​        但是，spring框架并没有支持AOP联盟对构造方法的拦截，原因很简单，spring框架本身，通过BeanPostProcessor的定义，对bean的生命周期扩展已经很充分了。

​        MethodInterceptor只定义了增强方式，我们可以通过实现此接口，自定义具体的增强内容。当然，spring框架也提供了3种预定义的增强内容——BeforeAdvice（前置通知）、AfterAdvice（后置通知）和DynamicIntroductionAdvice（动态引介通知）。BeforeAdvice和AfterAdvice更确切地说是定义了增强内容的执行时机（方法调用之前还是之后）；而DynamicIntroductionAdvice比较特殊，它可以编辑目标类要实现的接口列表。最后，spring预定义的通知还是要通过对应的适配器，适配成MethodInterceptor接口类型的对象（如：MethodBeforeAdviceInterceptor负责适配MethodBeforeAdvice）。

​        既然MethodInterceptor是核心，那么下面重点介绍以下MethodInterceptor的体系，如下图：

![img](http://static.oschina.net/uploads/space/2016/0601/161009_ajuL_2624635.png)

重点介绍几个常用拦截器（其他的读者可自行研究）：

1. MethodBeforeAdviceInterceptor：
           MethodBeforeAdvice（前置通知，其父接口是BeforeAdvice）接口的适配器，用于支持spring预定义的前置通知，在目标方法调用前调用MethodBeforeAdvice.before()。
2. AspectJAfterAdvice ：
           AspectJ框架的后置通知实现，在目标方法执行结束后，return之前，调用配置指定的方法（注意：此方法调用被写在finally块中，无论如何都会得到执行）。
3. AfterReturningAdviceInterceptor ：
           AfterReturningAdvice接口的适配器，用于支持spring预定义的后置通知，在目标方法执行结束后，return之前，调用AfterReturningAdvice.afterReturning()执行（注意：如果目标方法抛出异常，则不会执行这个方法）。
4. AspectJAfterThrowingAdvice ：
           AspectJ框架的异常通知，当目标方法执行时产生异常的时候，指定配置指定的方法。
5. AspectJAroundAdvice ：
           AspectJ框架的环绕通知，直接执行配置指定的方法。
6. ThrowsAdviceInterceptor ：
           spring框架预定义的异常通知的适配器，此适配器接受任意类型的对象，但是要求对象所在类必须声明public的名称为afterThrowing，且参数个数为1个或4个，且最后一个参数为Throwable类型的方法。该适配器会保存该Throwable对象的实际类型到该方法之间的映射，当目标方法执行产生异常时，根据产生的异常类型找到对应的通知方法进行调用。
7. DelegatingIntroductionInterceptor ：
           通过构造方法传入指定的引介对象，每当调用的目标方法是引介接口定义的方法时，都会调用该引介对象的对应方法。
8. DelegatePerTargetObjectIntroductionInterceptor ：
           通过构造函数传入指定的引介接口和接口对应的实现类，该拦截器会为每个目标对象创建新的引介对象（通过调用实现类的默认无参构造）。当调用的方法是引介接口定义的方法时，则调用该新建的引介对象对应的方法。