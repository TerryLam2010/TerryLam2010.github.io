# 【转】深入Spring Boot：那些注入不了的Spring占位符（${}表达式）

出自：http://hengyunabc.github.io/spring-placeholder-inject-failed-cases/

## Spring里的占位符

spring里的占位符通常表现的形式是：

```
<bean id="dataSource" destroy-method="close" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="url" value="${jdbc.url}"/>
</bean>
```

或者

```
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {
    @Value("${jdbc.url}")
    private String url;
}
```

Spring应用在有时会出现占位符配置没有注入，原因可能是多样的。

本文介绍两种比较复杂的情况。

## 占位符是在Spring生命周期的什么时候处理的

Spirng在生命周期里关于Bean的处理大概可以分为下面几步：

1. 加载Bean定义（从xml或者从`@Import`等）
2. 处理`BeanFactoryPostProcessor`
3. 实例化Bean
4. 处理Bean的property注入
5. 处理`BeanPostProcessor`

![spring-context-callback](http://hengyunabc.github.io/img/spring_context_callback.png)

当然这只是比较理想的状态，实际上因为Spring Context在构造时，也需要创建很多内部的Bean，应用在接口实现里也会做自己的各种逻辑，整个流程会非常复杂。

那么占位符（${}表达式）是在什么时候被处理的？

- 实际上是在`org.springframework.context.support.PropertySourcesPlaceholderConfigurer`里处理的，它会访问了每一个bean的BeanDefinition，然后做占位符的处理
- `PropertySourcesPlaceholderConfigurer`实现了`BeanFactoryPostProcessor`接口
- `PropertySourcesPlaceholderConfigurer`的 order是`Ordered.LOWEST_PRECEDENCE`，也就是最低优先级的

结合上面的Spring的生命周期，如果Bean的创建和使用在`PropertySourcesPlaceholderConfigurer`之前，那么就有可能出现占位符没有被处理的情况。

## 例子1：Mybatis 的 MapperScannerConfigurer引起的占位符没有处理

例子代码：<https://github.com/hengyunabc/spring-boot-inside/tree/master/demo-mybatis-placeholder>

- 首先应用自己在代码里创建了一个`DataSource`，其中`${db.user}`是希望从`application.properties`里注入的。代码在运行时会打印出`user`的实际值。

  ```
  @Configuration
  public class MyDataSourceConfig {
  	@Bean(name = "dataSource1")
  	public DataSource dataSource1(@Value("${db.user}") String user) {
  		System.err.println("user: " + user);
  		JdbcDataSource ds = new JdbcDataSource();
  		ds.setURL("jdbc:h2:˜/test");
  		ds.setUser(user);
  		return ds;
  	}
  }
  ```

- 然后应用用代码的方式来初始化mybatis相关的配置，依赖上面创建的`DataSource`对象

  ```
  @Configuration
  public class MybatisConfig1 {
  
  	@Bean(name = "sqlSessionFactory1")
  	public SqlSessionFactory sqlSessionFactory1(DataSource dataSource1) throws Exception {
  		SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
  		org.apache.ibatis.session.Configuration ibatisConfiguration = new org.apache.ibatis.session.Configuration();
  		sqlSessionFactoryBean.setConfiguration(ibatisConfiguration);
  
  		sqlSessionFactoryBean.setDataSource(dataSource1);
  		sqlSessionFactoryBean.setTypeAliasesPackage("sample.mybatis.domain");
  		return sqlSessionFactoryBean.getObject();
  	}
  
  	@Bean
  	MapperScannerConfigurer mapperScannerConfigurer(SqlSessionFactory sqlSessionFactory1) {
  		MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
  		mapperScannerConfigurer.setSqlSessionFactoryBeanName("sqlSessionFactory1");
  		mapperScannerConfigurer.setBasePackage("sample.mybatis.mapper");
  		return mapperScannerConfigurer;
  	}
  }
  ```

当代码运行时，输出结果是：

```
user: ${db.user}
```

为什么会`user`这个变量没有被注入？

分析下Bean定义，可以发现`MapperScannerConfigurer`它实现了`BeanDefinitionRegistryPostProcessor`。这个接口在是Spring扫描Bean定义时会回调的，远早于`BeanFactoryPostProcessor`。

所以原因是：

- `MapperScannerConfigurer`它实现了`BeanDefinitionRegistryPostProcessor`，所以它会Spring的早期会被创建
- 从bean的依赖关系来看，mapperScannerConfigurer依赖了sqlSessionFactory1，sqlSessionFactory1依赖了dataSource1
- `MyDataSourceConfig`里的`dataSource1`被提前初始化，没有经过`PropertySourcesPlaceholderConfigurer`的处理，所以`@Value("${db.user}") String user` 里的占位符没有被处理

要解决这个问题，可以在代码里，显式来处理占位符：

```
environment.resolvePlaceholders("${db.user}")
```

## 例子2：Spring boot自身实现问题，导致Bean被提前初始化

例子代码：<https://github.com/hengyunabc/spring-boot-inside/tree/master/demo-ConditionalOnBean-placeholder>

Spring Boot里提供了`@ConditionalOnBean`，这个方便用户在不同条件下来创建bean。里面提供了判断是否存在bean上有某个注解的功能。

```
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnBeanCondition.class)
public @interface ConditionalOnBean {
	/**
	 * The annotation type decorating a bean that should be checked. The condition matches
	 * when any of the annotations specified is defined on a bean in the
	 * {@link ApplicationContext}.
	 * @return the class-level annotation types to check
	 */
	Class<? extends Annotation>[] annotation() default {};
```

比如用户自己定义了一个Annotation：

```
@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
}
```

然后用下面的写法来创建abc这个bean，意思是当用户显式使用了`@MyAnnotation`（比如放在main class上），才会创建这个bean。

```
@Configuration
public class MyAutoConfiguration {
	@Bean
	// if comment this line, it will be fine.
	@ConditionalOnBean(annotation = { MyAnnotation.class })
	public String abc() {
		return "abc";
	}
}
```

这个功能很好，但是在spring boot 1.4.5 版本之前都有问题，会导致FactoryBean提前初始化。

在例子里，通过xml创建了`javaVersion`这个bean，想获取到java的版本号。这里使用的是spring提供的一个调用static函数创建bean的技巧。

```
<bean id="sysProps" class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
  <property name="targetClass" value="java.lang.System" />
  <property name="targetMethod" value="getProperties" />
</bean>

<bean id="javaVersion" class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
  <property name="targetObject" ref="sysProps" />
  <property name="targetMethod" value="getProperty" />
  <property name="arguments" value="${java.version.key}" />
</bean>
```

我们在代码里获取到这个`javaVersion`，然后打印出来：

```
@SpringBootApplication
@ImportResource("classpath:/demo.xml")
public class DemoApplication {

	public static void main(String[] args) {
		ConfigurableApplicationContext context = SpringApplication.run(DemoApplication.class, args);
		System.err.println(context.getBean("javaVersion"));
	}
}
```

在实际运行时，发现javaVersion的值是null。

这个其实是spring boot的锅，要搞清楚这个问题，先要看`@ConditionalOnBean`的实现。

- `@ConditionalOnBean`实际上是在ConfigurationClassPostProcessor里被处理的，它实现了`BeanDefinitionRegistryPostProcessor`

- `BeanDefinitionRegistryPostProcessor`是在spring早期被处理的

- `@ConditionalOnBean`的具体处理代码在`org.springframework.boot.autoconfigure.condition.OnBeanCondition`里

- `OnBeanCondition`在获取bean的Annotation时，调用了`beanFactory.getBeanNamesForAnnotation`

  ```
  private String[] getBeanNamesForAnnotation(
      ConfigurableListableBeanFactory beanFactory, String type,
      ClassLoader classLoader, boolean considerHierarchy) throws LinkageError {
    String[] result = NO_BEANS;
    try {
      @SuppressWarnings("unchecked")
      Class<? extends Annotation> typeClass = (Class<? extends Annotation>) ClassUtils
          .forName(type, classLoader);
      result = beanFactory.getBeanNamesForAnnotation(typeClass);
  ```

- `beanFactory.getBeanNamesForAnnotation` 会导致`FactoryBean`提前初始化，创建出`javaVersion`里，传入的`${java.version.key}`没有被处理，值为null。

- spring boot 1.4.5 修复了这个问题：<https://github.com/spring-projects/spring-boot/issues/8269>

## 实现spring boot starter要注意不能导致bean提前初始化

用户在实现spring boot starter时，通常会实现Spring的一些接口，比如`BeanFactoryPostProcessor`接口，在处理时，要注意不能调用类似`beanFactory.getBeansOfType`，`beanFactory.getBeanNamesForAnnotation` 这些函数，因为会导致一些bean提前初始化。

而上面有提到`PropertySourcesPlaceholderConfigurer`的order是最低优先级的，所以用户自己实现的`BeanFactoryPostProcessor`接口在被回调时很有可能占位符还没有被处理。

对于用户自己定义的`@ConfigurationProperties`对象的注入，可以用类似下面的代码：

```
@ConfigurationProperties(prefix = "spring.my")
public class MyProperties {
	String key;
}
```

```
public static MyProperties buildMyProperties(ConfigurableEnvironment environment) {
  MyProperties myProperties = new MyProperties();

  if (environment != null) {
    MutablePropertySources propertySources = environment.getPropertySources();
    new RelaxedDataBinder(myProperties, "spring.my").bind(new PropertySourcesPropertyValues(propertySources));
  }

  return myProperties;
}
```

https://www.oschina.net/translate/spring-boot-2-0-migration-guide?cmp

boot 2.0 已经去除bind包。

## 总结

- 占位符（${}表达式）是在`PropertySourcesPlaceholderConfigurer`里处理的，也就是`BeanFactoryPostProcessor`接口
- spring的生命周期是比较复杂的事情，在实现了一些早期的接口时要小心，不能导致spring bean提前初始化
- 在早期的接口实现里，如果想要处理占位符，可以利用spring自身的api，比如 `environment.resolvePlaceholders("${db.user}")`