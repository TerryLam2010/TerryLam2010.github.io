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

--- 2018年2月12日记录 ---

---


