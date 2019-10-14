# MyBatis Generator踩坑与自救

![96](https://upload.jianshu.io/users/upload_avatars/5858794/0f0b2860-1bb1-4015-8bbb-c6ef860a88ff.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96)

 

[WinstonYep](https://www.jianshu.com/u/a1b6f9ae4e2b)

 

关注

 1.0 2017.07.31 13:29* 字数 1873 阅读 8421评论 15喜欢 17赞赏 1

> 前言

我最近在使用MyBatis Generator的过程中遇到了点问题，网上虽然已有相关的解决方案，但结果不尽人意，都只是在规避问题，并没有真正的解决问题。所以我亲自操刀，深入源码，窥探其背后的秘密。

在进行了一番调试分析后，俺最终将问题解决了。这里就跟大伙剖析下鄙人是如何折腾的。

> 环境

JAVA8
数据库mysql 5.7
mysql驱动mysql-connector-java-6.0.6.jar
Mybatis Generator 1.3.5

> 问题

我本机的mysql数据库里面有个名为piwik的“schema”，里面有张表piwik_log_action，我想通过MyBatis Generator（以下简称为MBG）生成这张表的entity、mapper等文件。

但是执行MBG后，发现mapper里没有selectByPrimaryKey、updateByPrimaryKey等与主键相关的方法，而表里面是有主键的。

后来发现，执行MBG的时候控制台出现了这样的警告信息：Cannot obtain primary key information from the database, generated objects may be incomplete。意思是无法获取数据库的主键信息，那么当然就无法生成byPrimaryKey的方法了。

下面是我的generatorConfig.xml配置信息。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="winston" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/piwik?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC"
                        userId="root"
                        password="123456">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <javaModelGenerator targetPackage="com.winston.stone.ua.entity" targetProject="service-ua/src/main/java">
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <sqlMapGenerator targetPackage="src.main.resources.mybatis.mapper" targetProject="service-ua">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <javaClientGenerator type="XMLMAPPER" targetPackage="com.winston.stone.ua.repository"
                             targetProject="service-ua/src/main/java">
        </javaClientGenerator>

        <table schema="piwik" tableName="piwik_log_action" domainObjectName="Action"></table>

    </context>
</generatorConfiguration>
```

> 定位

考虑到我之前使用MBG1.3.2以及mysq驱动5.1.42，生成的mapper是没问题的，所以问题出现的原因就锁定在版本上了。OK，我轮番切换版本号，最后发现mysql驱动为5.1.42版本的时候，MBG不管是新旧版本都木有问题。反之，mysql驱动为6.0.6时，生成的mapper都是没有byPrimaryKey方法的。那么显然问题就出现在mysq驱动的版本上。

OK，那么驱动用旧版本的就行了。Nonono，对于敢于尝试新事物的我，当然不会屈就于用旧版本的驱动来规避这个问题啦。

那么，开启源码调试模式。

> 调试

首先，我先使用6.0.6的驱动，然后使用java形式的来执行MBG，代码如下

```
public class MybatisGeneratorExecutor {
    public static void main(String[] args) throws IOException, XMLParserException, InvalidConfigurationException, SQLException, InterruptedException, URISyntaxException {
        List<String> warnings = new ArrayList<>();
        boolean overwrite = true;

        File configFile = new File(MybatisGeneratorExecutor.class.getResource("generatorConfig.xml").toURI());

        Configuration config = new ConfigurationParser(warnings).parseConfiguration(configFile);

        DefaultShellCallback callback = new DefaultShellCallback(overwrite);

        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);

        warnings.forEach(System.out::println);
    }
}
```

这里，我注意到变量warnings是用来接收MBG执行时的警告信息，所以说只要找到了给这个变量添加了上述“找不到主键的警告信息”的代码，那么问题将迎刃而解。

果然，当执行到org.mybatis.generator.api.MyBatisGenerator的257行时。

```
context.introspectTables(callback, warnings, fullyQualifiedTableNames);
```

warnings就被赋值了，如下图所示。

![img](https://upload-images.jianshu.io/upload_images/5858794-0a8801be8cb7efa5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/963/format/webp)

image.png

OK，继续深追，看看具体在哪行代码赋值的。一直dig一直dig，经过一番微操后，终于发现在执行org.mybatis.generator.internal.db.DatabaseIntrospector的calculatePrimaryKey方法时，出现了异常SQLException

![img](https://upload-images.jianshu.io/upload_images/5858794-04ba6be564ae1a61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

image.png

异常的描述为Unknown table 'piwik_log_action' in information_schema，其中information_schema是数据库中的第一个“schema”，当时我感到很奇怪，为什么会在information_schema上面找这张表了，这张表我已经在generatorConfig.xml里配置了“schema”为piwik的。

```
<table schema="piwik" tableName="piwik_log_action" domainObjectName="Action"></table>
```

让我们在深入调查下calculatePrimaryKey方法究竟发生了什么事情。再深入调试，进入com.mysql.cj.jdbc.DatabaseMetaData的getPrimaryKeys方法，在这里，我发现异常就出现在执行代码`rs = stmt.executeQuery(queryBuf.toString());`,其中queryBuf.toString()的值为
`SHOW KEYS FROM `piwik_log_action` FROM `information_schema``，显然information_schema是没有那张表的，所以就出现sql异常了。

![img](https://upload-images.jianshu.io/upload_images/5858794-7b9f2cddf412c122.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

image.png

我看到他是迭代getCatalogIterator(catalog)的返回值catalogStr就是information_schema，那么这个值是怎么来的，再看看getCatalogIterator方法吧。

![img](https://upload-images.jianshu.io/upload_images/5858794-05f9a0e71f7b573d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/874/format/webp)

image.png

调试后发现进入的是最后一个分支，而这里就得到了数据库中所有的“schema”，而刚好我的数据库中的第一个“schema”就是information_schema，所以在遍历的时候，第一个取得就是他。

![img](https://upload-images.jianshu.io/upload_images/5858794-afde4e284476a203.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

image.png

OK，到这里我就算找到了6.0.6版本的问题出在哪，但是还不知道怎么解决，这时我就想到与5.1.42版本的做下对比，看看旧版本为啥没问题。
将mysql驱动改为5.1.42版本，再次进入源码调试模式。

直接进入com.mysql.jdbc.DatabaseMetaData，发现他进入的是第二个分支，然后获取的值刚好是“piwik”。

![img](https://upload-images.jianshu.io/upload_images/5858794-5b79b01e1b9e79e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

image.png

看来问题出在nullCatalogMeansCurrent，虽然两个版本的获取方式不同，但这个东东其实是连接mysql驱动的属性，在5.1.42中默认是true，而在6.0.6默认为false。

那这样的话，在连接串中加入nullCatalogMeansCurrent=true就行了。

```
jdbc:mysql://localhost:3306/piwik?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC&nullCatalogMeansCurrent=true
```

果然，加上后警告提示没了，byPrimaryKey的方法也都有了。Oh yeah!

OK，到这里，问题已经解决了。Nonono，问题似乎并没有这么simple，怎么说呢，因为机智的我发现里面还有玄机，来来来，请看下文分解。

在调试的过程中，看到过msyql驱动的源码里有这样的代码注释--“No schemas in MySQL”，不仅是6的版本，5的版本也是这么描述的。

![img](https://upload-images.jianshu.io/upload_images/5858794-ad4b0ee021cf14bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/904/format/webp)

image.png

看到这个我就开始觉得不对劲了，再加上前面的nullCatalogMeansCurrent属性，我铁定的怀疑我的配置出问题了。

前面给大家看了我在generatorConfig.xml里的配置

```
<table schema="piwik" tableName="piwik_log_action" domainObjectName="Action"></table>
```

有个属性就是配置的schema，那mysql没有schema的话，那么这咋整，机智的我跑去MBG官网看了下<table>的配置，发现除了schema属性外，还有个catalog属性。

![img](https://upload-images.jianshu.io/upload_images/5858794-68ce43cf282d4bcc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

image.png

此时我把table的配置schema改成了catalog。然后把连接串的nullCatalogMeansCurrent=true去掉。执行generator后发现还果真可以正常生成byPrimaryKey的方法。

这时我才知道，原来使用mysql驱动连接mysql时，指定数据库是用catalog，而不是schema。

那nullCatalogMeansCurrent就好理解了，因为之前配置的是schema="piwik"，那么catalog就是null了，null的话就means current，就取当前的catalog，这个catalog就是连接串`jdbc:mysql://localhost:3306/piwik`里的“piwik”。

OK，既然<table>里已经配置了catalog，那么连接串就没必要指定catalog了。

最终形态

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <context id="winston" targetRuntime="MyBatis3">
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC"
                        userId="root"
                        password="123456">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <javaModelGenerator targetPackage="com.winston.stone.ua.domain" targetProject="service-ua/src/main/java">
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <sqlMapGenerator targetPackage="src.main.resources.mybatis.mapper" targetProject="service-ua">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <javaClientGenerator type="XMLMAPPER" targetPackage="com.winston.stone.ua.repository"
                             targetProject="service-ua/src/main/java">
        </javaClientGenerator>

        <table catalog="piwik" tableName="piwik_log_action" domainObjectName="Action"></table>

    </context>
</generatorConfiguration>
```

> 总结

自己遇到的问题能够独立解决固然是好事，但是，如果深入问题的本质，就会发觉自己的不足。如果我能够更好的了解Mysql，根本就不会出现这样的问题。如果我能更好的了解MBG的配置，在配置有误后也能快速解决问题。所以说，要对自己的将要使用的武器有足够的了解，否则不仅无法发挥它们的最大作用，还有可能自损八百，得不偿失。

然后，我还是有疑问，就是catalog和schema在mysql里面究竟是怎么个情况，我在官网没有找到相关资料，网上有些人说schema就是database，也有人说database==schema==catalog，但是，按sql标准，database包含catalog，catalog包含schema。实在是搞不懂，摸不透。

难道mysql真的没有schema吗？有没有大神解答下呀。