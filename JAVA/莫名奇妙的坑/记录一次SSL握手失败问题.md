记一次httpclient  ssl问题

# 一、前言

本篇风格会偏向讲故事，来记录整个发现问题，解决问题的过程。具体的知识点总结放在后一篇。

前段阵子被分配了一个工单，要求抓取另一个险企B的数据。想着应该不会比上一家A麻烦了，险企A抓取数据过程中有几次请求是跨域的，很多数据都是由ajax动态请求到的，要分析js代码，模拟请求。

稍微观察了一下险企B的页面源代码，发现所有操作除了表单提交，其他都是get请求。而且模拟登录时不需要输验证码。美滋滋。。就是有2点麻烦的地方：

- 险企B是通过专线访问的，只有借助代理公司的网络才能访问，公司在代理公司那放了台电脑，然后我在公司远程连接那台电脑进行开发的。操作会有延时，有点卡。
- 险企B的网站看起来很古老，只支持ie8及以下的浏览器访问，chrome、火狐啥的就更打不开了。所以抓包都靠fiddler了，页面解析元素定位就只能靠旧版本的ie开发工具，

好吧，虽然不便，但是还是不怎么影响开发过程。

然后在一开始，访问第一个登录页面的时候我就被卡住了。我用原来的工具类发了一个get请求去获取登录页面，结果报错了。

# 二、错误1

```
debug:
    Unsupported record version SSLv2Hello
    javax.net.ssl.SSLException: Unsupported record version SSLv2Hello
    at sun.security.ssl.InputRecord.readV3Record(Unknown Source)
    at sun.security.ssl.InputRecord.read(Unknown Source)
    at sun.security.ssl.SSLSocketImpl.readRecord(Unknown Source)
    at sun.security.ssl.SSLSocketImpl.performInitialHandshake(Unknown Source)
    at sun.security.ssl.SSLSocketImpl.startHandshake(Unknown Source)
    at sun.security.ssl.SSLSocketImpl.startHandshake(Unknown Source)
    at org.apache.http.conn.ssl.SSLConnectionSocketFactory.createLayeredSocket(SSLConnectionSocketFactory.java:275)
    at org.apache.http.conn.ssl.SSLConnectionSocketFactory.connectSocket(SSLConnectionSocketFactory.java:254)
    at org.apache.http.impl.conn.HttpClientConnectionOperator.connect(HttpClientConnectionOperator.java:123)
    at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.connect(PoolingHttpClientConnectionManager.java:318)
    at org.apache.http.impl.execchain.MainClientExec.establishRoute(MainClientExec.java:363)
    at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.java:219)
    at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java:195)
    at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:86)
    at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:108)
    at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:184)
    at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:82)
    at httpcomponents.httpsTest.main(httpsTest.java:135)
```

一脸懵逼。直觉上是遇到了什么麻烦的东西。直接去Stack Overflow上面搜索了。发现有个相同错误的问题。

[https://stackoverflow.com/questions/26166121/unsupported-record-version-sslv2hello-using-closeablehttpclient](https://link.jianshu.com?t=https%3A%2F%2Fstackoverflow.com%2Fquestions%2F26166121%2Funsupported-record-version-sslv2hello-using-closeablehttpclient)

里面的答案大致就是说，我所要请求的这个server很古老，居然还支持SSLv2协议（还用了个incredibly加强语气，-_-||）。

## 2.1 解决方案：

使用`SSLConnectionSocketFactory`来强制只允许使用`TLSv1`协议。我的代码如下：

```
// ssl context
SSLContext sslcontext = SSLContexts.custom().build();
//  ssl socket factory
SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
                sslcontext,
                new String[]{"TLSv1"},
                null,
                SSLConnectionSocketFactory.getDefaultHostnameVerifier());
// httClient 实例
CloseableHttpClient httpClient = HttpClients.custom()
                .setSSLSocketFactory(sslsf)
                //.setDefaultCookieStore(cookieStore)
                // 异常重试机制 3次 （网络层面上的）
                //.setRetryHandler(new DefaultHttpRequestRetryHandler(3,true))
                //.setDefaultRequestConfig(defaultRequestConfig)
                .build();
```

至于这些安全协议，在下一章会总结。

上述代码加进去之后呢。。之前那个错误是解决了。然后又出现了新的错误。

# 三、错误2

```
Exception in thread "main" javax.net.ssl.SSLHandshakeException: Remote host closed connection during handshake
       at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:946)
       at sun.security.ssl.SSLSocketImpl.performInitialHandshake(SSLSocketImpl.java:1312)
       at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1339)
       at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1323)
       at sun.net.www.protocol.https.HttpsClient.afterConnect(HttpsClient.java:563)
       at sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.connect(AbstractDelegateHttpsURLConnection.java:185)
       at sun.net.www.protocol.http.HttpURLConnection.getOutputStream(HttpURLConnection.java:1091)
       at sun.net.www.protocol.https.HttpsURLConnectionImpl.getOutputStream(HttpsURLConnectionImpl.java:250)
       at com.labcorp.efone.vendor.TestATTConnectivity.main(TestATTConnectivity.java:43)
Caused by: java.io.EOFException: SSL peer shut down incorrectly
       at sun.security.ssl.InputRecord.read(InputRecord.java:482)
       at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:927)
       ... 8 more
```

看样子好像是握手时候失败了。。又滚去Stack Overflow上去搜索了一下，发现这还是个很火的问题。大家出问题的原因都不一样，也没有个综合的答案。所以我就不放出来了，如果有兴趣可以自己去看看。

我的这个问题涉及到了 SSL/TLS 的握手和通信过程中，安全认证被分为单向认证和双向认证。这里面的知识点也很多，具体下一篇总结篇。双向认证就是说，server也会要求验证client的证书，而我用Java程序模拟时没有启用证书，所以导致认证阶段出错，握手失败。

## 3.1 解决方法

相关图片由于那时候忘了截图，我直接引用的是参考资料中的。

1、访问https网站，下载证书

```
a. 浏览器地址栏旁边会有一个锁的图标，点击那个锁，会有查看证书的按钮；
b. 将证书信息导出，证书格式有很多种，der、cer等，我保存的是cer格式的
```

2、利用jdk的toolkey工具，将证书转换成密钥的形式

命令行或者shell执行下列命令：

```
keytool -import -alias "my alipay cert" -file steven.cert     -keystore my.store,
```

![img](//upload-images.jianshu.io/upload_images/8514567-82536955772c69f3.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/672/format/webp)

image

之后还需要设置密码，我直接设置成`123456`。

3、sslContext中载入信用证书

```
    private static SSLContext sslcontext;
        try {
            sslcontext = SSLContexts.custom()
                    .loadTrustMaterial(new File("D:\\my.keystore"), "123456".toCharArray(),
                            new TrustSelfSignedStrategy())
                    .build();
        } catch (Exception e) {
            e.printStackTrace();
        }
        SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(
                sslcontext,
                new String[]{"TLSv1"},
                null,
                SSLConnectionSocketFactory.getDefaultHostnameVerifier());
        httpClient = HttpClients.custom()
                .setSSLSocketFactory(sslsf)
                .setDefaultCookieStore(cookieStore)
                // 异常重试机制 3次 （网络层面上的）
                .setRetryHandler(new DefaultHttpRequestRetryHandler(3,true))
                .setDefaultRequestConfig(defaultRequestConfig)
                .build();
```

然后就解决了。

## 3.2 SSLHandshake 阶段的另一种报错

Btw，`javax.net.ssl.SSLHandshakeException`还有一种常见的错误：

```
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed
```

这个就是服务端的证书是不可信的情况。你可以理解为当你用浏览器访问某个网站时，页面弹出该网站证书不可信的情况。在这里就是当然这种错误我是没有遇到。解决方法详见[http://zhuyuehua.iteye.com/blog/1102347](https://link.jianshu.com?t=http%3A%2F%2Fzhuyuehua.iteye.com%2Fblog%2F1102347)

# 访问https服务其他的常见错误

- java.net.ConnectException: Connection refused: connect  服务器没有启动
- java.net.SocketException: Software caused connection abort: recv failed
   这是由于服务端配置的是SSL双向认证，而客户端发送数据是按照服务器是单向认证时发送的，即没有将客户端证书信息一起发送给服务端。
- org.apache.commons.httpclient.NoHttpResponseException  这一般是服务端防火墙的原因。拦截了客户端请求。另外，当服务端负载过重时，也会出现此问题。将客户端证书信息一起发送给服务端。

# 参考资料

[1] [http://zhuyuehua.iteye.com/blog/1122670](https://link.jianshu.com?t=http%3A%2F%2Fzhuyuehua.iteye.com%2Fblog%2F1122670)

[2] [https://blog.csdn.net/wanglha/article/details/51140846](https://link.jianshu.com?t=https%3A%2F%2Fblog.csdn.net%2Fwanglha%2Farticle%2Fdetails%2F51140846)

[3] [https://hc.apache.org/httpcomponents-client-ga/httpclient/examples/org/apache/http/examples/client/ClientCustomSSL.java](https://link.jianshu.com?t=https%3A%2F%2Fhc.apache.org%2Fhttpcomponents-client-ga%2Fhttpclient%2Fexamples%2Forg%2Fapache%2Fhttp%2Fexamples%2Fclient%2FClientCustomSSL.java)

[4] [https://blog.csdn.net/liuxiao723846/article/details/52695549](https://link.jianshu.com?t=https%3A%2F%2Fblog.csdn.net%2Fliuxiao723846%2Farticle%2Fdetails%2F52695549)

 

 

 

 

 尤其重要

<https://hc.apache.org/httpcomponents-client-ga/httpclient/examples/org/apache/http/examples/client/ClientCustomSSL.java> 

<https://blog.csdn.net/liuxiao723846/article/details/52695549> 

<https://blog.csdn.net/tianxue04/article/details/98957670> 

```
	public static CloseableHttpClient createNewHttpsClient() throws Exception {
		try {
			SSLContext sslContext =SSLContexts.custom().loadTrustMaterial(null,	new TrustStrategy() {
				//信任所有
				public boolean isTrusted(X509Certificate[] chain,
										 String authType) throws CertificateException {
					return true;

				}}).build();
			SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext,new String[]{"TLSv1", "TLSv1.2","TLSv1.1"},null, (hostName, sslSession) -> {
				return true; // 证书校验通过
			});
			HttpClientBuilder builder = HttpClients.custom().setSSLSocketFactory(sslsf);
			return builder.build();
		} catch (Exception e) {
			logger.error("createHttpsClient error : ", e);
		}
		return  HttpClients.createDefault();
	}
```

