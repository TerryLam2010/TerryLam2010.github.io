# 下载的Jar包 Bad Signature

标签（空格分隔）： Maven Hibernate Spring

---

若是下载jar包，项目打了一个叹号，说明Jar包依赖有问题。
maven build的时候，产生Bad Signature就是下载的Jar有问题。
这次的错误是maven 配置了阿里云，可能有些Jar是不存在。我添加上中央仓库的配置，并且把同事的仓库拷来使用，就把这问题解决了。

	<repositories>  
    <repository>  
      <id>central</id>  
      <name>Central Repository</name>  
      <url>http://repo.maven.apache.org/maven2</url>  
      <layout>default</layout>  
      <snapshots>  
        <enabled>false</enabled>  
      </snapshots>  
    </repository>  
  </repositories>

![此处输入图片的描述][1]
--- 2018年2月12日记录 ---

---


  [1]: https://raw.githubusercontent.com/TerryLam2010/TerryLam2010.github.io/master/image/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%AE%BF%E9%97%AE%E6%85%A2%E5%8E%9F%E5%9B%A0/%E6%9C%89mq%E7%9A%84%E5%86%85%E5%AD%98%E5%8D%A0%E7%94%A8.png