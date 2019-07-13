# 如何实现接口日志AOP

按照我的习惯，喜欢在控制台看到请求的一切参数和响应的内容，万物皆不能脱离劳资的掌控，我控制欲就是这么强！！

一个类即可完事

```
@Aspect
@Configuration
public class ApiLogAspect {

    private static final Logger logger = LoggerFactory.getLogger("apiAccessLogger");

    @Pointcut("execution(* org.tl.blog.api.controller.*Controller.*(..))")
    public void excudeService() {
    }

    @Around("excudeService()")
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        RequestAttributes ra = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes sra = (ServletRequestAttributes) ra;
        HttpServletRequest request = sra.getRequest();


        StringBuffer logSb = new StringBuffer();
        String method = request.getMethod();  //请求method
        String requestURI = request.getRequestURI();   //uri
        String queryString = request.getQueryString();  //请求参数，get请求才有
        String contentType = request.getContentType();  
        int contentLength = request.getContentLength();
        String paraString = JSON.toJSONString(request.getParameterMap());
        FrontUser curUser = UserUtil.getCurUser();
        logSb.append("request url : ").append(requestURI).append(" | ")
                .append("queryString : ").append(queryString).append(" | ")
                .append("method : ").append(method).append(" | ")
                .append("contentType : ").append(contentType).append(" | ")
                .append("contentLength : ").append(contentLength).append(" | ")
                .append("nowTime : ").append(DateUtils.formatDate(new Date(), "yyyy-MM-dd HH:mm:ss")).append(" | ");
        if ("application/x-www-form-urlencoded".equals(contentType)) {
            logSb.append(" params : ").append(paraString).append(" | ");
        }
        if ("application/json".equals(contentType)) {
            logSb.append(" params : ");
            logSb.append(JSON.toJSONString( pjp.getArgs())).append(" | ");
        }
        if(curUser != null){
            logSb.append(" curUserId : ").append(curUser.getUserId()).append(" | ");
        }
        Object result = pjp.proceed();
        logger.info(logSb.toString());
        logger.info("reponse result :" + JSON.toJSONString(result));
        return result;
    }
}
```

log4j.properties

不要一堆日志都打在同一个文件里面，看不清的。分开来。以下是配置

```
log4j.logger.apiAccessLogger=INFO,apiAccessAppender
log4j.appender.apiAccessAppender=org.apache.log4j.DailyRollingFileAppender
log4j.appender.apiAccessAppender.Threshold = INFO
log4j.appender.apiAccessAppender.Append=true
log4j.appender.apiAccessAppender.File = logs/apiAccess.log
log4j.appender.apiAccessAppender.DatePattern = '.'yyyy-MM-dd-HH
log4j.appender.apiAccessAppender.layout = org.apache.log4j.PatternLayout
log4j.appender.apiAccessAppender.layout.ConversionPattern =%d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5p %c %x - %m%n
```

完事！



last   MyBlog: blog.terrylam.cn