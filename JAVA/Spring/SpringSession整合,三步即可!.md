# SpringSession整合,三步即可!

### 其实整合十分的简单，首先上依赖

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>		
<!--spring session 与redis应用基本环境配置,需要开启redis后才可以使用，不然启动Spring boot会报错 -->
<dependency>
	<groupId>org.springframework.session</groupId>
	<artifactId>spring-session-data-redis</artifactId>
</dependency>
<!--引入该依赖是因为spring-session-data-redis 使用到了security的类，但是排除掉config 就可以不使用security了 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-config</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```

### 然后第二步：

你的application.yml 应该这样写

```
server:
  port: 8088
spring:
  devtools:
  redis:
 #   database: 7
 #   host: 127.0.0.1
 #   port: 6379
    timeout: 60s  # 数据库连接超时时间，2.0 中该参数的类型为Duration，这里在配置的时候需要指明单位
    password: aaaaaa
    cluster:
      nodes: 192.168.53.53:8010,192.168.53.53:8011,192.168.53.53:8012,192.168.53.53:8013,192.168.53.53:8014,192.168.53.53:8015
    lettuce:
      pool:
        max-active:  100 # 连接池最大连接数（使用负值表示没有限制）
        max-idle: 100 # 连接池中的最大空闲连接
        min-idle: 50 # 连接池中的最小空闲连接
        max-wait: 6000 # 连接池最大阻塞等待时间（使用负值表示没有限制）
  session:
    store-type: redis
    timeout: 1800
  autoconfigure.exclude: org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration #排除
```

### 完事的第三步：

当然，不止这么简单 ，我希望你是配好redis才到这一步哟

```
@SpringBootApplication
@ComponentScan({"cn.xxx.xxx","cn.xxx.xxxx.xxxx"})
@EnableRedisHttpSession   // 启动共享session
public class HWBAdminApplication extends SpringBootServletInitializer {

	public static void main(String[] args) {
		SpringApplication.run(HWBAdminApplication.class, args);
	}

}
```

完工！

使用了shiro也不怕，就这样做就行了。



最后附上我的redis配置(这个看看就好。。根据自己需要改)

```
@Configuration
@EnableCaching // 开启缓存支持
public class RedisConfig extends CachingConfigurerSupport {

	@Resource
	private LettuceConnectionFactory lettuceConnectionFactory;

	@Bean
	public KeyGenerator keyGenerator() {
		return new KeyGenerator() {
			@Override
			public Object generate(Object target, Method method, Object... params) {
				StringBuffer sb = new StringBuffer();
				sb.append(target.getClass().getName());
				sb.append(method.getName());
				for (Object obj : params) {
					//由于参数可能不同, 缓存的key也需要不一样
					sb.append(obj.toString());
				}
				return sb.toString();
			}
		};
	}

	// 缓存管理器
	@Bean
	public CacheManager cacheManager() {

		// 设置序列化
		RedisSerializer<String> stringSerializer = new StringRedisSerializer();
		GenericJackson2JsonRedisSerializer jackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

		RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
				.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(stringSerializer))
				// value序列化方式
				.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
				.disableCachingNullValues()
				// 缓存过期时间
				.entryTtl(Duration.ofMinutes(5));

		EnhanceRedisManager.EnhanceRedisManagerBuilder builder = EnhanceRedisManager.EnhanceRedisManagerBuilder
				.fromConnectionFactory(lettuceConnectionFactory)
				.cacheDefaults(config)
				.transactionAware();
		@SuppressWarnings("serial")
		Set<String> cacheNames = new HashSet<String>() {
			{
				add("codeNameCache");
			}
		};
		RedisCacheWriter cacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(lettuceConnectionFactory);
		builder.initialCacheNames(cacheNames);
		TedisCacheManager t = new TedisCacheManager(cacheWriter,config);
		return t;
	}

	/**
	 * RedisTemplate配置
	 */
	@Bean
	public RedisTemplate<String, Object> redisTemplate(LettuceConnectionFactory lettuceConnectionFactory) {
		// 设置序列化
		Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(
				Object.class);
		ObjectMapper om = new ObjectMapper();
		om.setVisibility(PropertyAccessor.ALL, Visibility.ANY);
		om.enableDefaultTyping(DefaultTyping.NON_FINAL);
		jackson2JsonRedisSerializer.setObjectMapper(om);
		// 配置redisTemplate
		RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
		redisTemplate.setConnectionFactory(lettuceConnectionFactory);
		RedisSerializer<?> stringSerializer = new StringRedisSerializer();
		redisTemplate.setKeySerializer(stringSerializer);// key序列化
		redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);// value序列化
		redisTemplate.setHashKeySerializer(stringSerializer);// Hash key序列化
		redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);// Hash value序列化
		redisTemplate.afterPropertiesSet();
		return redisTemplate;
	}

	@Bean
	public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
		StringRedisTemplate template = new StringRedisTemplate();
		template.setConnectionFactory(redisConnectionFactory);
		return template;
	}

}
```

