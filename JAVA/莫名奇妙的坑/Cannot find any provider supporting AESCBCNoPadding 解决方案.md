# Cannot find any provider supporting AES/CBC/NoPadding 解决方案

最终找到的是因为我们一个选项造成的：-Djava.ext.dirs=
我们覆盖了这个选项，导致一些java的功能无法使用。
写上同事的当时的总结：

![img](https://upload-images.jianshu.io/upload_images/1737506-1e392b7d1b0f37e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/741/format/webp)



-Djava.ext.dirs=$PLATFORM_HOME/lib:$JAVA_HOME/jre/lib/ext  指定/jre/lib/ext目录启动