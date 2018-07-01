# 【转】TCC型分布式事务原理和实现之：TransactionManager

# 前言

​      目前，还没有一款商用成熟的开源TCC框架，所以很多人基于不同思想实现了TCC的很多版本，本文所分析的TCC框架，是笔者在读了一些TCC开源代码、JTA设计思想之后逐步修改而来，由于代码还在逐步优化中，但是大体已经成型，所以将TCC框架的一点设计思想分享出来。

# 核心类图

​           ![img](https://static.oschina.net/uploads/space/2017/0515/223326_jF8f_2896894.png) 

事务管理器（TransactionManager）作为TCC分布式事务的核心组件，负责事务的管理工作（包括事务的提交和回滚）。 每一个事务的参与者（Participant）中都会有一个单例的事务管理器负责本地事务的管理工作。在TCC的try阶段，根事务（ROOT，表示事务的发起方）参与者中的事务管理器会负责创建根事务并持久化事务日志，同时会将创建的事务保存在ThreadLocal类型的队列中，处于分支事务参与者中的事务管理器会直接从事务上下文中传播（propagation）一个新的分支事务，同时也会持久化事务日志并将事务加入ThreadLocal队列中。在根事务端，try阶段结束后，TCC框架会根据try阶段是否有异常分别自动调用根事务管理器的commit和rollback方法，以commit为例，根事务管理器的commit又会调用根事务（transaction）的commit方法，根事务（transaction）的commit方法会遍历所有的事务参与者（Participant）的commit方法，这里的参与者一共有两种，分别为根事务参与者和分支事务参与者，这里的所谓的“根”和“分支”都只是逻辑上角色的区分，表示主动发起和被动参与的区别，它们在代码层面完全是共用一套逻辑的，比如，一个分支事务同样可以发起根事务，这样就是典型的嵌套型事务了。但是，整个事务的提交和回滚一定是由最顶层事务管理器发起的，其他的分支事务（包括多层嵌套事务）都是在最顶层根事务管理器的协调下完成自身的提交和回滚。至此，一个完整的分布式事务就结束了，当然，这只是一个大概的过程描述，还有很多细节没有提及，主要是目前的讨论还没有限制SOA框架，所以无法谈论具体的远程事务细节，等到后面专门讨论远程事务的文章（SOA为dubbo）时，会详细讨论根事务参与者和分支事务参与者。              

# 事务管理器

​      事务管理器的属性只有transactionRepository和CURRENT，其中transactionRepository用于持久化事务日志，CURRENT用于保存该事务管理器上活动的事务，它是一个ThreadLocal队列。          

​                                      ![img](https://static.oschina.net/uploads/space/2017/0516/095407_hq2a_2896894.png)

​      事务管理器具有较多的方法，下面分表分析：

## begin()

​    begin()表示开始一个事务，会在根事务（ROOT）的try方法中被调用。       

```
    public Transaction begin() {
        // 创建事务，事务类型为根事务ROOT
        Transaction transaction = new Transaction(TransactionType.ROOT);
        // 事务日志持久化
        transactionRepository.create(transaction);
        // 注册事务，就是加到线程局部变量的队列中
        registerTransaction(transaction);
        return transaction;
    }
```

## registerTransaction()

```
    private void registerTransaction(Transaction transaction) {
        // 如果队列还没有创建就先创建一个
        if (CURRENT.get() == null) {
            CURRENT.set(new LinkedList<Transaction>());
        }
        // 加入事务队列
        CURRENT.get().push(transaction);
    }
```

## propagationNewBegin()

​     propagationNewBegin用于从一个事务上下文中传播一个新事务，通常会在分支事务（比如dubbo中的provider端）的try阶段被调用，此处的事务上下文可以使用方法传参，也可以使用特定SOA框架的隐式传参（比如dubbo）。

```
 public Transaction propagationNewBegin(TransactionContext transactionContext) {
        // 从transactionContext创建一个事务
        Transaction transaction = new Transaction(transactionContext);
        // 事务日志持久化
        transactionRepository.create(transaction);
        // 注册事务到事务管理器
        registerTransaction(transaction);
        return transaction;
    }
```

## propagationExistBegin()

​      propagationExistBegin用于从事务上下文中传播一个已存在的事务，通常会在分支事务（比如dubbo中的provider端）的confirm阶段和cancel阶段被调用，此处的事务上下文可以使用方法传参，也可以使用特定SOA框架的隐式传参（比如dubbo）。

```
 public Transaction propagationExistBegin(TransactionContext transactionContext) throws NoExistedTransactionException {
        // 从持久化日志中根据事务id拿到事务
        Transaction transaction = transactionRepository.findByXid(transactionContext.getXid());
        // 如果找到了事物
        if (transaction != null) {
            // 更改事务状态为transactionContext中的状态
            transaction.changeStatus(TransactionStatus.valueOf(transactionContext.getStatus()));
            // 注册事务
            registerTransaction(transaction);
            return transaction;
        } else {
            throw new NoExistedTransactionException();
        }
    }
```

## commit()

​      commit会在事务try阶段没有异常的情况下，由TCC框架自动调用。它首先从ThreadLocal队列中取出当前要处理的事务（但不从队列中删除这个事务），然后将事务状态改为CONFIRMING状态，更新事务日志。随后调用事务的commit方法进行事务提交处理，如果事务提交成功（没有抛出任何异常），那么就从事务日志仓库中删除这个事务日志。如果在事务commit过程中抛出了异常，那么这个事物日志此时不会被删除（稍后会被recovery任务处理），同时，框架会将异常全部转为ConfirmingException向业务层抛出。

```
 public void commit() {
        // 获取本地线程上事务队列中的时间最久的事务
        Transaction transaction = getCurrentTransaction();
        if(transaction == null){
            return;
        }
        // 更改事务状态为CONFIRMING
        transaction.changeStatus(TransactionStatus.CONFIRMING);
        // 更新事务持久化
        transactionRepository.update(transaction);

        try {
            // 调用事务的commit
            transaction.commit();
            // 如果上面的commit没有抛出任何异常就说明事务成功，就从事务日志中删除这个事务
            transactionRepository.delete(transaction);
        } catch (Throwable commitException) {
            // 事务commit过程抛出了异常
            logger.error("compensable transaction confirm failed.", commitException);
            // 转为抛出ConfirmingException异常，这样会导致事务在事务日志中不被删除，recovery会去处理长时间没有被删除的事务
            throw new ConfirmingException(commitException);
        }
    }
```

## getCurrentTransaction()

​       获取当前要处理的事务，此处要注意的是，队列的peek只是取出队列头部元素，但是不会将其删除。

```
    public Transaction getCurrentTransaction() {
        if (isTransactionActive()) {
            // 拿到队列头的事务（但是不从队列中删除，删除是在cleanAfterCompletion中进行）
            return CURRENT.get().peek();
        }
        return null;
    }
```

## isTransactionActive()

​       判断当前事务管理中是否还有活动的事务。

```
   public boolean isTransactionActive() {
        Deque<Transaction> transactions = CURRENT.get();
        return transactions != null && !transactions.isEmpty();
    }
```

## rollback()

​      rollback和commit相对，当try阶段抛出了任何异常，TCC框架会自动调用。它首先从事务管理器中取出当前活动的事务，更改事务状态为CANCELLING，并更新事务日志。然后调用事务的rollback进行事务回滚（事务的rollback会遍历所有参与者，并分别调用参与者的rollback，通常，根事务端的参与者包含根事务参与者和分支事务参与者，而分支事务参与者通常只有一个本地的事务参与者，除非它也发起了TCC分布式事务）。如果rollback成功，事务会被从事务日志持久化仓库中删除，否则直接向业务层代码抛出CancellingException异常，残留的事务日志稍后会被recovery任务处理（可选、可配置）。

```
public void rollback() {
        // 回滚事务
        Transaction transaction = getCurrentTransaction();
        // 更改事务状态为CANCELLING
        transaction.changeStatus(TransactionStatus.CANCELLING);
        // 更新事务持久化日志
        transactionRepository.update(transaction);

        try {
            // 调用事务的rollback
            transaction.rollback();
            // 没有异常，就从事务日志中删除这个事务
            transactionRepository.delete(transaction);
        } catch (Throwable rollbackException) {
            logger.error("compensable transaction rollback failed.", rollbackException);
            // 否则事务异常，抛出CancellingException
            throw new CancellingException(rollbackException);
        }
    }
```

## cleanAfterCompletion()

​      还记得前文所说的，每次从事务管理器获取当前活动事务的时候，都不会从队列中将其删除，那么这些事务会在什么时候删除呢，这就是cleanAfterCompletion的作用。在每次事务处理结束时，TCC框架都会调用cleanAfterCompletion进行事务的清理操作。清理之前要比对要清理的事务是不是当前事务。

```
 public void cleanAfterCompletion(Transaction transaction) {
        if (isTransactionActive() && transaction != null) {
            Transaction currentTransaction = getCurrentTransaction();
            if (currentTransaction == transaction) {
                CURRENT.get().pop();
            } else {
                throw new SystemException("Illegal transaction when clean after completion");
            }
        }
    }
```

## enlistParticipant()

​      enlistParticipant用于向事务中添加一个事务参与者，同上，这里的参与者包含了本地参与者和远程参与者，添加参与者之后必须更新事务日志。enlistParticipant会在添加到TCC事务方法的切面中被调用。  

```
 public void enlistParticipant(Participant participant) {
        Transaction transaction = this.getCurrentTransaction();
        transaction.enlistParticipant(participant);//将参与者加入事务的参与者列表中
        transactionRepository.update(transaction);// 更新事务日志
    }
```

# 标记TCC事务方法      

​      本文多次提到了TCC事务方法和切面逻辑，那么什么是一个TCC事务方法呢？给一个方法添加标记肯定要使用注解了，TCC框架中，使用Compensable注解来表示一个方法是一个TCC事务方法，同时TCC框架针对标记了Compensable注解的方法提供了两个切面：CompensableTransactionAspect和ResourceCoordinatorAspect。其中CompensableTransactionAspect用于封装事务逻辑（事务开启、提交和回滚等），而ResourceCoordinatorAspect切面用于封装一个参与者并添加到事务中（上文提及）。由于还有专门的文章介绍这两个切面，所以本文暂时不作深入讨论。这里需要注意的是，这两个切面是有优先级排序的，CompensableTransactionAspect优先级高于ResourceCoordinatorAspect，这个只要实现spring框架中的Ordered接口就可以了。

​                        ![img](https://static.oschina.net/uploads/space/2017/0516/122612_5Tft_2896894.png)

来自：https://my.oschina.net/fileoptions/blog/900650