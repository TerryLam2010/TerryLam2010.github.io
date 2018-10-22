# java 查看jar包加载顺序

### 查看具体的class从哪个jar包中加载的

**看源码时可以观察class加载顺序。**

在JVM启动时，加上如下参数：

```
-verbose:class
```

然后哦就会有如下输出

[Loaded java.util.regex.Pattern from /Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home/jre/lib/rt.jar]



拓展：

**java –verbose:gc**

在虚拟机发生内存回收时在输出设备显示信息，格式如下： [Full GC 256K->160K(124096K), 0.0042708 secs] 该参数用来监视虚拟机内存回收的情况。





**java –verbose:jni**

-verbose:jni输出native方法调用的相关情况，一般用于诊断jni调用错误信息。

在虚拟机调用native方法时输出设备显示信息，格式如下： [Dynamic-linking native method java.lang.Object.registerNatives ... JNI] 该参数用来监视虚拟机调用本地方法的情况，在发生jni错误时可为诊断提供便利。