# 

使用spring-boot 2.0.3

一、代码整合

二、使用的注意事项

需要在pom文件添加。 不然idea不加载sql文件 ，freemarker就会报找不到文件的异常

```
<resources>
	<resource>
		<directory>src/main/java</directory>
		<includes>
			<include>**/*.sql</include>
		</includes>
	</resource>
</resources>
```

