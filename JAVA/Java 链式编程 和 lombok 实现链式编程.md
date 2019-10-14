# Java 链式编程 和 lombok 实现链式编程

### 文章目录

- - [1、链式编程定义](https://blog.csdn.net/xiaojin21cen/article/details/83478929#1_1)
  - [2、jdk中 StringBuffer 源码：](https://blog.csdn.net/xiaojin21cen/article/details/83478929#2jdk_StringBuffer__15)
  - [3、按照上面的方法写一个例子:](https://blog.csdn.net/xiaojin21cen/article/details/83478929#3_25)
  - [4、 `lombok` 链式编程](https://blog.csdn.net/xiaojin21cen/article/details/83478929#4_lombok__64)
  - [5、`lombok ` 实现静态的链式编程](https://blog.csdn.net/xiaojin21cen/article/details/83478929#5lombok___83)
  - [6、自定义 builder模式的链式Bean](https://blog.csdn.net/xiaojin21cen/article/details/83478929#6_builderBean_115)
  - [7、`lombok ` 实现 builder模式的链式bean](https://blog.csdn.net/xiaojin21cen/article/details/83478929#7lombok___builderbean_171)

## 1、链式编程定义

> 链式编程的原理就是返回一个this对象，就是返回本身，达到链式效果。

我们经常用的 `StringBuffer` 就是 实现了链式的写法。

```
StringBuffer builder = new StringBuffer();
builder.append("blake").append("bob").append("alice").append("linese").append("eve");
12
```

是不是很方便呢!

怎么实现呢，其实就是在设置的 **返回当前的对象** 。

## 2、jdk中 StringBuffer 源码：

```
@Override
public StringBuilder append(String str) {
    super.append(str);
    return this;
}
12345
```

## 3、按照上面的方法写一个例子:

```
public class StudentBean {
	private String name;
	private int age;

	public String getName() {
		return name;
	}

	public StudentBean setName(String name) {
		this.name = name;
		return this;
	}

	public int getAge() {
		return age;
	}

	public StudentBean setAge(int age) {
		this.age = age;
		return this;
	}
}

1234567891011121314151617181920212223
```

测试：

```
public class Main {

	public static void main(String[] args) {
		
		StudentBean studentBean = new StudentBean().setAge(22).setName("ly");
		System.out.println(studentBean.getAge());
		System.out.println(studentBean.getName());
	}
}
123456789
```

## 4、 `lombok` 链式编程

其实，lombok 已经提供该 style，我们把这个bean 改成 lombok 实现只需要加上一个 `@Accessors(chain = true)` 即可。

```
@Accessors(chain = true)
@Getter
@Setter
public class StudentBean {

	private String name;	
	private int age;
	
}
123456789
```

这样代码简洁多了 ，而且实现了链式编程。

测试代码与上面的代码完全一样。

## 5、`lombok` 实现静态的链式编程

写StudentBean这个bean的时候，会有一些必输字段，比如StudentBean中的name字段，一般处理的方式是将name字段包装成一个构造方法，只有传入name这样的构造方法，才能创建一个StudentBean对象。

使用 `lombok` 将更改成如下写法： `@RequiredArgsConstructor` 和 `@NonNull`

```
@Accessors(chain = true)
@Getter
@Setter
@RequiredArgsConstructor(staticName = "of")
public class StudentBean {

	@NonNull
	private String name;
	
	private int age;
}
1234567891011
```

测试方法：

```
public class Main {	
	public static void main(String[] args) {		
		StudentBean studentBean = StudentBean.of("zhangsan").setAge(22);
		System.out.println(studentBean.getAge());
		System.out.println(studentBean.getName());
	}
}
1234567
```

这样不仅实现了链式编程，还实现了静态创建。

## 6、自定义 builder模式的链式Bean

build模式实现原理为在bean里面创建一个 **静态builder方法** 和一个 **静态内部Builder类** ，通过调用静态builder方法来创建 Builder类，然后通过 builder类 中的 build方法直接创建一个Bean，具体如下：

```
public class StudentBean {
	private String name;
	
	private int age;
 
	public String getName() {
		return name;
	} 
	public void setName(String name) {
		this.name = name;
	} 
	public int getAge() {
		return age;
	} 
	public void setAge(int age) {
		this.age = age;
	}
		
	public static Builder builder() {
		return new Builder();
	}
	
	public static class Builder{
		private String name;
		
		private int age;
 
		public Builder name(String name) {
			this.name = name;
			return this;
		}
 
		public Builder age(int age) {
			this.age = age;
			return this;
		}
		
		public StudentBean build() {
			StudentBean studentBean = new StudentBean();
			studentBean.setName(name);
			studentBean.setAge(age);
			return studentBean;
		}
	}
}
123456789101112131415161718192021222324252627282930313233343536373839404142434445
```

测试方法：

```
StudentBean studentBean = StudentBean.builder().name("zhangsan").age(11).build();
1
```

## 7、`lombok` 实现 builder模式的链式bean

这样就实现了一个builder模式的链式bean。其实用lombok就一个注解的事情，调用与上面同样

```
@Builder
public class StudentBean {
	private String name;
	
	private int age;
}
```