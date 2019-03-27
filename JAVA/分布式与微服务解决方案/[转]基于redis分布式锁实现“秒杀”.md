# [转]基于redis分布式锁实现“秒杀”

最近在项目中遇到了类似“秒杀”的业务场景，在本篇博客中，我将用一个非常简单的demo，阐述实现所谓“秒杀”的基本思路。

# 业务场景

所谓秒杀，从业务角度看，是短时间内多个用户“争抢”资源，这里的资源在大部分秒杀场景里是商品；将业务抽象，技术角度看，秒杀就是多个线程对资源进行操作，所以实现秒杀，就必须控制线程对资源的争抢，既要保证高效并发，也要保证操作的正确。

# 一些可能的实现

刚才提到过，实现秒杀的关键点是控制线程对资源的争抢，根据基本的线程知识，可以不加思索的想到下面的一些方法： 
1、秒杀在技术层面的抽象应该就是一个方法，在这个方法里可能的操作是将商品库存-1，将商品加入用户的购物车等等，在不考虑缓存的情况下应该是要操作数据库的。那么最简单直接的实现就是在这个方法上加上`synchronized`关键字，通俗的讲就是锁住整个方法； 
2、锁住整个方法这个策略简单方便，但是似乎有点粗暴。可以稍微优化一下，只锁住秒杀的代码块，比如写数据库的部分； 
3、既然有并发问题，那我就让他“不并发”，将所有的线程用一个队列管理起来，使之变成串行操作，自然不会有并发问题。

上面所述的方法都是有效的，但是都不好。为什么？第一和第二种方法本质上是“加锁”，但是锁粒度依然比较高。什么意思？试想一下，如果两个线程同时执行秒杀方法，这两个线程操作的是不同的商品,从业务上讲应该是可以同时进行的，但是如果采用第一二种方法，这两个线程也会去争抢同一个锁，这其实是不必要的。第三种方法也没有解决上面说的问题。

那么如何将锁控制在更细的粒度上呢？可以考虑为每个商品设置一个互斥锁，以和商品ID相关的字符串为唯一标识，这样就可以做到只有争抢同一件商品的线程互斥，不会导致所有的线程互斥。分布式锁恰好可以帮助我们解决这个问题。

# 何为分布式锁

分布式锁是控制分布式系统之间同步访问共享资源的一种方式。在分布式系统中，常常需要协调他们的动作。如果不同的系统或是同一个系统的不同主机之间共享了一个或一组资源，那么访问这些资源的时候，往往需要互斥来防止彼此干扰来保证一致性，在这种情况下，便需要使用到分布式锁。

我们来假设一个最简单的秒杀场景：数据库里有一张表，column分别是商品ID，和商品ID对应的库存量，秒杀成功就将此商品库存量-1。现在假设有1000个线程来秒杀两件商品，500个线程秒杀第一个商品，500个线程秒杀第二个商品。我们来根据这个简单的业务场景来解释一下分布式锁。 
通常具有秒杀场景的业务系统都比较复杂，承载的业务量非常巨大，并发量也很高。这样的系统往往采用分布式的架构来均衡负载。那么这1000个并发就会是从不同的地方过来，商品库存就是共享的资源，也是这1000个并发争抢的资源，这个时候我们需要将并发互斥管理起来。这就是分布式锁的应用。 
而key-value存储系统，如redis，因为其一些特性，是实现分布式锁的重要工具。

# 具体的实现

先来看看一些redis的基本命令： 
`SETNX key value` 
如果key不存在，就设置key对应字符串value。在这种情况下，该命令和SET一样。当key已经存在时，就不做任何操作。SETNX是”SET if Not eXists”。 
`expire KEY seconds` 
设置key的过期时间。如果key已过期，将会被自动删除。 
`del KEY` 
删除key 
由于笔者的实现只用到这三个命令，就只介绍这三个命令，更多的命令以及redis的特性和使用，可以参考[redis官网](https://blog.csdn.net/u010359884/article/details/www.redis.io)。

## 需要考虑的问题

1、用什么操作redis？幸亏redis已经提供了jedis客户端用于java应用程序，直接调用jedis API即可。 
2、怎么实现加锁？“锁”其实是一个抽象的概念，将这个抽象概念变为具体的东西，就是一个存储在redis里的key-value对，key是于商品ID相关的字符串来唯一标识，value其实并不重要，因为只要这个唯一的key-value存在，就表示这个商品已经上锁。 
3、如何释放锁？既然key-value对存在就表示上锁，那么释放锁就自然是在redis里删除key-value对。 
4、阻塞还是非阻塞？笔者采用了阻塞式的实现，若线程发现已经上锁，会在特定时间内轮询锁。 
5、如何处理异常情况？比如一个线程把一个商品上了锁，但是由于各种原因，没有完成操作（在上面的业务场景里就是没有将库存-1写入数据库），自然没有释放锁，这个情况笔者加入了锁超时机制，利用redis的expire命令为key设置超时时长，过了超时时间redis就会将这个key自动删除，即强制释放锁（可以认为超时释放锁是一个异步操作，由redis完成，应用程序只需要根据系统特点设置超时时间即可）。

## talk is cheap,show me the code

在代码实现层面，注解有并发的方法和参数，通过动态代理获取注解的方法和参数，在代理中加锁，执行完被代理的方法后释放锁。

几个注解定义： 
cachelock是方法级的注解，用于注解会产生并发问题的方法:

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CacheLock {
    String lockedPrefix() default "";//redis 锁key的前缀
    long timeOut() default 2000;//轮询锁的时间
    int expireTime() default 1000;//key在redis里存在的时间，1000S
}12345678
```

`lockedObject`是参数级的注解，用于注解商品ID等基本类型的参数：

```
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LockedObject {
    //不需要值
}123456
```

`LockedComplexObject`也是参数级的注解，用于注解自定义类型的参数：

```
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LockedComplexObject {
    String field() default "";//含有成员变量的复杂对象中需要加锁的成员变量，如一个商品对象的商品ID

}1234567
```

`CacheLockInterceptor`实现`InvocationHandler`接口，在invoke方法中获取注解的方法和参数，在执行注解的方法前加锁，执行被注解的方法后释放锁：

```
public class CacheLockInterceptor implements InvocationHandler{
    public static int ERROR_COUNT  = 0;
    private Object proxied;

    public CacheLockInterceptor(Object proxied) {
        this.proxied = proxied;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        CacheLock cacheLock = method.getAnnotation(CacheLock.class);
        //没有cacheLock注解，pass
        if(null == cacheLock){
            System.out.println("no cacheLock annotation");          
            return method.invoke(proxied, args);
        }
        //获得方法中参数的注解
        Annotation[][] annotations = method.getParameterAnnotations();
        //根据获取到的参数注解和参数列表获得加锁的参数
        Object lockedObject = getLockedObject(annotations,args);
        String objectValue = lockedObject.toString();
        //新建一个锁
        RedisLock lock = new RedisLock(cacheLock.lockedPrefix(), objectValue);
        //加锁
        boolean result = lock.lock(cacheLock.timeOut(), cacheLock.expireTime());
        if(!result){//取锁失败
            ERROR_COUNT += 1;
            throw new CacheLockException("get lock fail");

        }
        try{
            //加锁成功，执行方法
            return method.invoke(proxied, args);
        }finally{
            lock.unlock();//释放锁
        }

    }
    /**
     * 
     * @param annotations
     * @param args
     * @return
     * @throws CacheLockException
     */
    private Object getLockedObject(Annotation[][] annotations,Object[] args) throws CacheLockException{
        if(null == args || args.length == 0){
            throw new CacheLockException("方法参数为空，没有被锁定的对象");
        }

        if(null == annotations || annotations.length == 0){
            throw new CacheLockException("没有被注解的参数");
        }
        //不支持多个参数加锁，只支持第一个注解为lockedObject或者lockedComplexObject的参数
        int index = -1;//标记参数的位置指针
        for(int i = 0;i < annotations.length;i++){
            for(int j = 0;j < annotations[i].length;j++){
                if(annotations[i][j] instanceof LockedComplexObject){//注解为LockedComplexObject
                    index = i;
                    try {
                        return args[i].getClass().getField(((LockedComplexObject)annotations[i][j]).field());
                    } catch (NoSuchFieldException | SecurityException e) {
                        throw new CacheLockException("注解对象中没有该属性" + ((LockedComplexObject)annotations[i][j]).field());
                    }
                }

                if(annotations[i][j] instanceof LockedObject){
                    index = i;
                    break;
                }
            }
            //找到第一个后直接break，不支持多参数加锁
            if(index != -1){
                break;
            }
        }

        if(index == -1){
            throw new CacheLockException("请指定被锁定参数");
        }

        return args[index];
    }
}12345678910111213141516171819202122232425262728293031323334353637383940414243444546474849505152535455565758596061626364656667686970717273747576777879808182838485
```

最关键的RedisLock类中的lock方法和unlock方法：

```
/**
     * 加锁
     * 使用方式为：
     * lock();
     * try{
     *    executeMethod();
     * }finally{
     *   unlock();
     * }
     * @param timeout timeout的时间范围内轮询锁
     * @param expire 设置锁超时时间
     * @return 成功 or 失败
     */
    public boolean lock(long timeout,int expire){
        long nanoTime = System.nanoTime();
        timeout *= MILLI_NANO_TIME;
        try {
            //在timeout的时间范围内不断轮询锁
            while (System.nanoTime() - nanoTime < timeout) {
                //锁不存在的话，设置锁并设置锁过期时间，即加锁
                if (this.redisClient.setnx(this.key, LOCKED) == 1) {
                    this.redisClient.expire(key, expire);//设置锁过期时间是为了在没有释放
                    //锁的情况下锁过期后消失，不会造成永久阻塞
                    this.lock = true;
                    return this.lock;
                }
                System.out.println("出现锁等待");
                //短暂休眠，避免可能的活锁
                Thread.sleep(3, RANDOM.nextInt(30));
            } 
        } catch (Exception e) {
            throw new RuntimeException("locking error",e);
        }
        return false;
    }

    public  void unlock() {
        try {
            if(this.lock){
                redisClient.delKey(key);//直接删除
            }
        } catch (Throwable e) {

        }
    }123456789101112131415161718192021222324252627282930313233343536373839404142434445
```

上述的代码是框架性的代码，现在来讲解如何使用上面的简单框架来写一个秒杀函数。 
先定义一个接口，接口里定义了一个秒杀方法：

```
public interface SeckillInterface {
/**
*现在暂时只支持在接口方法上注解
*/
    //cacheLock注解可能产生并发的方法
    @CacheLock(lockedPrefix="TEST_PREFIX")
    public void secKill(String userID,@LockedObject Long commidityID);//最简单的秒杀方法，参数是用户ID和商品ID。可能有多个线程争抢一个商品，所以商品ID加上LockedObject注解
}12345678
```

上述`SeckillInterface`接口的实现类，即秒杀的具体实现：

```
public class SecKillImpl implements SeckillInterface{
    static Map<Long, Long> inventory ;
    static{
        inventory = new HashMap<>();
        inventory.put(10000001L, 10000l);
        inventory.put(10000002L, 10000l);
    }

    @Override
    public void secKill(String arg1, Long arg2) {
        //最简单的秒杀，这里仅作为demo示例
        reduceInventory(arg2);
    }
    //模拟秒杀操作，姑且认为一个秒杀就是将库存减一，实际情景要复杂的多
    public Long reduceInventory(Long commodityId){
        inventory.put(commodityId,inventory.get(commodityId) - 1);
        return inventory.get(commodityId);
    }

}1234567891011121314151617181920
```

模拟秒杀场景，1000个线程来争抢两个商品：

```
@Test
    public void testSecKill(){
        int threadCount = 1000;
        int splitPoint = 500;
        CountDownLatch endCount = new CountDownLatch(threadCount);
        CountDownLatch beginCount = new CountDownLatch(1);
        SecKillImpl testClass = new SecKillImpl();

        Thread[] threads = new Thread[threadCount];
        //起500个线程，秒杀第一个商品
        for(int i= 0;i < splitPoint;i++){
            threads[i] = new Thread(new  Runnable() {
                public void run() {
                    try {
                        //等待在一个信号量上，挂起
                        beginCount.await();
                        //用动态代理的方式调用secKill方法
                        SeckillInterface proxy = (SeckillInterface) Proxy.newProxyInstance(SeckillInterface.class.getClassLoader(), 
                            new Class[]{SeckillInterface.class}, new CacheLockInterceptor(testClass));
                        proxy.secKill("test", commidityId1);
                        endCount.countDown();
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
            });
            threads[i].start();

        }
        //再起500个线程，秒杀第二件商品
        for(int i= splitPoint;i < threadCount;i++){
            threads[i] = new Thread(new  Runnable() {
                public void run() {
                    try {
                        //等待在一个信号量上，挂起
                        beginCount.await();
                        //用动态代理的方式调用secKill方法
                        SeckillInterface proxy = (SeckillInterface) Proxy.newProxyInstance(SeckillInterface.class.getClassLoader(), 
                            new Class[]{SeckillInterface.class}, new CacheLockInterceptor(testClass));
                        proxy.secKill("test", commidityId2);
                        //testClass.testFunc("test", 10000001L);
                        endCount.countDown();
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }
                }
            });
            threads[i].start();

        }


        long startTime = System.currentTimeMillis();
        //主线程释放开始信号量，并等待结束信号量，这样做保证1000个线程做到完全同时执行，保证测试的正确性
        beginCount.countDown();

        try {
            //主线程等待结束信号量
            endCount.await();
            //观察秒杀结果是否正确
            System.out.println(SecKillImpl.inventory.get(commidityId1));
            System.out.println(SecKillImpl.inventory.get(commidityId2));
            System.out.println("error count" + CacheLockInterceptor.ERROR_COUNT);
            System.out.println("total cost " + (System.currentTimeMillis() - startTime));
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }1234567891011121314151617181920212223242526272829303132333435363738394041424344454647484950515253545556575859606162636465666768697071
```

在正确的预想下，应该每个商品的库存都减少了500，在多次试验后，实际情况符合预想。如果不采用锁机制，会出现库存减少499，498的情况。 
这里采用了动态代理的方法，利用注解和反射机制得到分布式锁ID，进行加锁和释放锁操作。当然也可以直接在方法进行这些操作，采用动态代理也是为了能够将锁操作代码集中在代理中，便于维护。 
通常秒杀场景发生在web项目中，可以考虑利用spring的AOP特性将锁操作代码置于切面中，当然AOP本质上也是动态代理。

# 小结

这篇文章从业务场景出发，从抽象到实现阐述了如何利用redis实现分布式锁，完成简单的秒杀功能，也记录了笔者思考的过程，希望能给阅读到本篇文章的人一些启发。

源码仓库：<https://github.com/lsfire/redisframework>