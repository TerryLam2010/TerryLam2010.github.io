## spring-aop组件详解——TargetSource目标源 【转】

<https://my.oschina.net/lixin91/blog/688188> 

​        TargetSource（目标源）是被代理的target（目标对象）实例的来源。TargetSource被用于获取当前MethodInvocation（方法调用）所需要的target（目标对象），这个target通过反射的方式被调用（如：method.invode(target,args)）。**换句话说，proxy（代理对象）代理的不是target，而是TargetSource，这点非常重要！！！**

​        那么问题来了：为什么SpringAOP代理不直接代理target，而需要通过代理TargetSource（target的来源，其内部持有target），间接代理target呢？

​        通常情况下，一个proxy（代理对象）只能代理一个target，每次方法调用的目标也是唯一固定的target。但是，如果让proxy代理TargetSource，可以使得每次方法调用的target实例都不同（当然也可以相同，这取决于TargetSource实现）。这种机制使得方法调用变得灵活，可以扩展出很多高级功能，如：target pool（目标对象池）、hot swap（运行时目标对象热替换），等等。

​        接下来要说的另外一点，可能会颠覆你的既有认知：TargetSource组件本身与SpringIoC容器无关，换句话说，target的生命周期不一定是受spring容器管理的，我们以往的XML中的AOP配置，只是对受容器管理的bean而言的，我们当然可以手动创建一个target，同时使用Spring的AOP框架（而不使用IoC容器）。Demo如下：

```
// 首先定义一个被代理的目标类
public class TargetBean {
    // 此方法演示使用
	public void show() {
		System.out.println("show");
	}
}
// 接下来是创建代理对象的过程
public class AOPDemo {

	public static void main(String[] args) {
		TargetBean target = new TargetBean();
		TargetSource targetSource = new SingletonTargetSource(target);
		// 使用SpringAOP框架的代理工厂直接创建代理对象
		TargetBean proxy = (TargetBean) ProxyFactory.getProxy(targetSource);
        // 这里会在控制台打印：com.lixin.aopdemo.TargetBean$$EnhancerBySpringCGLIB$$767606b3
		System.out.println(proxy.getClass().getName());
	}

}
```

​        这个demo只是创建target的代理对象，并没有添加任何增强逻辑。，从输出可以看到该目标类已经被CGLIB增强。整个代理过程十分简单，没有使用任何BeanFactory或ApplicationContext一样可以完成代理。

TargetSource功能定义如下：

```
public interface TargetSource extends TargetClassAware {

	/**
	 * 返回当前目标源的目标类型。
	 * 可以返回null值，如：EmptyTargetSource（未知类会使用这个目标源）
	 */
	@Override
	Class<?> getTargetClass();


	/**
	 * 当前目标源是否是静态的。
	 * 如果为false，则每次方法调用结束后会调用releaseTarget()释放目标对象.
	 * 如果为true，则目标对象不可变，也就没必要释放了。
	 * @return
	 */
	boolean isStatic();

	/**
	 * 获取一个目标对象。
	 * 在每次MethodInvocation方法调用执行之前获取。
	 * @return
	 * @throws Exception
	 */
	Object getTarget() throws Exception;

	/**
	 * 释放指定的目标对象。
	 * @param target
	 * @throws Exception
	 */
	void releaseTarget(Object target) throws Exception;

}
```

TargetSource组件体系图如下：

![img](http://static.oschina.net/uploads/space/2016/0607/122929_GFIk_2624635.png)

TargetSource包含4个简单实现和3大类实现。

​    四个简单实现包括：

1. EmptyTargetSource：
   静态目标源，当不存在target目标对象，或者甚至连targetClass目标类都不存在（或未知）时，使用此类实例。
2. HotSwappableTargetSource：
   动态目标源，支持热替换的目标源，支持spring应用运行时替换目标对象。
3. JndiObjectTargetSource：
   spring对JNDI管理bean的支持，static属性可配置。
4. SingletonTargetSource：
   静态目标源，单例目标源。Spring的AOP框架默认为受IoC容器管理的bean创建此目标源。换句话说，SingletonTargetSource、proxy与目标bean三者的声明周期均相同。如果bean被配置为prototype，则spring会在每次getBean时创建新的SingletonTargetSource实例。

​      三大类实现包括：

1. AbstractBeanFactoryBasedTargetSource：
   此类目标源基于IoC容器实现，也就是说target目标对象可以通过beanName从容器中获取。此类又扩展出：
   （1）SimpleBeanTargetSource：简单实现，直接调用getBean从容器获取目标对象；
   （2）LazyInitTargetSource：延迟初始化目标源，子类可重写postProcessTargetObject方法后置处理目标对象；
   （3）AbstractPrototypeBasedTargetSource：原型bean目标源，此抽象类可确保beanName对应的bean的scope属性为prototype。其子类做了简单原型、池化原型、线程隔离原型这3种实现。
2. AbstractRefreshableTargetSource：
   可刷新的目标源。此类实现可根据配置的刷新延迟时间，在每次获取目标对象时自动刷新目标对象。
3. AbstractLazyCreationTargetSource：
   此类实现在调用getTarget()获取时才创建目标对象。

​        总结一下，本文详解了什么是TargetSource，它的作用是什么，以及该接口的各种实现。最后提醒一下：SpringAOP的自动代理（*AutoProxyCreator），只会为受SpringIoC容器管理的bean创建SingletonTargetSource。其他实现类均在手动代理时按需使用。