# SpringBoot 增加Druid数据源监控

使用spring boot中配置druid的时候遇到的几个坑

首先spring boot版本 2.1.0

```
<parent>

		<groupId>org.springframework.boot</groupId>

		<artifactId>spring-boot-starter-parent</artifactId>

		<version>2.1.0.RELEASE</version>

		<relativePath/> <!-- lookup parent from repository -->

	</parent>

druid版本  1.1.10

<!-- https://mvnrepository.com/artifact/com.alibaba/druid-spring-boot-starter druid 数据源 -->

        <dependency>

            <groupId>com.alibaba</groupId>

            <artifactId>druid-spring-boot-starter</artifactId>

            <version>1.1.10</version>

        </dependency>

```


版本一定要一致

yml文件配置

```
spring: 

  datasource:

    url: jdbc:mysql://你自己的url

    username: 数据库账号

    password: 数据库密码

    type: com.alibaba.druid.pool.DruidDataSource

    druid:

      # 下面为连接池的补充设置，应用到上面所有数据源中

      # 初始化大小，最小，最大

      initial-size: 5

      min-idle: 5

      max-active: 20

      # 配置获取连接等待超时的时间

      max-wait: 60000

      # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒

      time-between-eviction-runs-millis: 60000

      # 配置一个连接在池中最小生存的时间，单位是毫秒

      min-evictable-idle-time-millis: 300000

      validation-query: SELECT 1 FROM DUAL

      test-while-idle: true

      test-on-borrow: false

      test-on-return: false

      # 打开PSCache，并且指定每个连接上PSCache的大小

      pool-prepared-statements: true

      #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙

      max-pool-prepared-statement-per-connection-size: 20

      filters: stat,wall

      use-global-data-source-stat: true

      # 通过connectProperties属性来打开mergeSql功能；慢SQL记录

      connect-properties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000

      # 配置监控服务器

      stat-view-servlet:

        login-username: admin

        login-password: 123456

        reset-enable: false

        url-pattern: /druid/*

        # 添加IP白名单

        #allow:

        # 添加IP黑名单，当白名单和黑名单重复时，黑名单优先级更高

        #deny:

      web-stat-filter:

        # 添加过滤规则

        url-pattern: /*

        # 忽略过滤格式

        exclusions: ".js,.gif,.jpg,.png,.css,.ico,/druid/*"

```


坑一：

    一开始根本不知道怎么玩，查找到的文章中的yml文件千篇一律，因为版本的原因，yml文件有的时候不能一致和生效

特别是在 配置

filters: stat,wall


的时候，总是报错。网上给的参数的话这里都是三个参数，还有一个log4j

在他们的配置中是不会报错的，一般是spring boot 1.5中可以使用，但是到了2.0之后加上log4j就会报错，2.0之后2.1记得去掉。

坑二：

    看了很多，都是使用了还要写一个配置类然后用@Configuration注解交给spring管理创建 stat-view-servlet（监控服务器）和web-stat-filter（拦截器）

然后我就跟着文章上面的去配了，在配置类中配置url-mapping啦，账号，用户名啦，然后拦截规则和放行规则啦等等

然后打开 localhost:8080/druid 很开心，我擦成功了

然后有一天我把spring boot版本换到了2.1问题就来了， 项目启动报错，意思是startfilter重复定义 就是说在yml中已经默认给你配了startfilter然后你在配置类中又用@Bean注入了一个startfilter 在spring boot 2.0中是可以启动的，但是2.1不行，这个问题搞了我好久。然后发现可以完全不用配置类，全部都用yml参数来注入。参照我上面的配置吧，注意boot和druid的版本哦！



 

把这个去掉就行了。

 只要版本是我写的这两个都是蛮新的，照搬就行了。到此纯yml配置druid完成。

 

对了最后提一下

 web-stat-filter:
        # 添加过滤规则
        url-pattern: /*
        # 忽略过滤格式
        exclusions: "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*"
这里的exclusions忽略过滤格式 记得一定要加 " " 不然也是错的