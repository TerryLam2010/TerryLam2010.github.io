JDK8必知必会

- - - [1. Java 8 新特性简介](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#1java8)
    - \2. Lambda 表达式
      - [2.1 什么是 Lambda 表达式](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#21lambda)
      - [2.2 内部匿名类](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#22)
      - [2.3 Lambda 表达式语法](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#23lambda)
      - [2.4 方法引用](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#24)
    - [3. 函数式接口](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#3)
    - \4. Stream API
      - [4.1 创建 Stream 流](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#41stream)
      - [4.2 Stream 的中间操作](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#42stream)
      - [4.3 Stream 的终止操作](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#43stream)
      - [4.4 Stream 练习](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#44stream)
    - [5. 并行和串行](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#5)
    - [6. 用 Optional 取代空指针](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#6optional)
    - [7. Java 日期操作](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#7java)
    - [8. Java 8 Nashorn JavaScript](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#8java8nashornjavascript)
    - [9. Java 8 Base64 编码解码](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#9java8base64)
    - [10. Java 8 基础面试题、知识回顾](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#10java8)
    - [11. Java 8 编程面试题、知识回顾](https://gitbook.cn/books/5d9f2ae08d85b37bbb712f12/index.html#11java8)

### 1. Java 8 新特性简介

Java 8（又称为 JDK 1.8）是 Java 语言开发的一个主要版本。 Oracle 公司于 2014 年 3 月 18 日发布 Java 8，它支持函数式编程、新的 JavaScript 引擎、新的日期 API、新的Stream API 等。

Java 8 有如下优势：

- 速度更快
- 减少代码的 Lambda 表达式
- 强大的 Stream API
- 便于并行
- 最大化减少空指针异常：Optional
- Nashorn，JavaScript 引擎，允许我们在 JVM 上运行特定的 JavaScript 应用

> 本章实例均在 JDK 1.8 环境上实现，建议使用前，输入下命令 `java -version` 查看当前 JDK 的版本。

### 2. Lambda 表达式

#### 2.1 什么是 Lambda 表达式

Lambda 表达式可以理解为简洁地表示可传递的匿名函数的一种方式：它没有名称，但它有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。使用 Lambda 表达式的代码会更加简洁，可读性更强。

Lambda 表达式由三个部分组成：

- 第一部分为一个括号内用逗号分隔的形式参数，参数是函数式接口里面方法的参数
- 第二部分为一个箭头符号（ Lambda ）：->
- 第三部分为方法体，可以是表达式和代码块。语法如下：

```
() -> expression
或
(parameters) -> expression
或
(parameters) -> { statements; }
```

![image](https://images.gitbook.cn/644eb110-eb5f-11e9-a0db-c9a5fffa48ce)

以下是 Lambda 表达式的重要特征：

- 可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。
- 可选的参数圆括号：一个参数无需定义圆括号，但无参和多个参数需要定义圆括号。
- 可选的大括号：如果主体包含了一个语句，就不需要使用大括号。
- 可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个参数。

#### 2.2 内部匿名类

在 Java 8 之前设计匿名内部类的目的，就是为了方便 Java 程序员将代码作为数据传递。不过，匿名内部类还是不够简便。接下来通过两个例子感受下函数式编程对内部匿名类的简洁写法。

**范例 1：无参内部匿名类**

```
    @Test
    public void test01(){
        System.out.println("*************  JDK 1.7 *****************");
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("学习啦！学习啦！");
            }
        };
        runnable.run();

        System.out.println("*************  JDK 1.8 Lambda *****************");
        Runnable runnable2 = () -> {
            System.out.println("学习啦！学习啦！！");
        };
        runnable2.run();
    }
```

运行结果：

```
*************  JDK 1.7 *****************
学习啦！学习啦！
************  JDK 1.8 Lambda ************
学习啦！学习啦！！
```

**范例 2：有参数内部匿名类**

```
    @Test
    public void test02(){
        System.out.println("*************  JDK 1.7 比较函数   *****************");
        Comparator<Integer> comparator = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return Integer.compare(o1, o2);
            }
        };
        int compare = comparator.compare(10, 20);
        System.out.println(compare);

        System.out.println(" ************** JDK 1.8 比较函数 Lambda  ****************");
        // Lambda 表达式
        Comparator<Integer>  comp1 =  (o1,o2) ->Integer.compare(o1,o2);
        int comparator2 = comp1.compare(10,20);
        System.out.println(comparator2);

        System.out.println("************ JDK 1.8 比较函数 方法引用  *******************");
        // 方法引用
        Comparator<Integer>  comp2=  Integer::compare;
        int compare1 = comp2.compare(10, 20);
        System.out.println(compare1);
    }
```

运行结果：

```
****************  JDK 1.7 比较函数   ********************
-1
 ************** JDK 1.8 比较函数 Lambda  ****************
-1
************** JDK 1.8 比较函数 方法引用  ****************
-1
```

通过 JDK 1.7 和 JDK 1.8 的匿名实现类，大家应该看出来在 JDK 1.8 更加紧凑、简洁，大家对上面的代码是否充满了困惑和不解，接下来我们逐步学习。

> 此小节代码：com.mtcarpenter.lambda.AnonymousTest.java

#### 2.3 Lambda 表达式语法

**Lambda 表达式的语法使用**

\1. 没有参数。没有返回值。

```
    @Test
    public void test01(){
        System.out.println("********* jdk 1.7 **********");
        Runnable r1 =  new Runnable(){
            @Override
            public void run() {
                System.out.println("lambda 学习");
            }
        };
        r1.run();
        System.out.println("********* jdk 1.8 **********");
        Runnable r2 = ()->  System.out.println("lambda 学习");
        r2.run();
    }
```

运行结果：

```
********* jdk 1.7 **********
lambda 学习
********* jdk 1.8 **********
lambda 学习
```

\2. 类型声明，因为可由编译器推断得出，成为“类型推断”。

```
    @Test
    public void test02() {
        System.out.println("************ 类型推断 *************");
        Consumer<String> con = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };


        //  Consumer  类型 决定 new Consumer 类型
       /* Consumer<String> con = new Consumer<Integer>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };
        con.accept("传递参数!");
        */

        // accept 方法中类型 由 new Consumer 决定
        /*
       new Consumer<String>() {
            @Override
            public void accept(Integer s) {
                System.out.println(s);
            }
        };
        */

        System.out.println("************ lambda *************");
        Consumer<String> con1 = (String s) -> {
            System.out.println(s);
        };
        con1.accept("传递参数！");

        Consumer<String> con2 = (s) -> {
            System.out.println(s);
        };
        con2.accept("传递参数！");
    }
```

运行结果：

```
************ 类型推断 *************
************ lambda *************
传递参数！
传递参数！
```

在上面的代码中我注解的内容大家可以打开，你会发现报错，第一段注释（/**/）的内容中， Consumer 类型 决定了 new Consumer 类型。第二段注释内容中， `new Consumer<Integer>` 中的类型必须跟接收的对象的类型一样 ，而 `accept` 的类型必须跟 `new Consumer` 一致。类型必须保持一致才能运行，因此 `(String s)` 可以省略为 `(s)`，编译器直接根据对象的指定类型推断，lambda 表达式中的类型推断做了增强。类型推断在之前 JDK 1.7 及之前应该我们在日常中数组初始值经常使用，代码如下：

```
    @Test
    public void test03(){
        int[] arr1 = new int[]{1,2,3};
        // 数组类型自动推断
        int[] arr2= {1,2,3};

        // 集合类型自动推断 
        List<String>  list1 = new ArrayList<String>();
        List<String>  list2 = new ArrayList<>();

        HashMap<String, String> hashMap1 = new HashMap<String, String>();
        HashMap<String, String> hashMap2 = new HashMap<>();
    }
```

不用明确声明泛型类型，编译器就可以自己推断出来。

\3. 一个参数，一条语句，其都可以省略。

```
    @Test
     public void test04(){
        System.out.println("*************************");
        // 复制 test02 最后中类型推断语句
        Consumer<String> con1 = (s) ->{
            System.out.println(s);
        };
        con1.accept("传递参数！");

        System.out.println("*************************");
        // (s) 形参中只有一个参数，() 括号直接省略不写 
        Consumer<String> con2 =  s ->{
            System.out.println(s);
        };
        con2.accept("传递参数！");
        System.out.println("*************************");
        //  函数体只有一条语句也可以省略，{ System.out.println(s);} --->System.out.println(s);
        Consumer<String> con3 =  s -> System.out.println(s);
        con3.accept("传递参数！");
     }
```

运行结果：

```
*************************
传递参数！
*************************
传递参数！
*************************
传递参数！
```

\4. 两个或以上的参数，多条执行语句，并且可以有返回值。

```
    @Test
    public void test05 (){

        System.out.println("********** jdk 1.7 *************");
        List<Integer> list = Arrays.asList(15, 5, 35, 25, 45);
        Collections.sort(list, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return Integer.compare(o1,o2);
            }
        });
        System.out.println("list = " + list);

        System.out.println("********** jdk 1.8 *************");

        List<Integer> list1 = Arrays.asList(15, 5, 35, 25, 45);
        // { return  o1-o2;} 函数体中只包含一条语句的时候 {} 花括号和 return 都可以省略
       // Collections.sort(list1,(o1, o2) -> { return  o1-o2;});
        Collections.sort(list1,(o1, o2) -> o1-o2);
        System.out.println("list1 = " + list1);
    }
```

运行结果：

```
********** jdk 1.7 *************
list = [5, 15, 25, 35, 45]
********** jdk 1.8 *************
list1 = [5, 15, 25, 35, 45]
```

在 Lambda 表达式中 从 `{ return o1-o2;}` 花括号中只有一条语句，就可以直接省略为 `(o1, o2) -> o1-o2` ，JDK 1.8 新特性大家会不会觉得还是很强呢？

\5. 以上表达式语法整合：

```
    @Test
    public void test06() {
        System.out.println("********** jdk 1.8 lambda 表达式语法整合 *************");
        // 类型声名 (String o1)
        Consumer<String> con1 = (String o1) -> {
            System.out.println(o1);
        };
        con1.accept("类型声名 (String o1)");
        // 不使用类型 (String o1) 》 ( o1 )
        Consumer<String> con2 = (o1) -> {
            System.out.println(o1);
        };
        con2.accept("不使用类型 (String o1) 》 ( o1 )");
        // 一个参数 可以取消 形式参数的括号  ( o1 ) 》 o1
        Consumer<String> con3 = o1 -> {
            System.out.println(o1);
        };
        con3.accept("一个参数 可以取消 形式参数的括号  ( o1 ) 》 o1");
        // 没有参数 不能省略形式参数的括号 ()
        Runnable r1 = () -> {
            System.out.println("没有参数 不能省略形式参数的括号");
        };
        // 函数体只有一条语句可以取消花括号 {}, { System.out.println(o1); } 》  System.out.println(o1)
        Consumer<String> con4 = o1 -> System.out.println(o1);
        con4.accept("函数体只有一条语句可以取消花括号 {}, { System.out.println(o1); } 》  System.out.println(o1)");
        // 多个参数 一条语句 形式参数的括号（） 不能省略
        Comparator<Integer> com1 = (o1, o2) -> {
            return o1.compareTo(o2);
        };
        System.out.println("o1  o2 比较大小 "+com1.compare(10, 20));
        // 多个参数 一条语句 可以省略 {} 和 return， {   return o1.compareTo(o2); } 》 o1.compareTo(o2)
        Comparator<Integer> com2 = (o1, o2) -> o1.compareTo(o2);
        System.out.println("o1  o2 比较大小 "+com2.compare(10, 20));
        // 多个参数 多条语句 括号（）和花括号 {} 都不能省略
        Comparator<Integer> com3 = (o1, o2) -> {
            System.out.println("o1 = "+o1 + "  o2 = "+o2);
            return o1.compareTo(o2);
        };
        System.out.println("o1  o2 比较大小 "+com3.compare(10,20));
    }
```

运行结果：

```
********** jdk 1.8 lambda 表达式语法整合 *************
类型声名 (String o1)
不使用类型 (String o1) 》 ( o1 )
一个参数 可以取消 形式参数的括号  ( o1 ) 》 o1
函数体只有一条语句可以取消花括号 {}, { System.out.println(o1); } 》  System.out.println(o1)
o1  o2 比较大小 -1
o1  o2 比较大小 -1
o1 = 10  o2 = 20
o1  o2 比较大小 -1
```

> 此小节代码：com.mtcarpenter.lambda.Lambda01Test.java

#### 2.4 方法引用

在前面的 Lambda 表达式语法大家觉得对比 JDK 1.7 是不是很简化了呢，那么方法引用会让大家觉得更加简洁，但是呢，如果大家前面的 Lambda 表达式没有理解，方法引用理解可能会比较困难，方法引用对 Lambda 又进一步优化升级，class::methodName 语法引用类或对象的方法被称为方法引用。方法引用使用一对冒号 `::`。

列举以下方法引用：

- 1.类名 :: 静态方法名
- 2.对象 :: 实例方法名
- 3.类名 :: 实例方法名
- 4.类名 ::new
- 5 数组 ::new

细心的你是否会问下怎么没有对象 :: 静态方法呢？其实我们在日常开发中，静态方法我们都是直接通过类名直接访问，无需通过对象访问。

**1. 类名 :: 静态方法名**

![images](https://images.gitbook.cn/05922db0-eb5e-11e9-8dea-797ca875d03d)

```
    @Test
    public void test01(){
        System.out.println("****************  JDK 1.7 比较函数   ********************");
        Comparator<Integer> comp1 = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return Integer.compare(o1, o2);
            }
        };
        int compare = comp1.compare(10, 20);
        System.out.println(compare);

        System.out.println("****************  JDK 1.8 比较函数   ********************");
        System.out.println("****************    Lambda 表达式    ********************");
        // Lambda 表达式
        Comparator<Integer> comp2 =  (o1, o2) ->Integer.compare(o1,o2);
        int comparator1 = comp2.compare(10,20);
        System.out.println(comparator1);

        System.out.println("****************    Lambda 方法引用  ********************");
        //  Lambda 方法引用
        Comparator<Integer>  comp3  =  Integer::compare;
        int compare1 = comp3.compare(10, 20);
        System.out.println(compare1);
    }
```

运行结果：

```
****************  JDK 1.7 比较函数   ********************
-1
****************  JDK 1.8 比较函数   ********************
****************    Lambda 表达式    ********************
-1
****************    Lambda 方法引用  ********************
-1
```

**2. 对象 :: 实例方法名**

![image](https://images.gitbook.cn/4a4eeb50-eb5e-11e9-a0db-c9a5fffa48ce)

```
    @Test
    public void test02(){
        Consumer<String> consumer = s -> System.out.println(s);
        consumer.accept("lambda 表达式");
        System.out.println("**************");
        // 方法引用 println 方法替换了 accept 方法
        PrintStream ps =  System.out;
        Consumer<String> consumer2 = ps::println;
        consumer2.accept("method reference");
    }
```

运行结果：

```
lambda 表达式
**************
method reference
```

**3. 类名 :: 实例方法名**

![image](https://images.gitbook.cn/5c5c0b20-eb5e-11e9-8dea-797ca875d03d)

```
    @Test
    public void test03(){
        Comparator<String> com1  = (o1,o2) -> o1.compareTo(o2);
        System.out.println(com1.compare("a","b"));

        System.out.println("**************");

        Comparator<String> com2 = String::compareTo;
        System.out.println(com2.compare("a","c"));
    }
```

运行结果：

```
-1
**************
-2
```

**4. 类名 ::new**

![image](https://images.gitbook.cn/6dfa6390-eb5e-11e9-821e-bba2257c3064)

```
class MathOperations {
    public MathOperations(int o1, int o2) {
        System.out.println("o1 + o2  =" +(o1 + o2));
    }
}


class MathOperations {
    public MathOperations(int o1, int o2) {
        System.out.println("o1 + o2  =" +(o1 + o2));
    }
}

//  范例 4 ： 类名::new
public class MethodReferenceTest4 {

    @Test
    public void tset01() {
        System.out.println("-------------------- lambda 表达式 ----------------------");
        BiConsumer<Integer, Integer> addtion1 = (a, b) -> new MathOperations(a, b);
        addtion1.accept(10, 20);

        System.out.println("--------------------- 方法引用     ---------------------");
        BiConsumer<Integer, Integer> addtion2 = MathOperations::new;
        addtion2.accept(20, 30);
    }
}
```

运行结果：

```
-------------------- 原始方法 ----------------------
o1 + o2  = 30
-------------------- lambda 表达式 ----------------------
o1 + o2  = 30
--------------------- 方法引用     ---------------------
o1 + o2  = 50
```

**5. 数组 ::new**

![image](https://images.gitbook.cn/81865b30-eb5e-11e9-821e-bba2257c3064)

```
    @Test
    public void test06(){
        System.out.println("--------------------   原始方法  ---------------------");
        Function<Integer,String[]> function1 = new Function<Integer, String[]>() {
            @Override
            public String[] apply(Integer length) {
                return new String[length];
            }
        };
         // 初始化数组长度为 5
        String[] str1 = function1.apply(5);
        System.out.println(Arrays.toString(str1));

        System.out.println("-------------------- lambda 表达式 ----------------------");
        Function<Integer,String[]> function2 = length -> new String[length];
        // 初始化数组长度为 6
        String[] str2 = function2.apply(6);
        System.out.println(Arrays.toString(str2));

        System.out.println("---------------------- 方法引用  ----------------------");
        Function<Integer,String[]> function3 =  String[]::new;
         // 初始化数组长度为 10  
        String[] str3 =function3.apply(10);
        System.out.println(Arrays.toString(str3));

    }
```

运行结果：

```
--------------------   原始方法  ---------------------
[null, null, null, null, null]
-------------------- lambda 表达式 ----------------------
[null, null, null, null, null, null]
---------------------- 方法引用  ----------------------
[null, null, null, null, null, null, null, null, null, null]
```

> 此小节代码：com.mtcarpenter.lambda.Lambda02Test.java

### 3. 函数式接口

Java 8 引入的一个核心概念是函数式接口（Functional Interfaces）,也称单例抽象方法接口 。

接口中只能包含一个抽象方法的接口。通过 @FunctionalInterface 在接口上面注解声明，表示该接口是一个函数式接口，该注解不是必须的，如果加上虚拟机会自动判断，是否是函数式接口。前面的实验我们使用最频繁的 Runnable 就是一个函数式接口，代码如下：

```
@FunctionalInterface
public interface Runnable {

    public abstract void run();
}
```

函数式接口也可以自定义，下面我们通过自定义来进一步了解。

```
@FunctionalInterface
public interface MyFunctionalInterface {

    public abstract void message();

    public abstract void type();
}
```

在上面的例子上，如果加上 @FunctionalInterface 编译器也会报错提示抽象方法不能有多个，注释其中一个方法即可，在实际开发中我建议大家接口中加上，尽量养成良好开发的习惯。

```
@FunctionalInterface
public interface MyFunctionalInterface {

    public abstract void message();
    //  在接口上加入注解 @FunctionalInterface 多个抽象方法编译器会提示错误 
    //public abstract void type();
}
```

**Java 8 中重要的函数接口**

| 函数式接口        | 参数类型 | 返回类型 | 描述                                                         | 抽象方法           |
| ----------------- | -------- | -------- | ------------------------------------------------------------ | ------------------ |
| Predicate<T>      | T        | boolean  | 断定型接口。接受一个类型 T 的参数，返回一个布尔值结果。      | boolean test(T t)  |
| Consumer<T>       | T        | void     | 消费型接口。接受一个类型 T 的参数，无返回操作。              | void accept(T t)   |
| Function<T,R>     | T        | R        | 函数型接口。接受一个类型 T 的参数，返回一个 R 类型的结果。   | R apply(T t)       |
| Supplier<T>       | none     | T        | 供给型接口。不接受任何参数，，返回一个 T 类型的结果。        | T get()            |
| UnaryOperator<T>  | T        | T        | 集成 Function 接口。接受一个类型 T 的参数，也返回 T 类型的结果。 | T apply(T t)       |
| BiFunction<T,U,R> | (T, U)   | T        | 接受两个不同类型的参数，返回一个其它类型的结果。             | R apply(T t, U u)  |
| BinaryOperator<T> | (T, T)   | T        | 代表了一个作用于于两个同类型操作符的操作，并且返回了操作符同类型的结果 | T apply(T t1,T t2) |

在 Java 8 还有很多函数接口在这里没有列举 ，它们都在 java.util.function 包下，接下来通过案例对上面的函数式接口，下面就列举 Predicate 和 Consumer 为范例。

**1. Predicate 断定型接口**

```
    /**
     * java 内置函数 Predicate
     * 断定型接口。接受一个类型 T 的参数，返回一个布尔值结果。
     */
    @Test
    public void test01(){
        // Predicate 来日 jdk 1.8 新接口
        Predicate<Integer> predicate1 = new Predicate<Integer>() {
            @Override
            public boolean test(Integer o1) {
                return o1 > 0;
            }
        };
        System.out.println(predicate1.test(12));

        System.out.println("************* lambda 表达式写法 *************");
        Predicate<Integer> predicate2 =   o1 -> {return  o1 > 0; } ;
        System.out.println(predicate2.test(-12));

        System.out.println("************* lambda 优化 ********************");
        Predicate<Integer> predicate3 =   o1 -> o1 > 0 ;
        System.out.println(predicate3.test(10));
    }
```

运行结果：

```
true
************* lambda 表达式写法 *************
false
************* lambda 优化 ********************
true
```

JDK 1.8 新引入的内置函数同样是可以像之前的 JDK 1.7 那种原始方式使用。

**2. Consumer 消费型接口**

```
    @Test
    public void test02(){
        // 在最开始的时候 我们就接触了这个函数
        System.out.println("************** 原始方式 ****************");
        Consumer<String> consumer1 = new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        };
        consumer1.accept("Consumer1 消费型接口");

        System.out.println("************* lambda 表达式 *************");
        Consumer<String> consumer2 = s ->  System.out.println(s);
        consumer2.accept("Consumer2 消费型接口");

        System.out.println("*************** 方法引用 *****************");
        Consumer<String> consumer3 =  System.out::println;
        consumer3.accept("Consumer3 消费型接口");
    }
```

运行结果：

```
************** 原始方式 ****************
Consumer1 消费型接口
************* lambda 表达式 *************
Consumer2 消费型接口
*************** 方法引用 *****************
Consumer3 消费型接口
```

> 此小节代码：com.mtcarpenter.functionalInterface.MyFunctionalInterfaceTest.java

### 4. Stream API

前面我们说了 JDK 1.8 重大改变的 lambda 表达式，Stream 同样是一个重磅级的功能。Stream API 用于处理对象的集合。

**Stream 是什么？**

Stream 它并不是一个容器，它只是对容器的功能进行了增强，添加了很多便利的操作，例如查找、过滤、分组、排序等一系列的操作。并且有串行、并行两种执行模式，并行模式充分的利用了多核处理器的优势，使用 fork/join 框架进行了任务拆分，同时提高了执行速度。简而言之，Stream 就是提供了一种高效且易于使用的处理数据的方式。

**Stream 的特点：**

- Stream 自己不会存储元素。
- Stream 的操作不会改变源对象。相反，他们会返回一个持有结果的新 Stream。
- Stream 操作是延迟执行的。它会等到需要结果的时候才执行。也就是执行终端操作的时候。

**创建 Stream 的三个步骤：**

- 第一：从集合数据源中，获取一个数据流。
- 第二：中间操作链，对数据进行处理。
- 第三：终止操作，用来执行中间操作链，返回结果。

图解：

![image](https://images.gitbook.cn/9e1a55d0-eb5e-11e9-a0db-c9a5fffa48ce)

#### 4.1 创建 Stream 流

\1. 通过集合的方式创建 Stream 流

```
    @Test
    public void test01(){
        List<String> list = Arrays.asList("a","b","c","d");
        // 创建串行流
        Stream<String> stream = list.stream();
        // 创建并行流
        Stream<String> parallelStream = list.parallelStream();
    }
```

创建的 Stream 需要通过终止条件才能显示，下面的代码为了大家运行代码能有显示的效果，我提前加入了 `forEach(System.out::println);`，终止并打印结果。

\2. 通过 Stream.of() 的方式创建 Stream 流

```
    @Test
    public void test02(){
        Stream<Integer> integerStream = Stream.of(1, 2, 3, 4, 5, 6);
        integerStream.forEach(System.out::println);
    }
```

运行结果：

```
1
2
3
4
5
6
```

\3. 通过 Arrays.stream() 创建 stream 流

```
    @Test
    public void test03(){
        int[] arr = new int[]{1,2,3,4,54,4};
        IntStream stream = Arrays.stream(arr);
        stream.forEach(System.out::println);
    }
```

运行结果：

```
1
2
3
4
54
4
```

\4. 通过 iterate（迭代） 和 generate（生成） 创建无限流

```
    @Test
    public void test05(){
        // 迭代
        Stream<Integer> iterate = Stream.iterate(0, t -> t + 2);
        iterate.forEach(System.out::println);
        //Stream.iterate(0,t->t+2).limit(10).forEach(System.out::println);
        // 生成
        Stream<Double>  generate= Stream.generate(Math::random).limit(10);
        generate.forEach(System.out::println);
    }
```

运行结果：

```
132196
132198
132200
132202
.....
```

> 此小节代码：com.mtcarpenter.stream.StreamApi01Test.java

#### 4.2 Stream 的中间操作

该操作会保持 Stream 处于中间状态，允许做进一步的操作。它返回的还是的 Stream，允许更多的链式操作。常见的中间操作有：

- filter()：对元素进行过滤
- sorted()：对元素排序
- map()：元素的映射
- flatMap()：一个或多个流合并成一个新流
- limit()：限流
- skip()：跳过元素
- distinct()：去除重复元素
- subStream()：获取子 Stream 等

为了更好的演示 Stream 流，在这里新建实体类 fruit，并模拟一些水果信息。

```
public class Fruit {

    private int id;

    private String name;

    private int num;

    private double price;

    public Fruit() {
    }

    public Fruit(int id, String name, int num, double price) {
        this.id = id;
        this.name = name;
        this.num = num;
        this.price = price;
    }

    // 省略 set/get
}
```

Fruit 数据模拟：

```
public class FruitDataTest {

    public static List<Fruit> getFruites(){
        List<Fruit> fruits = new ArrayList<>();
        fruits.add(new Fruit(1,"苹果",100,10));
        fruits.add(new Fruit(2,"葡萄",90,19));
        fruits.add(new Fruit(3,"葡萄柚",30,3.9));
        fruits.add(new Fruit(4,"提子",50,6.8));
        fruits.add(new Fruit(5,"构树果实",200,38.9));
        fruits.add(new Fruit(6,"黄心猕猴桃",405,32.8));
        fruits.add(new Fruit(7,"牛油果",45,5.8));
        fruits.add(new Fruit(8,"圣女果",225,12.8));
        fruits.add(new Fruit(9,"白玉樱桃",325,10.5));
        fruits.add(new Fruit(10,"南洋红香蕉",25,15.3));
        return fruits;
    }

}
```

中间操作并不能打印结果，在这里为了结果的显示提前引入终止操作 forEach。

**1. filter 对元素进行过滤**

filter 方法中是接收一个和 Predicate 函数对应 Lambda 表达式，返回一个布尔值，从流中过滤某些元素。

![image](https://images.gitbook.cn/afa96c00-eb5e-11e9-821e-bba2257c3064)

```
    @Test
    public void test01(){
        // 获取我们模拟的 fruit 数组数据
        List<Fruit> fruites = FruitDataTest.getFruites();
        // 创建数据流
        Stream<Fruit> stream = fruites.stream();
        System.out.println("价格（price）大于 30 的水果 ");
        // 筛选 price 价格大于 30 的水果
        stream.filter(s->s.getPrice()>30.0).forEach(System.out::println);
    }
```

运行结果：

```
价格（price）大于 30 的水果 
Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}
Fruit{id=6, name='黄心猕猴桃', num=405, price=32.8, type=3}
```

**2. filter 对元素进行过滤，可以复用方式**

```
    @Test
    public void test02(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        System.out.println("价格（price）大于 30 的水果 ,方法复用");
        List<Fruit> fruits = filterFruitPrice(fruites, s -> s.getPrice() > 30);
        fruits.forEach(System.out::println);
    }

    public List<Fruit> filterFruitPrice(List<Fruit> fruits, Predicate<Fruit> predicate){
        List<Fruit> list = new ArrayList<>();
        for (Fruit fruit : fruits) {
            if(predicate.test(fruit)){
                list.add(fruit);
            }
        }
        return list;
    }
```

运行结果跟 1 一样。

**3. distinct 去除重复元素**

distinct() 去重，通过流所生成元素的 hashCode() 和 equals() 去除重复元素。如果利用上面的代码是无法进行去重，在实际开发中，大家使用 distinct 需要注意，在这里为了演示效果我在 fruit 实体类重写了 hashCode() 和 equals() 方法才达到去重成功。除了重写了 hashCode() 和 equals() 方法，还可以通过 filter() 自定义函数达到去重效果。

![1570367179654](https://images.gitbook.cn/bf98b4e0-eb5e-11e9-8dea-797ca875d03d)

```
    /**
     * distinct - 去除重复元素
     */
    @Test
    public void test03(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        // 复制 模拟数据的前两条继续追加
        fruites.add(new Fruit(1,"苹果",100,10,2));
        fruites.add(new Fruit(2,"葡萄",90,19,1));
        fruites.add(new Fruit(1,"苹果",100,10,2));
        System.out.println("数据没有去重");
        fruites.stream().forEach(System.out::println);
        System.out.println("\n distinct() 数据去重");
        fruites.stream().distinct().forEach(System.out::println);
    }
```

运行结果：

```
数据没有去重:
Fruit{id=1, name='苹果', num=100, price=10.0, type=2}
Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}
Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}
Fruit{id=4, name='提子', num=50, price=6.8, type=1}
Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}
Fruit{id=6, name='黄心猕猴桃', num=405, price=32.8, type=3}
Fruit{id=7, name='牛油果', num=45, price=5.8, type=2}
Fruit{id=8, name='圣女果', num=225, price=12.8, type=1}
Fruit{id=9, name='白玉樱桃', num=325, price=10.5, type=2}
Fruit{id=10, name='南洋红香蕉', num=25, price=15.3, type=1}
Fruit{id=1, name='苹果', num=100, price=10.0, type=2}
Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}
Fruit{id=1, name='苹果', num=100, price=10.0, type=2}

 distinct() 数据处理
Fruit{id=1, name='苹果', num=100, price=10.0, type=2}
Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}
Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}
Fruit{id=4, name='提子', num=50, price=6.8, type=1}
Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}
Fruit{id=6, name='黄心猕猴桃', num=405, price=32.8, type=3}
Fruit{id=7, name='牛油果', num=45, price=5.8, type=2}
Fruit{id=8, name='圣女果', num=225, price=12.8, type=1}
Fruit{id=9, name='白玉樱桃', num=325, price=10.5, type=2}
Fruit{id=10, name='南洋红香蕉', num=25, price=15.3, type=1}
```

**4. map 映射**

接收一个 Function 函数作为参数，该函数会被应用到每个元素上，并将其映射成由新元素组成的 Stream。 跟 map 相关的还有 mapToInt、mapToLong、mapToDouble，这三个方法可以免除自动装箱/拆箱的额外消耗。

![image](https://images.gitbook.cn/ce849780-eb5e-11e9-b8da-5f5c9df620e8)

```
    @Test
    public void test04(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        // 取出集合中所有的水果名称
        System.out.println("lambda - map() 取出水果名称 ");
        fruites.stream().map(s->s.getName()).forEach(System.out::println);
        System.out.println("\n lambda 方法引用 - map() 取出水果名称 ");
        fruites.stream().map(Fruit::getName).forEach(System.out::println);
    }
```

运行结果：

```
lambda - map() 取出水果名称 
苹果
葡萄
......
 lambda 方法引用 - map() 取出水果名称 
苹果
葡萄
.......
```

**5. flatMap 一个或多个流合并成一个新流**

flatMap 将一个或多个流合并成一个新流。跟 flatMap 相关的还有 flatMapToInt、flatMapToLong、flatMapToDouble，其作用跟上面的 map 衍生方法相同。

![1570373582988](https://images.gitbook.cn/df5a21b0-eb5e-11e9-8dea-797ca875d03d)

理解 map 和 flatMap 之前我们先看来一段代码加深下理解。

```
    /**
     * 集合add 和 addAll
     */
    @Test
    public void test05(){
        ArrayList list1 = new ArrayList();
        list1.add(1);
        list1.add(2);
        list1.add(3);
        ArrayList list2 = new ArrayList();
        list2.add(4);
        list2.add(5);
        list2.add(6);
        // list1.add(list2); //map
        list1.addAll(list2);  // floatmap
        System.out.println(list1);
    }
```

运行结果：

```
add  :
[1, 2, 3, [4, 5, 6]]
addAll :
[1, 2, 3, 4, 5, 6]
```

map 就像集合的 add，而 flatMap 就像集合的 addAll。

```
    /**
     * flatMap- 一个或多个流合并成一个新流
     */
    @Test
    public void test06() {
        // map 则需要通过两次 forEach 才能输出结果 
        Stream<Stream<Integer>> streamStream = Stream.of(Arrays.asList(1, 2, 3), Arrays.asList(4, 5, 6))
                .map(s -> s.stream());
        streamStream.forEach(s -> {
            s.forEach(System.out::print);
            System.out.println("---->");
        });
        // flatMap 形成一个新的流 一次性输出
        Stream.of(Arrays.asList(1, 2, 3), Arrays.asList(4, 5, 6)).flatMap(s -> s.stream()).forEach(s -> {
            System.out.print(s + ", ");
        });
    }
```

运行结果：

```
123---->
456---->
1, 2, 3, 4, 5, 6, 
```

**6. limit 限流**

limit 限流，限定的数量不能未负数，超过元素大小则获取所有的元素。

![image](https://images.gitbook.cn/ede2aea0-eb5e-11e9-821e-bba2257c3064)

```
    @Test
    public void test07(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        System.out.println("");
        // 取出前5条
        System.out.println("limit 限流 前 3 条");
        fruites.stream().limit(3).forEach(System.out::println);
    }
```

运行结果：

```
limit 限流 前 3 条
Fruit{id=1, name='苹果', num=100, price=10.0, type=2}
Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}
Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}
```

**7. skip 跳过元素**

跳过元素，从前第 n 个元素开始组成一个新的流。若流中元素不足 n 个，则返回一个空流。

![image](https://images.gitbook.cn/f9cc26b0-eb5e-11e9-b8da-5f5c9df620e8)

```
    @Test
    public void test08(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        // 取出第 3 条后面的数据
        System.out.println("skip 取出第 3 条后面的数据");
        fruites.stream().skip(3).forEach(System.out::println);
    }
```

运行结果：

```
skip 取出第 3 条后面的数据
Fruit{id=4, name='提子', num=50, price=6.8, type=1}
Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}
Fruit{id=6, name='黄心猕猴桃', num=405, price=32.8, type=3}
Fruit{id=7, name='牛油果', num=45, price=5.8, type=2}
Fruit{id=8, name='圣女果', num=225, price=12.8, type=1}
Fruit{id=9, name='白玉樱桃', num=325, price=10.5, type=2}
Fruit{id=10, name='南洋红香蕉', num=25, price=15.3, type=1}
```

**8. sorted 排序**

指定比较条件进行排序。

![image](https://images.gitbook.cn/08430060-eb5f-11e9-a0db-c9a5fffa48ce)

```
    @Test
    public void test09(){
        // 数组 排序 sorted() 无参
        List<Integer> nums = Arrays.asList(1, 2, 3, 6, 8, 7, 5);
        nums.stream().sorted().forEach(System.out::print);

        List<Fruit> fruites = FruitDataTest.getFruites();

        System.out.println("sorted() 指定条件比较");
        // 按照 价格进行比较
        fruites.stream().sorted((e1,e2)->Double.compare(e1.getPrice(),e2.getPrice())).forEach(System.out::println);
        System.out.println("******** 优化之后 ********");
        // 优化代码
        fruites.stream().sorted(Comparator.comparingDouble(Fruit::getPrice)).forEach(System.out::println);
    }
```

运行结果：

```
1235678
 sorted() 指定条件比较
Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}
Fruit{id=7, name='牛油果', num=45, price=5.8, type=2}
Fruit{id=4, name='提子', num=50, price=6.8, type=1}
Fruit{id=1, name='苹果', num=100, price=10.0, type=2}
Fruit{id=9, name='白玉樱桃', num=325, price=10.5, type=2}
Fruit{id=8, name='圣女果', num=225, price=12.8, type=1}
Fruit{id=10, name='南洋红香蕉', num=25, price=15.3, type=1}
Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}
Fruit{id=6, name='黄心猕猴桃', num=405, price=32.8, type=3}
Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}
```

**9. peek 数据状态**

主要用来查看流中元素的数据状态。

```
    @Test
    public void test10(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        fruites.stream().peek(s-> System.out.println(s.getId())).forEach(System.out::println);

    }
```

运行结果：

```
1
Fruit{id=1, name='苹果', num=100, price=10.0, type=2}
2
Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}
3
Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}
4
Fruit{id=4, name='提子', num=50, price=6.8, type=1}
.......
```

> 此小节代码：com.mtcarpenter.stream.StreamApi02Test.java

#### 4.3 Stream 的终止操作

终止操作将执行中间操作链，并返回结果。在这里列举一下的终止操作：

- forEach()：循环操作 Stream 数据
- toArray()：返回流中元素对应的数组对象
- anyMatch()：检查是否至少匹配一个元素
- allMatch()：检查是否匹配的所有元素
- noneMatch()：检查是否没有匹配的元素
- min()、max()、count()：聚合操作，最小值，最大值，总数量
- reduce()：聚合操作，用来做统计
- collect()：聚合操作，封装目标数据
- findFirst()：获取第一个元素
- findAny()：获取任一元素

**1. forEach 循环操作 Stream 数据**

在前面的例子我们大量使用了这个 forEach，在 stream 中有了终止操作，才回去执行中间操作链。一旦执行终止操作，就执行中间操作链，并产生结果。之后，不会再被使用。

```
    @Test
    public void test01(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        // 获得一个 stream 流
        Stream<Fruit> stream = fruites.stream();
        // 取出前三条
        stream.limit(3).forEach(System.out::println);
        // 已经使用了 终止操作，再次使用会报错
        stream.limit(3).forEach(System.out::println);
    }
```

运行结果：

```
Fruit{id=1, name='苹果', num=100, price=10.0, type=2}
Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}
Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}



java.lang.IllegalStateException: stream has already been operated upon or closed
    at java.util.stream.AbstractPipeline.<init>(AbstractPipeline.java:203)
    at java.util.stream.ReferencePipeline.<init>(ReferencePipeline.java:94)
    at java.util.stream.ReferencePipeline$StatefulOp.<init>(ReferencePipeline.java:647)
    at java.util.stream.SliceOps$1.<init>(SliceOps.java:120)
    .....
```

执行终止操作后，再次使用流会提示我们 Stream 流已经关闭。

**2. toArray——Stream 转数组**

```
    @Test
    public void test02(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        Object[] objects = fruites.stream().map(Fruit::getName).toArray();
        for (Object object : objects) {
            System.out.println(object);
        }
    }
```

运行结果：

```
苹果
葡萄
葡萄柚
提子
构树果实
黄心猕猴桃
牛油果
圣女果
白玉樱桃
南洋红香蕉
```

**3. anyMatch、allMatch、noneMatch**

- anyMatch(Predicate p) 检查是否至少匹配一个元素。
- allMatch(Predicate p) 检查是否匹配的所有元素。
- noneMatch(Predicate p) 检查是否没有匹配的元素。

```
    @Test
    public void test03(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        // anyMatch(Predicate p) 检查是否至少匹配一个元素
        // 检查水果价格是否超过 10
        boolean anyMatch = fruites.stream().anyMatch(s -> s.getPrice() > 10);
        System.out.println("anyMatch:检查水果价格是否超过 10 :"+anyMatch);
        // allMatch(Predicate p) 检查是否匹配的所有元素
        // 检查水果数量都超过 100
        boolean allMatch = fruites.stream().allMatch(s -> s.getNum() > 100);
        System.out.println("allMatch:检查水果数量都超过 100 :"+allMatch);
        // noneMatch(Predicate p) 检查是否没有匹配的元素
        // 检查没有 type 为 1 的元素
        boolean noneMatch = fruites.stream().noneMatch(s -> s.getType() == 1);
        System.out.println("noneMatch:检查没有 type 为 1 的元素 :"+noneMatch);

    }
```

运行结果：

```
anyMatch:检查水果价格是否超过 10 :true
allMatch:检查水果数量都超过 100 :false
noneMatch:检查没有 type 为 1 的元素 :false
```

**4. min、max、count**

min(Comparator p) 返回流中最小值。 max(Comparator c) 返回流中最大值。count 返回流中元素个数。

```
    @Test
    public void test04(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        //min(Comparator p ) 返回流中最小值
        // 返回价格最低的水果
        Optional<Fruit> min = fruites.stream().min((o1,o2)->Double.compare(o1.getPrice(),o2.getPrice() ));
        System.out.println("最低的水果: "+min);
        // max(Comparator c) 返回流中最大值
        // 返回价格最高的水果
        Optional<Fruit> max = fruites.stream().max((o1,o2)->Double.compare(o1.getPrice(),o2.getPrice() ));
        System.out.println("价格最高的水果: "+max);
        // count 返回流中元素个数
        long count = fruites.stream().count();
        System.out.println("流中元素个数 : "+count);
    }
```

运行结果：

```
最低的水果: Optional[Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}]
价格最高的水果: Optional[Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}]
流中元素个数 : 10
```

Optional 下一个知识点进行专门的讲解。

**5. reduce 统计**

reduce(T identity ,BinaryOperator b) 将流中元素反复集合起来，得到一个值，返回 T。reduce 可以传递两个参数，identity 它允许用户提供一个循环计算的初始值。

```
    @Test
    public void test05(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        // reduce(T identity ,BinaryOperator b)  将流中元素反复集合起来，得到一个值。返回 T
        //求水果总数量
        Integer numTotal = fruites.stream().map(Fruit::getNum).reduce(0, Integer::sum);
        System.out.println("水果总数量 : " + numTotal);
        Optional<Integer> reduce = fruites.stream().map(Fruit::getNum).reduce(Integer::sum);
        System.out.println("水果总数量 : " + reduce.get());
    }
```

运行结果：

```
水果总数量 : 2495
水果总数量 : 2495
```

**6. collect——list、set、map**

```
    @Test
    public void test06(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        // collect(Collect c) 将流转换为其他形式。接收一个 Collector 接口的实现。收集、将流转换为其他形式
        // 收集水果名到 list
        System.out.println("************ 收集水果名到 list *******************");
        List<String> list = fruites.stream().map(Fruit::getName).collect(Collectors.toList());
        list.forEach(System.out::println);
        // 收集水果名到 set
        System.out.println("************ 收集水果名到 set  *******************");
        Set<String> set = fruites.stream().map(Fruit::getName).collect(Collectors.toSet());
        set.forEach(System.out::println);
        // 收集到 map,id 作为key,Fruit 对象作为 value
        System.out.println("************ 收集水果到 map   *******************");
        Map<Integer, Fruit> collect = fruites.stream().collect(Collectors.toMap(Fruit::getId, Function.identity()));
        collect.forEach( (s, fruit) -> {
            System.out.println(s+" = "+fruit);
        });
    }
```

运行结果：

```
************ 收集水果名到 list *******************
苹果
葡萄
葡萄柚
提子
构树果实
黄心猕猴桃
牛油果
圣女果
白玉樱桃
南洋红香蕉
************ 收集水果名到 set  *******************
苹果
葡萄
黄心猕猴桃
牛油果
白玉樱桃
葡萄柚
南洋红香蕉
提子
构树果实
圣女果
************ 收集水果到 map   *******************
1 = Fruit{id=1, name='苹果', num=100, price=10.0, type=2}
2 = Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}
3 = Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}
4 = Fruit{id=4, name='提子', num=50, price=6.8, type=1}
5 = Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}
6 = Fruit{id=6, name='黄心猕猴桃', num=405, price=32.8, type=3}
7 = Fruit{id=7, name='牛油果', num=45, price=5.8, type=2}
8 = Fruit{id=8, name='圣女果', num=225, price=12.8, type=1}
9 = Fruit{id=9, name='白玉樱桃', num=325, price=10.5, type=2}
10 = Fruit{id=10, name='南洋红香蕉', num=25, price=15.3, type=1}
```

**7. collect——joining()、groupingBy**

```
joining(CharSequence delimiter, CharSequence prefix,  CharSequence suffix)
```

delimiter 分隔符、prefix 前缀、suffix 后缀。

```
groupingBy(Function<? super T, ? extends K> classifier)
```

classifier 需要操作的元素。

```
   @Test
    public void test07(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        List<String> list = Arrays.asList("a", "b", "c", "d");
        // joining()  delimiter  分隔符, prefix 前缀, suffix 后缀
        System.out.println("*************  joining 连接操作符 *********************");
        String collect = list.stream().collect(Collectors.joining("->","",""));
        System.out.println(collect);
        // groupingBy 分组
        System.out.println("*************  根据 type 进行简单分组 *********************");
        Map<Integer, List<Fruit>> collect1 = fruites.stream().collect(Collectors.groupingBy(Fruit::getType));
        collect1.forEach((k,v)->{
            System.out.println(k+"--"+v);
        });
        // 根据 type 分组求和
        System.out.println("*************  根据 type 分组求和 *************************");
        Map<Integer, Integer> collect2 = fruites.stream().collect(Collectors.groupingBy(Fruit::getType, Collectors.summingInt(Fruit::getNum)));
        collect2.forEach((k,v)->{
            System.out.println("type = "+k+" ---- 总数 = "+v);
        });
        // 求出每一个 type 不同分类的元素总个数
        System.out.println("*************  根据  type 不同分类的元素总个数 ******************");
        Map<Integer, Long> collect3 = fruites.stream().collect(Collectors.groupingBy(Fruit::getType, Collectors.counting()));
        collect3.forEach((k,v)->{
            System.out.println("type = "+k+" ---- 个数 = "+v);
        });
    }
```

运行结果：

```
*************  joining 连接操作符 *********************
a->b->c->d
*************  根据 type 进行简单分组 *********************
1--[Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}, Fruit{id=4, name='提子', num=50, price=6.8, type=1}, Fruit{id=8, name='圣女果', num=225, price=12.8, type=1}, Fruit{id=10, name='南洋红香蕉', num=25, price=15.3, type=1}]
2--[Fruit{id=1, name='苹果', num=100, price=10.0, type=2}, Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}, Fruit{id=7, name='牛油果', num=45, price=5.8, type=2}, Fruit{id=9, name='白玉樱桃', num=325, price=10.5, type=2}]
3--[Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}, Fruit{id=6, name='黄心猕猴桃', num=405, price=32.8, type=3}]
*************  根据 type 分组求和 *************************
type = 1 ---- 总数 = 390
type = 2 ---- 总数 = 500
type = 3 ---- 总数 = 605
*************  根据  type 不同分类的元素总个数 ******************
type = 1 ---- 个数 = 4
type = 2 ---- 个数 = 4
type = 3 ---- 个数 = 2
```

**8. findFirst、findAny**

- findFirst()：获取第一个元素
- findAny()：获取任一元素

```
    @Test
    public void test08(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        //  findFirst 获取第一个元素
        Optional<Fruit> first = fruites.stream().findFirst();
        System.out.println("获取第一个元素："+first);
        // findAny  获取任一元素
        Optional<Fruit> any = fruites.stream().findAny();
        System.out.println("获取任一元素："+any);
    }
```

运行结果：

```
获取第一个元素：Optional[Fruit{id=1, name='苹果', num=100, price=10.0, type=2}]
获取任一元素：Optional[Fruit{id=1, name='苹果', num=100, price=10.0, type=2}]
```

经过多次的实验发现，findAny 每次还是返回第一个元素。

> 此小节代码：com.mtcarpenter.stream.StreamApi03Test.java

#### 4.4 Stream 练习

练习 1：查询所有价格（price）大于 20 的水果名，结果返回为一个 list，并将结果打印显示。

```
/**
 * 
 */
@Test
public void test01() {
    List<Fruit> fruites = FruitDataTest.getFruites();
    List<String> list = fruites.stream().filter(s -> s.getPrice() > 20).map(Fruit::getName).collect(Collectors.toList());
    System.out.println("价格（price）大于 20 的水果名:");
    list.forEach(System.out::println);
}
```

运行结果：

```
价格（price）大于 20 的水果名:
构树果实
黄心猕猴桃
```

练习 2：取出水果列表中排列 3、4、5 的水果，并将结果打印显示。

```
    @Test
    public void test02() {
        List<Fruit> fruites = FruitDataTest.getFruites();
        System.out.println("水果中 3，4，5 的水果:");
        fruites.stream().skip(2).limit(3).forEach(System.out::println);
    }
```

运行结果:

```
水果中 3，4，5 的水果:
Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}
Fruit{id=4, name='提子', num=50, price=6.8, type=1}
Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}
```

练习 3：取出水果列表中，水果价格最高的水果名字，并将结果打印显示。

```
    @Test
    public void test03() {
        List<Fruit> fruites = FruitDataTest.getFruites();
        System.out.println("水果价格最高的水果:");
        // 优化前
        fruites.stream().max(((o1, o2) -> Double.compare(o1.getPrice(), o2.getPrice()))).map(Fruit::getName);
        // 优化后
        Optional<String> optional = fruites.stream().max((Comparator.comparingDouble(Fruit::getPrice))).map(Fruit::getName);
        System.out.println(optional.get());
    }
```

运行结果：

```
水果价格最高的水果:
构树果实
```

练习 4：取出水果列表中，水果价格最低的水果名字，并将结果打印显示。

```
    @Test
    public void test04() {
        List<Fruit> fruites = FruitDataTest.getFruites();
        System.out.println("水果价格最低的水果:");
        Optional<String> optional = fruites.stream().min((Comparator.comparingDouble(Fruit::getPrice))).map(Fruit::getName);
        System.out.println(optional.get());
    }
```

运行结果：

```
水果价格最低的水果:
葡萄柚
```

练习 5：取出水果列表 type 为 2 的水果价格总和,并将结果打印显示。

```
    @Test
    public void test05() {
        List<Fruit> fruites = FruitDataTest.getFruites();
        Optional<Double> optional = fruites.stream().filter(s -> s.getType() == 2).map(Fruit::getPrice).reduce(Double::sum);
        System.out.println("水果 type 为 2 的水果价格总和：" + optional.get());
    }
```

运行结果：

```
水果 type 为 2 的水果价格总和：30.2
```

练习 6：根据水果列表中 type 进行分类，结果返回为一个 list，并将结果打印显示。

```
    @Test
    public void test06() {
        List<Fruit> fruites = FruitDataTest.getFruites();
        Map<Integer, List<Fruit>> listMap = fruites.stream().collect(Collectors.groupingBy(Fruit::getType, Collectors.toList()));
        listMap.forEach((key, fruits) -> System.out.println(key + "----" + fruits));
    }
```

运行结果：

```
1----[Fruit{id=2, name='葡萄', num=90, price=19.0, type=1}, Fruit{id=4, name='提子', num=50, price=6.8, type=1}, Fruit{id=8, name='圣女果', num=225, price=12.8, type=1}, Fruit{id=10, name='南洋红香蕉', num=25, price=15.3, type=1}]
2----[Fruit{id=1, name='苹果', num=100, price=10.0, type=2}, Fruit{id=3, name='葡萄柚', num=30, price=3.9, type=2}, Fruit{id=7, name='牛油果', num=45, price=5.8, type=2}, Fruit{id=9, name='白玉樱桃', num=325, price=10.5, type=2}]
3----[Fruit{id=5, name='构树果实', num=200, price=38.9, type=3}, Fruit{id=6, name='黄心猕猴桃', num=405, price=32.8, type=3}]
```

练习 7：根据水果列表进行价格降序，取出前 3 的水果名，结果返回一个 list，并将结果打印显示。

```
    @Test
    public void test07() {
        List<Fruit> fruites = FruitDataTest.getFruites();
        List<String> result = fruites.stream().sorted((o1, o2) -> Double.compare(o2.getPrice(), o1.getPrice()))
                .map(Fruit::getName).limit(3).collect(Collectors.toList());
        System.out.println("前 3 的水果名为: " + result);
    }
```

运行结果：

```
前 3 的水果名为: [构树果实, 黄心猕猴桃, 葡萄]
```

练习 8：求出水果列表每一个分类（type）下水果的数量总和（num）。

```
    @Test
    public void test08(){
        List<Fruit> fruites = FruitDataTest.getFruites();
        Map<Integer, Integer> result = fruites.stream().collect(Collectors.groupingBy(Fruit::getType,
                Collectors.summingInt(Fruit::getNum)));
        result.forEach((k,v)->{
            System.out.println("分类为："+k+"---"+v);
        });
    }
```

运行结果：

```
分类为：1---390
分类为：2---500
分类为：3---605
```

> 此小节代码：com.mtcarpenter.stream.QuestionTest.java

### 5. 并行和串行

Stream 流有串行和并行两种，串行流上的操作是在一个线程中依次完成，而并行流则是在多个线程上同时执行。并行与串行的流可以相互切换：通过 stream.sequential() 返回串行的流，通过 stream.parallel() 返回并行的流。相比较串行的流，并行的流可以很大程度上提高程序的执行效率。并行流就是把一个内容分成多个数据块，并用不同的线程分别处理每个数据块的流。

Stream 的并行模式使用了 Fork/Join 框架。Fork/Join 框架是 Java 7 中加入的一个并行任务框架，可以将任务拆分为多个小任务，每个小任务执行完的结果在合并成为一个结果，减少了线程的等待时间，提高了性能。

![image](https://images.gitbook.cn/3ada8250-eb5f-11e9-a0db-c9a5fffa48ce)

下面是分别用串行和并行的方式对集合进行排序。

**1. sequential 通过集合方式创建串行流**

```
    @Test
    public void test01() {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 1000000; i++) {
            double d = Math.random() * 1000;
            list.add(d + "");
        }
        //获取系统开始排序的时间点
        long start = System.nanoTime();
        int count = (int) ((Stream) list.stream().sequential()).sorted().count();
        //获取系统结束排序的时间点
        long end = System.nanoTime();
        //得到串行排序所用的时间
        long ms = TimeUnit.NANOSECONDS.toMillis(end - start);
        System.out.println(ms + "ms");

    }
```

运行结果：

```
// 随机生成 100 万个数 ，用时
720ms
// 随机生成 1000 万个数，用时
8595ms
```

**2. parallel 通过集合方式创建并行流**

```
    @Test
    public void test02(){
        List<String> list = new ArrayList<String>();
        for(int i=0;i<10000000;i++){
            double d = Math.random()*1000;
            list.add(d+"");
        }
        //获取系统开始排序的时间点
        long start = System.nanoTime();
        int count = (int)((Stream) list.stream().parallel()).sorted().count();
        //获取系统结束排序的时间点
        long end = System.nanoTime();
        //得到并行排序所用的时间
        long ms = TimeUnit.NANOSECONDS.toMillis(end-start);
        System.out.println(ms+"ms");
    }
```

运行结果：

```
// 随机生成 100 万个数 ，用时
420ms
// 随机生成 1000 万个数，用时
3504ms
```

通过两个简单生成的数组的串行和并行例子，数目越大运行时间相差越大。

> 此小节代码：com.mtcarpenter.sequentialparallel.SequentialParallelTest.java

### 6. 用 Optional 取代空指针

Optional 类主要解决的问题是臭名昭著的空指针异常（NullPointerException）。Google 公司著名的 Guava 项目引入了 Optional 类，Guava 通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。受到 Google Guava 的启发，Optional 类已经成为 Java 8 类库的一部分。

Optional<T> 类（java.util.Optional）是一个容器类，代表一个值存在或不存在，原来用 null 表示一个值不存在，现在 Optional 可以更好的表达这个概念。并且可以避免空指针异常。

**创建 Optional 类对象的方法：**

- Optional.of(T t)：创建一个 Optional 实例
- Optional.empty()：创建一个空的 Optional 实例
- Optional.ofNullable(T t)：t 可以为 null

**判断Optional 容器中是否包含对象：**

- isPresent()：判断容器中是否包含值，用此方法可避免 NoSuchElementException 异常。

**获取Optional 容器的对象：**

- T get()：如果此Optional中存在值，则返回该值，否则抛出 NoSuchElementException。
- T orElse(T other)：返回值（如果存在），否则返回其他。
- `T orElseGet(Supplier<? extends T> other)`：返回值（如果存在），否则调用其他并返回该调用的结果。
- `<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)`：返回包含的值（如果存在），否则将引发由提供的供应商创建的异常。

**1. Optional.of(T t)：创建一个 Optional 实例**

```
/**
 * Optional.of(T t)：创建一个 Optional 实例
 */
@Test
public void test01(){
    Fruit fruit = new Fruit();
    Optional<Fruit> fruit1= Optional.of(fruit);
    System.out.println(fruit1.get());
    fruit = null;
    Optional<Fruit> fruit2 = Optional.of(fruit);
    // of 传入 null，使用 get() 报错 NullPointerException
    System.out.println(fruit2.get());
}
```

运行结果：

```
Fruit{id=0, name='null', num=0, price=0.0, type=0}

java.lang.NullPointerException
    at java.util.Objects.requireNonNull(Objects.java:203)
    at java.util.Optional.<init>(Optional.java:96)
    at java.util.Optional.of(Optional.java:108)
    ......
```

使用Optional.of() 创建 Optional 实例的时候不能传入 null，否则会报空指针异常。

**2. Optional.empty()：创建一个空的 Optional 实例**

```
    @Test
    public void test02(){
        Optional<Object> empty = Optional.empty();
        System.out.println(empty.get());
    }
```

运行结果：

```
java.util.NoSuchElementException: No value present
    at java.util.Optional.get(Optional.java:135)
    .........
```

一旦 Optional 容器 empty 时再调用其 get()，就会报 NoSuchElementException 异常。

**3. isPresent()：判断容器中是否包含值**

```
    @Test
    public void test03(){
        // isPresent 传入 null
        Optional<Fruit> fruitOptional = Optional.ofNullable(null);
        if (fruitOptional.isPresent()){
            System.out.println("********* isPresent 传入 null *********");
            System.out.println(fruitOptional.get());
        }
        // isPresent 传入非 null
        Optional<Fruit> fruitOptional1 = Optional.ofNullable(new Fruit());
        if (fruitOptional1.isPresent()){
            System.out.println("*************** isPresent 传入非 null *************");
            System.out.println(fruitOptional1.get());
        }
    }
```

运行结果：

```
*************** isPresent 传入非 null *************
Fruit{id=0, name='null', num=0, price=0.0, type=0}
```

**3. ofNullable 和 orElse**

```
    @Test
    public void test04(){
        Fruit fruit = new Fruit();
        System.out.println("******** ofNullable 传入非 null    **********");
        Optional<Fruit> fruit1 = Optional.ofNullable(fruit);
        Fruit fruit2 = (Fruit) fruit1.orElse(new Fruit("a"));
        System.out.println(fruit2);
        System.out.println("******** ofNullable 传入 null   **********");
        fruit = null;
        Optional<Fruit> fruit3 = Optional.ofNullable(fruit);
        Fruit fruit4 = (Fruit) fruit3.orElse(new Fruit("我是null"));
        System.out.println(fruit4);
    }
```

运行结果：

```
******** ofNullable 传入非 null    **********
Fruit{id=0, name='null', num=0, price=0.0, type=0}
******** ofNullable 传入 null   **********
Fruit{id=0, name='我是null', num=0, price=0.0, type=0}
```

如果 ofNullable 调用对象包含值，返回该值，否则返回 orElse 中给出的默认值，此方法也可以避免出现空指针异常。

> 此小节代码：com.mtcarpenter.option.Option01Test.java

### 7. Java 日期操作

Java 8 中引入了新的日期时间 API，以克服旧的日期时间 API 的以下缺点：

- 非线程安全的：java.util.Date不是线程安全的，新的日期时间 API 是不可变的，并且没有 setter 方法。
- 更少的操作：在旧的API中，日期操作很少，但是新的 API 为我们提供了许多日期操作。

Java 8 在 java.time 包下引入了很多新的关于日期的 API，其中最重要的两个 API：

- Local（本地）：简化了日期时间的处理，没有时区的问题。
- Zoned（分区）：通过制定的时区处理日期时间。

接下来 将通过例子对 Java 8 中提供了 LocalDate、LocalTime 和 LocalDateTime 分别表示日期、时间和日期时间进行讲解。

**1. LocalDate 日期操作**

```
    @Test
    public void test01(){
        // 获取当前时间
        LocalDate now = LocalDate.now();
        System.out.println(now);
        // 构造指定日期 2019-10-01
        LocalDate localDateOf = LocalDate.of(2019, Month.OCTOBER, 1);
        System.out.println("构造指定日期 2019-10-01 = "+localDateOf);
        // 获取指定时区的日期
        LocalDate shanghai = LocalDate.now(ZoneId.of("Asia/Shanghai"));
        System.out.println("获取当前上海的日期="+shanghai);
        // 分别获取年月日
        System.out.println("年："+now.getYear()+" 月："+now.getMonthValue()+" 日："+now.getDayOfMonth());
        // 日期比较
        System.out.println("now 和 localDateOf 日期比较 :"+now.equals(localDateOf));
    }
```

运行效果：

```
当前日期：2019-10-07
构造指定日期 2019-10-01 = 2019-10-01
获取当前上海的日期=2019-10-07
年：2019 月：10 日：7
now 和 localDateOf 日期比较 :false
```

**2. LocalTime 时间**

```
    public void test02() {
        // 获取当前时间
        LocalTime now = LocalTime.now();
        System.out.println("获取当前时间：" + now);

        //获取指定时间 23：10
        LocalTime date1 = LocalTime.of(23, 10);
        System.out.println("获取指定时间: " + date1);

        //将字符串格式化成时间
        LocalTime date2 = LocalTime.parse("23:10:30");
        System.out.println("将字符串格式化成时间: " + date2);

        // 在当前时间增加两个小时
        LocalTime date3 = now.plusHours(2);
        System.out.println("当前时间:" + now + " 增加两个小时:" + date3);
    }
```

运行效果：

```
获取当前时间：23:26:29.786
获取指定时间: 23:10
将字符串格式化成时间: 23:10:30
当前时间:23:26:29.786 增加两个小时:01:26:29.786
```

**3. LocalDateTime 日期时间**

```
    @Test
    public void test03() {
        // 获取当前日期时间
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println("获取当前日期时间：" + localDateTime);
        // DateTimeFormatter 是线程安全的 ，而之前的 SimpleDateFormat 是线程安全的
        DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String formatDateTime = localDateTime.format(format);
        System.out.println("格式化后的日期 ： " + formatDateTime);
        // 自定义日期时间 2019-10-01 23:30
        LocalDateTime localDateTime1 = LocalDateTime.of(2019, Month.OCTOBER, 1, 23, 30);
        System.out.println("自定义日期时间：" + localDateTime1);
        // 字符串转日期时间
        String datetimeText = "2019-10-07 23:30:30";
        LocalDateTime localDateTime2 = LocalDateTime.parse(datetimeText, format);
        System.out.println("字符串转日期时间：" + localDateTime2);

    }
```

运行效果：

```
获取当前日期时间：2019-10-07T23:29:59.511
格式化后的日期 ： 2019-10-07 23:29:59
自定义日期时间：2019-10-01T23:30
字符串转日期时间：2019-10-07T23:30:30
```

在上面我们列举了部分的方法的使用，还有更多的方法在这里大家自行联系下。在这里把 LocalDateTime 方法列举在下方方便大家练习：

- now()：返回当前日期和时间的静态方法
- of()：从指定年份、月份、日期、小时、分钟、秒和毫秒创建LocalDateTime的静态方法。
- getYear()、getMonthValue()、getDayOfMonth(), getHour()、getMinute()、getSecond()：以int形式返回此LocalDateTime的年，月，日，小时，分钟或秒部分。
- plusDays()、minusDays()：使用当前 LocalDateTime 添加或减去指定的天数。
- plusWeeks()、minusWeeks() 使用当前 LocalDateTime 添加或减去指定的周数。
- plusMonths、minusMonths()：使用当前 LocalDateTime 添加或减去指定的月数。
- plusYears()、minusYears()：使用当前 LocalDateTime 添加或减去指定的年数。
- plusHours()、minusHours()：使用当前 LocalDateTime 添加或减去指定的小时数。
- plusMinutes()、minusMinutes() 使用当前 LocalDateTime 添加或减去指定的分钟数。
- plusSeconds()、minusSeconds()：使用当前 LocalDateTime 添加或减去指定的秒数。
- IsAfter()、isBefore()：检查当前 LocalDateTime 是否在指定的日期时间之后或之前。
- withDayOfMonth()：拷贝当前 LocalDateTime，并将月份中的某天设置为指定值。
- withMonth()、withYear()：拷贝当前 LocalDateTime，设置月或年为指定值。
- withHour()、withMinute(), withSecond()：拷贝当前 LocalDateTime，将小时/分钟/秒设置为指定值。

其 LocalDate 和 LocalTime 也包含如 now、of 等。如 LocalDate 中不可用的 plusHours，plusMinutes 和 plusSeconds。 LocalTIme 中不可用 getYear()、getMonthValue()、getDayOfMonth()。

> 此小节代码：com.mtcarpenter.date.DateTest.java

### 8. Java 8 Nashorn JavaScript

从 JDK 1.8 开始，Nashorn 取代 Rhino（JDK 1.6、JDK1.7）成为 Java 的嵌入式 JavaScript 引擎。Nashorn 完全支持 ECMAScript 5.1 规范以及一些扩展。与先前的 Rhino 实现相比，这带来了 2 到 10 倍的性能提升。

在 JDK 1.8 中还引入了 jjs，jjs 是个基于 Nashorn 引擎的命令行工具。用于在控制台执行 JavaScript 代码。

怎样使用 jjs 呢？我们在任意盘中打开 CMD 窗口便可进入，如下：

![image](https://images.gitbook.cn/50354680-eb5f-11e9-b8da-5f5c9df620e8)

\1. jjs 状态下可以直接输入 JavaScript 代码，打开控制台，输入以下命令：

```
F:\>jjs
jjs> print("Java 8 新特性必知必会")
Java 8 新特性必知必会
jjs> 4*5
20
jjs> 10-2
8
jjs> 10+10
20
jjs>
```

\2. jjs 也可以直接运行 JS 文件：

```
//  hello.js 文件内容  print("Java 8 新特性必知必会")
F:\>jjs hello.js
Java 8 新特性必知必会
```

\3. Java 中运行 JavaScript：

```
    /**
     * 
     */
    @Test
    public void test01(){
        ScriptEngineManager scriptEngineManager = new ScriptEngineManager();
        ScriptEngine nashorn = scriptEngineManager.getEngineByName("nashorn");
        String name = "Java 8 新特性必知必会";

        try {
            nashorn.eval("print('" + name + "')");
            Integer result = (Integer)nashorn.eval("5 + 20/2 ");
            System.out.println(result);
        } catch (ScriptException e) {
            System.out.println("script 执行出错 : " + e.getMessage());
        }
    }
```

运行结果：

```
Java 8 新特性必知必会
15
```

> 此小节代码：com.mtcarpenter.nashorn.NashornScriptTest.java

### 9. Java 8 Base64 编码解码

Base64 是一种常见的字符编码解码方式 ，一般用于将二进制数据编码为更具可读性的 Base64 进制格式。在 JDK 1.7 及之前我们会使用 sun.misc 或者 Apache Commons Codec 所提供的 Base64 编解码器 。在 Java 8 中，Base64 编码已经成为 Java 类库的标准。

Java 8 中的 java.util.Base64 类提供了三种类型的 Base64 编码解码格式：

- 基本：输出被映射到一组字符 `A-Za-z0-9+/`，编码不添加任何行标，输出的解码仅支持 `A-Za-z0-9+/`。
- URL：输出映射到一组字符 `A-Za-z0-9+_`，输出是 URL 和文件。
- MIME：输出隐射到 MIME 友好格式。输出每行不超过 76 字符，并且使用 `\r` 并跟随 `\n` 作为分割。编码输出最后没有行分割。

**1. 基本的编码解码器**

```
    /**
     * Base64 编码 解码
     */
    @Test
    public void test01() {
        try {
            // 编码
            String base64encoded = Base64.getEncoder().encodeToString(
                    "Java 8 新特性必知必会 ".getBytes("utf-8"));
            System.out.println("编码: " + base64encoded);
            // 解码
            byte[] base64decoded = Base64.getDecoder().decode(base64encoded);
            System.out.println("解码："+new String(base64decoded, "utf-8"));

        } catch (UnsupportedEncodingException e) {
            System.out.println("编码异常：" + e.getMessage());
        }

    }
```

运行结果：

```
编码: SmF2YSA4IOaWsOeJueaAp+W/heefpeW/heS8miA=
解码：Java 8 新特性必知必会 
```

**2. MIME Base64 编码解码器**

```
    @Test
    public void test02()  {
        try {
            StringBuilder stringBuilder = new StringBuilder();
            for (int i = 0; i < 5; ++i) {
                stringBuilder.append(UUID.randomUUID().toString());
            }
            // 编码
            byte[] bytes = stringBuilder.toString().getBytes("utf-8");
            String mimeEncoded = Base64.getMimeEncoder().encodeToString(bytes);
            System.out.println("加密："+mimeEncoded);
            // 解码
            byte[] base64decoded = Base64.getMimeDecoder().decode(mimeEncoded);
            System.out.println("解码："+new String(base64decoded, "utf-8"));

        } catch(UnsupportedEncodingException e) {
            System.out.println("编码解码异常：" + e.getMessage());
        }
    }
```

运行结果：

```
加密：N2JhMjRjZTItM2Y1Yy00OWQ5LTk3NzEtOWIzMmMwYWI4MjcxMzY2YTgyYWQtZWMwMi00OWUxLTgx
Y2YtOTQ4MzRlOGY2NmI4ZDJlZjcxMDEtN2FhOS00MTQyLWI4N2MtZmVhMzdkYWQ5YTdkMDFhNDk4
NmItNmI3NC00OWM3LWE0MTUtNmI2Y2UwZjJmMzk2ZTgzMzUxYzQtNGUwNS00NTRhLWFjYTgtNzlk
ZDMzNTM1ZjQ5
解码：7ba24ce2-3f5c-49d9-9771-9b32c0ab8271366a82ad-ec02-49e1-81cf-94834e8f66b8d2ef7101-7aa9-4142-b87c-fea37dad9a7d01a4986b-6b74-49c7-a415-6b6ce0f2f396e83351c4-4e05-454a-aca8-79dd33535f49
```

**3. URL 和文件名安全的编码解码器**

```
    @Test
    public void test03() {
        try {
            // 编码
            String base64encoded = Base64.getUrlEncoder().encodeToString(
                    "Java 8 新特性必知必会 ".getBytes("utf-8"));
            System.out.println("编码：" + base64encoded);

            // 解码
            byte[] base64decodedBytes = Base64.getUrlDecoder().decode(base64encoded);
            System.out.println("解码：" + new String(base64decodedBytes, "utf-8"));

        } catch (UnsupportedEncodingException e) {
            System.out.println("编码解码异常：" + e.getMessage());
        }
    }
```

运行结果：

```
编码：SmF2YSA4IOaWsOeJueaAp-W_heefpeW_heS8miA=
解码：Java 8 新特性必知必会 
```

> 此小节代码：com.mtcarpenter.base64.Base64Test.java

### 10. Java 8 基础面试题、知识回顾

**1. JDK 8 引入了那些新特性？**

Java 8 中添加了许多新特性。以下是重要功能列表：

- Lambda 表达式
- 方法引用
- 默认方法
- Stream API
- Date Time API
- Optional
- Nashorn, JavaScript 引擎

**2. JDK 8 主要优点？**

Java 有如下优势：

- 速度更快
- 减少代码的 Lambda 表达式
- 强大的 Stream API
- 便于并行
- 最大化减少空指针异常：Optional
- Nashorn，JavaScript 引擎，允许我们在 JVM 上运行特定的 JavaScript 应用

**3. 什么是 Lambda 表达式？**

Lambda 表达式可以理解为简洁地表示可传递的匿名函数的一种方式：它没有名称，但它有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。使用 Lambda 表达式的代码会更加简洁，可读性更强。

**4. 简要说明 Lambda 表达构成的三分部分？**

- ->：Lambda 操作符 或 箭头操作符
- -> 左边：Lambda 形参列表 （接口中的抽象方法的形参列表）
- -> 右边：Lambda 体 （重写的抽象方法的方法体）

**5. Lambda 表达式有哪些重要特征？**

以下是 Lambda 表达式的重要特征：

- 可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。
- 可选的参数圆括号：一个参数无需定义圆括号，但无参和多个参数需要定义圆括号。
- 可选的大括号：如果主体包含了一个语句，就不需要使用大括号。
- 可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个参数。

**6. 简要阐述 Java 8 中的方法引用？**

方法参考用于参考功能接口的方法。Lambda 表达式只不过是一种紧凑的方式。您可以简单地将 Lambda 表达式替换为方法引用。

方法引用有以下几大类：

- 类名 :: 静态方法名
- 对象 :: 实例方法名
- 类名 :: 实例方法名
- 类名 ::new
- 数组 ::new

**7. 什么是功能接口？**

功能接口是那些只能具有一种抽象方法的接口，可以具有静态方法，默认方法或可以覆盖 Object 的类方法。Java中已经存在许多功能接口，例如 Comparable，Runnable。由于我们在 Runnable 中只有一种方法，因此将其视为功能接口。

我们需要遵循以下规则来定义功能接口：

- 用一个并且只有一个抽象方法定义一个接口。
- 我们不能定义多个抽象方法。
- 在接口定义中使用 @FunctionalInterface 注解。
- 我们可以定义许多其他方法，例如默认方法，静态方法。
- 如果我们将 java.lang.Object 类的方法重写为抽象方法，则该方法不算作抽象方法。

**8. 谓词和函数有什么区别？**

这两个都是功能接口。

谓词是一个单参数函数，返回 true 或 false 。该表达式可用作 Lambda 表达式或任何方法引用的赋值目标。

Function <T,R> 也是单参数函数，但是区别在于它返回一个对象。这里 T 代表函数的输入，R 代表结果的类型。这两个都可用作 Lambda 表达式或方法引用的分配目标。

**9. Stream 是什么？**

Stream 是 Java 8 中处理集合的关键抽象概念，它可以指定希望对集合的操作，可以执行复杂的查找、过滤和映射数据等操作。

Stream 的特点：

- Stream 自己不会存储元素。
- Stream 的操作不会改变源对象。相反，他们会返回一个持有结果的新 Stream。
- Stream 操作是延迟执行的。它会等到需要结果的时候才执行。也就是执行终端操作的时候。

**10. 创建 Stream 的三个步骤？**

- 创建 Stream：从集合数据源中，获取一个数据流。
- 中间操作：中间操作链，对数据进行处理
- 终止：终止操作，用来执行中间操作链，返回结果。

**11. Java 8 中的 Optional 是什么？**

它是一个有界集合，它最多仅包含一个元素。它是“空”值的替代方法。可选是作为 Java SE 8 的一部分引入的最终类。它在 java.util 包中定义。

它用于表示存在或不存在的可选值。它可以包含一个值或零值。如果它包含一个值，我们可以得到它。否则，我们什么也得不到。

Optional 的主要优点是：

- 它用于避免空检查。
- 它用于避免“ NullPointerException”。

**12. 解释 System.out :: println 表达式吗？**

System.out :: println 方法是对 System 类的 out 对象的静态方法 println 方法引用。

**13. Biconsumer <t,u> 功能接口的目的是什么？**

表示该接口接受两个输入参数且不返回结果的操作。t 是函数的第一个参数类型，u 是函数的第二个参数类型。

**14. Bifunction <t,u,r> 功能接口的目的是什么？**

表示该接口两个参数并产生一个结果值返回。t 是函数的第一个参数类型，u 是函数的第二个参数类型，r 是函数的结果类型。

**15. Nashorn是什么？**

这是 Java 8 中附带的用于 Java 平台的新 Java 处理引擎。直到 JDK 7 Java 平台使用 Rhino 作为处理引擎。这是一个 JavaScript 处理引擎。Nashorn 可更好地符合 ECMA 标准化 JavaScript 规范。与以前的版本相比，它还提供了更好的运行时性能。

**16. Java 8 中的 Time API 和 旧日期的差别？**

- 线程安全：java.util.Date 是可变的，并且不是线程安全的。即使 java.text.SimpleDateFormat 也不是线程安全的。Java 8 新的日期和时间 API 是线程安全的。
- 性能：Java 8 新的 API 在性能上优于旧的 Java API。
- 更具可读性：诸如 Calendar 和 Date 之类的旧 API 设计不良且难以理解。Java 8 日期和时间 API 易于理解并符合 ISO 标准。

**17. 简单说下 Java 8 中 java.time 下两个比较重要的 API？**

- Local（本地）：简化了日期时间的处理，没有时区的问题。
- Zoned（时区）：通过制定的时区处理日期时间。

**18. Java 8 中日期常用的三个类？**

LocalDate、LocalTime 和 LocalDateTime 类。

### 11. Java 8 编程面试题、知识回顾

**1. 使用 Lambda 表达式实现 Runnable？**

```
    @Test
    public void test01(){
        System.out.println("*************  JDK 1.7 *****************");
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println(" JDK 1.7  实现 Runnable");
            }
        };
        runnable.run();

        System.out.println("*************  JDK 1.8 Lambda *****************");
        Runnable runnable2 = () -> {
            System.out.println("JDK 1.8  实现 Runnable");
        };
        runnable2.run();
    }
```

运行结果：

```
*************  JDK 1.7 *****************
 JDK 1.7  实现 Runnable
*************  JDK 1.8 Lambda ***********
JDK 1.8  实现 Runnable
```

**2. 使用大小 Lambda 比较两个数的大小？**

```
    @Test
    public void test02(){
        System.out.println("*************  JDK 1.7 *****************");
        Comparator<Integer> comparator1  =   new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return Integer.compare(o1,o2);
            }
        };
        int compare1 = comparator1.compare(10,20);
        System.out.println("10 和 20 比较："+compare1);

        System.out.println("*************  JDK 1.8 *****************");
        Comparator<Integer> comparator2 =  (o1,o2) -> Integer.compare(o1,o2);
        int compare2 = comparator2.compare(10, 10);
        System.out.println("10 和 10 比较："+compare2);

        Comparator<Integer> comparator3 = Integer::compare;
        int compare3 = comparator3.compare(20, 10);
        System.out.println("20 和 10 比较："+compare3);
    }
```

运行结果：

```
*************  JDK 1.7 *****************
10 和 20 比较：-1
*************  JDK 1.8 *****************
10 和 10 比较：0
20 和 10 比较：1
```

**3. 使用 Stream 打印集合中的字符串非空和空的数量？**

```
    @Test
    public void test03(){
        List<String> stringList = Arrays.asList("Java","8","","新特性","","","必知必会");
        // 非空字符串的数量
        long count = stringList.stream().filter(s -> !s.isEmpty()).count();
        System.out.println("集合中非空字符串的数量为："+count);
        // 空字符串的数量
        long count1 = stringList.stream().filter(String::isEmpty).count();
        System.out.println("集合中空字符串的数量为："+count1);
    }
```

运行结果：

```
集合中空字符串的数量为：4
集合中空字符串的数量为：3
```

**4. 使用 Stream 将集合的中的非空字符串转为 list？**

```
    @Test
    public void test04(){
        List<String> stringList = Arrays.asList("Java","8","","新特性","","","必知必会");
        // 非空字符串的收集
        List<String> list = stringList.stream().filter(s -> !s.isEmpty()).collect(Collectors.toList());
        list.forEach(System.out::println);
    }
```

运行结果：

```
Java
8
新特性
必知必会
```

**5. 使用 Stream 将集合的中的非空字符串使用逗号（,）拼接？**

```
    @Test
    public void test05(){
        List<String> stringList = Arrays.asList("Java","8","","新特性","","","必知必会");
        // 非空字符串的 逗号（,）拼接
        String collect = stringList.stream().filter(s -> !s.isEmpty()).collect(Collectors.joining(","));
        System.out.println("非空字符串的 逗号（,）拼接："+collect);
    }
```

**6. 使用 Stream 将集合的中偶数求和？**

```
    @Test
    public void test06(){
        List<Integer> number = Arrays.asList(5,4,3,8,6);
        // 偶数求和
        int result = number.stream().filter(x->x%2==0).reduce(0,(o1,o2)-> o1+o2);
        System.out.println("偶数和为："+ result);
    }
```

运行结果：

```
偶数和为：18
```

**7. 查找水果列表中，水果 type 为 1、价格最高的水果名？**

```
    @Test
    public void test07() {
        List<Fruit> fruites = FruitDataTest.getFruites();
        Optional<String> optional = fruites.stream().filter(s -> s.getType() == 1).max((Comparator.comparingDouble(Fruit::getPrice)))
                .map(Fruit::getName);
        System.out.println("水果 type 为 1 价格最高的水果名为：" + optional.get());
    }
```

运行结果：

```
水果 type 为 1 价格最高的水果名为：葡萄
```

**8. 查找水果列表中，水果 type 为 1，价格最低的水果名？**

```
    @Test
    public void test08() {
        List<Fruit> fruites = FruitDataTest.getFruites();
        Optional<String> optional = fruites.stream().filter(s -> s.getType() == 1).min((Comparator.comparingDouble(Fruit::getPrice)))
                .map(Fruit::getName);
        System.out.println("水果 type 为 1 价格最低的水果名为：" + optional.get());
    }
```

运行结果：

```
水果 type 为 1 价格最低的水果名为：提子
```

**9. 使用 summaryStatistics 方法，求出水果列表中价格最高、最低、总和、平均值？**

```
    @Test
    public void test09() {
        List<Fruit> fruites = FruitDataTest.getFruites();
        DoubleSummaryStatistics statistics = fruites.stream().mapToDouble(Fruit::getPrice).summaryStatistics();
        System.out.println("价高最高值为：" + statistics.getMax());
        System.out.println("价高最低值为：" + statistics.getMin());
        System.out.println("价高总和值为：" + statistics.getSum());
        System.out.println("价高平均值为：" + statistics.getAverage());
    }
```

运行结果：

```
价高最高值为：38.9
价高最低值为：3.9
价高总和值为：155.79999999999998
价高平均值为：15.579999999999998
```

**10. 使用 Java 8 格式化当前日期格式？**

```
    @Test
    public void test10() {
        LocalDateTime localDateTime = LocalDateTime.now();
        DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String formatDateTime = localDateTime.format(format);
        System.out.println("格式化后的日期 ： " + formatDateTime);
    }
```

运行结果：

```
格式化后的日期 ： 2019-10-09 02:32:21
```

> 此小节代码：com.mtcarpenter.interview.QuestionTest.java

代码地址：

> <https://gitee.com/mtcarpenter/Java-8-Learning>