# 【转】TCC和两阶段分布式事务处理的区别

一个TCC事务框架需要解决的当然是分布式事务的管理。关于TCC事务机制的介绍，可以参考[TCC事务机制简介](http://www.bytesoft.org/tcc-intro)。
TCC事务模型虽然说起来简单，然而要基于TCC实现一个通用的分布式事务框架，却比它看上去要复杂的多，不只是简单的调用一下Confirm/Cancel业务就可以了的。

本文将以Spring容器为例，试图分析一下，实现一个通用的TCC分布式事务框架需要注意的一些问题。

#### 一、TCC全局事务必须基于RM本地事务来实现全局事务

TCC服务是由Try/Confirm/Cancel业务构成的，
其Try/Confirm/Cancel业务在执行时，会访问资源管理器（Resource Manager，下文简称RM）来存取数据。这些存取操作，必须要参与RM本地事务，以使其更改的数据要么都commit，要么都rollback。

这一点不难理解，考虑一下如下场景：

![image](http://www.bytesoft.org/images/how-to-impl-tcc/tcc-intro.png)

 

 

反之，基于RM本地事务的TCC事务，这种情况则会很容易处理：[B:Try]操作中途执行失败，TCC事务框架将其参与RM本地事务直接rollback即可。后续TCC事务框架决定回滚全局事务时，在知道“[B:Try]操作涉及的RM本地事务已经rollback”的情况下，根本无需执行[B:Cancel]操作。

换句话说，基于RM本地事务实现TCC事务框架时，一个TCC型服务的cancel业务要么执行，要么不执行，不需要考虑部分执行的情况。

#### 二、TCC事务框架应该接管Spring容器的TransactionManager

基于RM本地事务的TCC事务框架，可以将各Try/Confirm/Cancel业务看着一个原子服务：一个RM本地事务提交，参与该RM本地事务的所有Try/Confirm/Cancel业务操作都生效；反之，则都不生效。掌握每个RM本地事务的状态以及它们与Try/Confirm/Cancel业务方法之间的对应关系，以此为基础，TCC事务框架才能有效的构建TCC全局事务。

TCC服务的Try/Confirm/Cancel业务方法在RM上的数据存取操作，其RM本地事务是由Spring容器的PlatformTransactionManager来commit/rollback的，TCC事务框架想要了解RM本地事务的状态，只能通过接管Spring的事务管理器功能。

2.1. 为什么TCC事务框架需要掌握RM本地事务的状态？
首先，根据TCC机制的定义，TCC事务是通过执行Cancel业务来达到回滚效果的。仔细分析一下，这里暗含一个事实：
只有生效的Try业务操作才需要执行对应的Cancel业务操作。换句话说，只有Try业务操作所参与的RM本地事务被commit了，后续TCC全局事务回滚时才需要执行其对应的Cancel业务操作；否则，如果Try业务操作所参与的RM本地事务被rollback了，后续TCC全局事务回滚时就不能执行其Cancel业务，此时若盲目执行Cancel业务反而会导致数据不一致。

其次，Confirm/Cancel业务操作必须保证生效。Confirm/Cancel业务操作也会涉及RM数据存取操作，其参与的RM本地事务也必须被commit。TCC事务框架需要在确切的知道所有Confirm/Cancel业务操作参与的RM本地事务都被成功commit后，才能将标记该TCC全局事务为完成。如果TCC事务框架误判了Confirm/Cancel业务参与RM本地事务的状态，就会造成全局事务不一致。

最后，未完成的TCC全局，TCC事务框架必须重新尝试提交/回滚操作。重试时会再次调用各TCC服务的Confirm/Cancel业务操作。如果某个服务的Confirm/Cancel业务之前已经生效（其参与的RM本地事务已经提交），重试时就不应该再次被调用。否则，其Confirm/Cancel业务被多次调用，就会有“服务幂等性”的问题。

2.2. 拦截TCC服务的Try/Confirm/Cancel业务方法的执行，根据其异常信息可否知道其RM本地事务是否commit/rollback了呢？
基本上很难做到。为什么这么说呢？
第一，事务是可以在多个（本地/远程）服务之间互相传播其事务上下文的，一个业务方法（Try/Confirm/Cancel）执行完毕并不一定会触发当前事务的commit/rollback操作。比如，被传播事务上下文的业务方法，在它开始执行时，容器并不会为其创建新的事务，而是它的调用方参与的事务，使得二者操作在同一个事务中；同样，在它执行完毕时，容器也不会提交/回滚它参与的事务的。因此，这类业务方法上的异常情况并不能反映他们是否生效。不接管Spring的TransactionManager，就无法了解事务于何时被创建，也无法了解它于何时被提交/回滚。
第二、一个业务方法可能会包含多个RM本地事务的情况。比如： A(REQUIRED)->B(REQUIRES_NEW)->C(REQUIRED)，这种情况下，A服务所参与的RM本地事务被提交时，B服务和C服务参与的RM本地事务则可能会被回滚。
第三、并不是抛出了异常的业务方法，其参与的事务就回滚了。Spring容器的声明式事务定义了两类异常，其事务完成方向都不一样：系统异常（一般为Unchecked异常，默认事务完成方向是rollback）、应用异常（一般为Checked异常，默认事务完成方向是commit）。二者的事务完成方向又可以通过@Transactional配置显式的指定，如rollbackFor/noRollbackFor等。
第四、Spring容器还支持使用setRollbackOnly的方式显式的控制事务完成方向；
最后、自行拦截业务方法的拦截器和Spring的事务处理的拦截器还会存在执行先后、拦截范围不同等问题。例如，如果自行拦截器执行在前，就会出现业务方法虽然已经执行完毕但此时其参与的RM本地事务还没有commit/rollback。

TCC事务框架的定位应该是一个TransactionManager，其职责是负责commit/rollback事务。而一个事务应该commit、还是rollback，则应该是由Spring容器来决定的：Spring决定提交事务时，会调用TransactionManager来完成commit操作；Spring决定回滚事务时，会调用TransactionManager来完成rollback操作。

接管Spring容器的TransactionManager，TCC事务框架可以明确的得到Spring的事务性指令，并管理Spring容器中各服务的RM本地事务。否则，如果通过自行拦截的机制，则使得业务系统存在TCC事务处理、RM本地事务处理两套事务处理逻辑，二者互不通信，各行其是。这种情况下要协调TCC全局事务，基本上可以说是缘木求鱼，本地事务尚且无法管理，更何谈管理分布式事务？

#### 三、TCC事务框架应该具备故障恢复机制

一个TCC事务框架，若是没有故障恢复的保障，是不成其为分布式事务框架的。

分布式事务管理框架的职责，不是做出全局事务提交/回滚的指令，而是管理全局事务提交/回滚的过程。它需要能够协调多个RM资源、多个节点的分支事务，保证它们按全局事务的完成方向各自完成自己的分支事务。这一点，是不容易做到的。因为，实际应用中，会有各种故障出现，很多都会造成事务的中断，从而使得统一提交/回滚全局事务的目标不能达到，甚至出现”一部分分支事务已经提交，而另一部分分支事务则已回滚”的情况。比较常见的故障，比如：业务系统服务器宕机、重启；数据库服务器宕机、重启；网络故障；断电等。这些故障可能单独发生，也可能会同时发生。作为分布式事务框架，应该具备相应的故障恢复机制，无视这些故障的影响是不负责任的做法。

一个完整的分布式事务框架，应该保障即使在最严苛的条件下也能保证全局事务的一致性，而不是只能在最理想的环境下才能提供这种保障。退一步说，如果能有所谓“理想的环境”，那也无需使用分布式事务了。

TCC事务框架要支持故障恢复，就必须记录相应的事务日志。事务日志是故障恢复的基础和前提，它记录了事务的各项数据。TCC事务框架做故障恢复时，可以根据事务日志的数据将中断的事务恢复至正确的状态，并在此基础上继续执行先前未完成的提交/回滚操作。

#### 四、TCC事务框架应该提供Confirm/Cancel服务的幂等性保障

一般认为，服务的幂等性，是指针对同一个服务的多次(n>1)请求和对它的单次(n=1)请求，二者具有相同的副作用。

在TCC事务模型中，Confirm/Cancel业务可能会被重复调用，其原因很多。比如，全局事务在提交/回滚时会调用各TCC服务的Confirm/Cancel业务逻辑。执行这些Confirm/Cancel业务时，可能会出现如网络中断的故障而使得全局事务不能完成。因此，故障恢复机制后续仍然会重新提交/回滚这些未完成的全局事务，这样就会再次调用参与该全局事务的各TCC服务的Confirm/Cancel业务逻辑。

既然Confirm/Cancel业务可能会被多次调用，就需要保障其幂等性。
那么，应该由TCC事务框架来提供幂等性保障？还是应该由业务系统自行来保障幂等性呢？
个人认为，应该是由TCC事务框架来提供幂等性保障。如果仅仅只是极个别服务存在这个问题的话，那么由业务系统来负责也是可以的；然而，这是一类公共问题，毫无疑问，所有TCC服务的Confirm/Cancel业务存在幂等性问题。TCC服务的公共问题应该由TCC事务框架来解决；而且，考虑一下由业务系统来负责幂等性需要考虑的问题，就会发现，这无疑增大了业务系统的复杂度。

#### 五、TCC事务框架不能盲目的依赖Cancel业务来回滚事务

前文以及提到过，TCC事务通过Cancel业务来对Try业务进行回撤的机制暗含了一个事实：Try操作已经生效。也就是说，只有Try操作所参与的RM本地事务已经提交的情况下，才需要执行其Cancel操作进行回撤。没有执行、或者执行了但是其RM本地事务被rollback的Try业务，是一定不能执行其Cancel业务进行回撤的。因此，TCC事务框架在全局事务回滚时，应该根据TCC服务的Try业务的执行情况选择合适的处理机制。而不能盲目的执行Cancel业务，否则就会导致数据不一致。

一个TCC服务的Try操作是否生效，这是TCC事务框架应该知道的，因为其Try业务所参与的RM事务也是由TCC事务框架所commit/rollbac的（前提是TCC事务框架接管了Spring的事务管理器）。所以，TCC事务回滚时，TCC事务框架可考虑如下处理策略：
1）如果TCC事务框架发现某个服务的Try操作的本地事务尚未提交，应该直接将其回滚，而后就不必再执行该服务的cancel业务；
2）如果TCC事务框架发现某个服务的Try操作的本地事务已经回滚，则不必再执行该服务的cancel业务；
3）如果TCC事务框架发现某个服务的Try操作尚未被执行过，那么，也不必再执行该服务的cancel业务。

总之，TCC事务框架应该保障：
1）已生效的Try操作应该被其Cancel操作所回撤；
2）尚未生效的Try操作，则不应该执行其Cancel操作。这一点，不是幂等性所能解决的问题。如上文所述，幂等性是指服务被执行一次和被执行n(n>0)次所产生的影响相同。但是，未被执行和被执行过，二者效果肯定是不一样的，这不属于幂等性的范畴。

#### 六、Cancel业务与Try业务并行，甚至先于Try操作完成

这应该算TCC事务机制特有的一个不可思议的陷阱。一般来说，一个特定的TCC服务，其Try操作的执行，是应该在其Confirm/Cancel操作之前的。Try操作执行完毕之后，Spring容器再根据Try操作的执行情况，指示TCC事务框架提交/回滚全局事务。然后，TCC事务框架再去逐个调用各TCC服务的Confirm/Cancel操作。

然而，超时、网络故障、服务器的重启等故障的存在，使得这个顺序会被打乱。比如：

![image](http://www.bytesoft.org/images/how-to-impl-tcc/tcc-impl.png)

 

这种情况下，由于[B:Cancel]执行时，[B:Try]尚未生效（其RM本地事务尚未提交），因此，[B:Cancel]是不能执行的，至少是不能生效（执行了其RM本地事务也要rollback）的。然而，当
[B:Cancel]处理完毕（跳过执行、或者执行后rollback其RM本地事务）后，[B:Try]操作完成又生效了（其RM本地事务成功提交），这就会使得[B:Cancel]虽然提供了，但却没有起到回撤[B:Try]的作用，导致数据的不一致。

所以，TCC框架在这种情况下，需要：
1）将[B:Try]的本地事务标注为rollbackOnly，阻止其后续生效；
2）禁止其再次将事务上下文传递给其他远程分支，否则该问题将在其他分支上出现；
3）相应地，[B:Cancel]也不必执行，至少不能生效。

> 当然，TCC事务框架也可以简单的选择阻塞[B:Cancel]的处理，待[B:Try]执行完毕后，再根据它的执行情况判断是否需要执行[B:Cancel]。不过，这种处理方式因为需要等待，所以，处理效率上会有所不及。

同样的情况也会出现在confirm业务上，只不过，发生在Confirm业务上的处理逻辑与发生在Cancel业务上的处理逻辑会不一样，TCC框架必须保证：
1）Confirm业务在Try业务之后执行，若发现并行，则只能阻塞相应的Confirm业务操作；
2）在进入Confirm执行阶段之后，也不可以再提交同一全局事务内的新的Try操作的RM本地事务。

#### 七、TCC服务复用性是不是相对较差？

TCC事务机制的定义，决定了一个服务需要提供三个业务实现：Try业务、Confirm业务、Cancel业务。可能会有人因此认为TCC服务的复用性较差。怎么说呢，要是将 Try/Confirm/Cancel业务逻辑单独拿出来复用，其复用性当然是不好的，Try/Confirm/Cancel 逻辑作为TCC型服务中的一部分，是不能单独作为一个组件来复用的。Try、Confirm、Cancel业务共同才构成一个组件，如果要复用，应该是复用整个TCC服务组件，而不是单独的Try/Confirm/Cancel业务。

#### 八、TCC服务是否需要对外暴露三个服务接口？

不需要。TCC服务与普通的服务一样，只需要暴露一个接口，也就是它的Try业务。Confirm/Cancel业务逻辑，只是因为全局事务提交/回滚的需要才提供的，因此Confirm/Cancel业务只需要被TCC事务框架发现即可，不需要被调用它的其他业务服务所感知。

换句话说，业务系统的其他服务在需要调用TCC服务时，根本不需要知道它是否为TCC型服务。因为，TCC服务能被其他业务服务调用的也仅仅是其Try业务，Confirm/Cancel业务是不能被其他业务服务直接调用的。

#### 九、TCC服务A的Confirm/Cancel业务中能否调用它依赖的TCC服务B的Confirm/Cancel业务？

最好是不要这样做。首先，没有必要。TCC服务A依赖TCC服务B，那么[A:Try]已经将事务上下文传播给[B:Try]了，后续由TCC事务框架来调用各自的Confirm/Cancel业务即可；其次，Confirm/Cancel业务如果被允许调用其他服务，那么它就有可能再次发起新的TCC全局事务。如此递归下去，将会导致全局事务关系混乱且不可控。

TCC全局事务，应该尽量在Try操作阶段传播事务上下文。Confirm/Cancel操作阶段仅需要完成各自Try业务操作的确认操作/补偿操作即可，不适合再做远程调用，更不能再对外传播事务上下文。

综上所述，本文倾向于认为，实现一个通用的TCC分布式事务管理框架，还是相对比较复杂的。一般业务系统如果需要使用TCC事务机制，并不推荐自行设计实现。
这里，给大家推荐一款开源的TCC分布式事务管理器[ByteTCC](https://github.com/liuyangming/ByteTCC)。ByteTCC基于Try/Confirm/Cancel机制实现，可与Spring容器无缝集成，兼容Spring的声明式事务管理。提供对dubbo框架、Spring Cloud的开箱即用的支持，可满足多数据源、跨应用、跨服务器等各种分布式事务场景的需求。

# [TCC是两阶段提交的一种么？](http://www.bytesoft.org/is-tcc-a-type-of-2pc/)

经常在网络上看见有人介绍TCC时，都提一句，”TCC是两阶段提交的一种”。其理由是TCC将业务逻辑分成try、confirm/cancel在两个不同的阶段中执行。其实这个说法，是不正确的。可能是因为既不太了解两阶段提交机制、也不太了解TCC机制的缘故，于是将两阶段提交机制的prepare、commit两个事务提交阶段和TCC机制的try、confirm/cancel两个业务执行阶段互相混淆，才有了这种说法。

两阶段提交（Two Phase Commit，下文简称2PC），简单的说，是将事务的提交操作分成了prepare、commit两个阶段。其事务处理方式为：
1、 在全局事务决定提交时，a）逐个向RM发送prepare请求；b）若所有RM都返回OK，则逐个发送commit请求最终提交事务；否则，逐个发送rollback请求来回滚事务；
2、 在全局事务决定回滚时，直接逐个发送rollback请求即可，不必分阶段。
*** 需要注意的是：2PC机制需要RM提供底层支持（一般是兼容XA），而TCC机制则不需要。

TCC（Try-Confirm-Cancel），则是将业务逻辑分成try、confirm/cancel两个阶段执行，具体介绍见[TCC事务机制简介](http://www.bytesoft.org/tcc-intro/)。其事务处理方式为：
1、 在全局事务决定提交时，调用与try业务逻辑相对应的confirm业务逻辑；
2、 在全局事务决定回滚时，调用与try业务逻辑相对应的cancel业务逻辑。
可见，TCC在事务处理方式上，是很简单的：要么调用confirm业务逻辑，要么调用cancel逻辑。这里为什么没有提到try业务逻辑呢？因为try逻辑与全局事务处理无关。

当讨论2PC时，我们只专注于事务处理阶段，因而只讨论prepare和commit，所以，可能很多人都忘了，使用2PC事务管理机制时也是有业务逻辑阶段的。正是因为业务逻辑的执行，发起了全局事务，这才有其后的事务处理阶段。实际上，使用2PC机制时————以提交为例————一个完整的事务生命周期是：begin -> 业务逻辑 -> prepare -> commit。

再看TCC，也不外乎如此。我们要发起全局事务，同样也必须通过执行一段业务逻辑来实现。该业务逻辑一来通过执行触发TCC全局事务的创建；二来也需要执行部分数据写操作；此外，还要通过执行来向TCC全局事务注册自己，以便后续TCC全局事务commit/rollback时回调其相应的confirm/cancel业务逻辑。所以，使用TCC机制时————以提交为例————一个完整的事务生命周期是：begin -> 业务逻辑(try业务) -> commit(comfirm业务)。

综上，我们可以从执行的阶段上将二者一一对应起来：
1、 2PC机制的业务阶段 等价于 TCC机制的try业务阶段；
2、 2PC机制的提交阶段（prepare & commit） 等价于 TCC机制的提交阶段（confirm）；
3、 2PC机制的回滚阶段（rollback） 等价于 TCC机制的回滚阶段（cancel）。

因此，可以看出，虽然TCC机制中有两个阶段都存在业务逻辑的执行，但其中try业务阶段其实是与全局事务处理无关的。认清了这一点，当我们再比较TCC和2PC时，就会很容易地发现，TCC不是两阶段提交，而只是它对事务的提交/回滚是通过执行一段confirm/cancel业务逻辑来实现，仅此而已。

# [TCC事务机制简介](http://www.bytesoft.org/tcc-intro/)

关于TCC（Try-Confirm-Cancel）的概念，最早是由Pat Helland于2007年发表的一篇名为《Life beyond Distributed Transactions:an Apostate’s Opinion》的论文提出。在该论文中，TCC还是以Tentative-Confirmation-Cancellation作为名称；正式以Try-Confirm-Cancel作为名称的，可能是Atomikos（Gregor Hohpe所著书籍《Enterprise Integration Patterns》中收录了关于TCC的介绍，提到了Atomikos的Try-Confirm-Cancel，并认为二者是相似的概念）。

国内最早关于TCC的报道，应该是InfoQ上对阿里程立博士的一篇采访。经过程博士的这一次传道之后，TCC在国内逐渐被大家广为了解并接受。相应的实现方案和开源框架也先后被发布出来，[ByteTCC](https://github.com/liuyangming/ByteTCC)就是其中之一。

TCC事务机制相对于传统事务机制（X/Open XA Two-Phase-Commit），其特征在于它不依赖资源管理器(RM)对XA的支持，而是通过对（由业务系统提供的）业务逻辑的调度来实现分布式事务。

对于业务系统中一个特定的业务逻辑S，其对外提供服务时，必须接受一些不确定性，即对业务逻辑执行的一次调用仅是一个临时性操作，调用它的消费方服务M保留了后续的取消权。如果M认为全局事务应该rollback，它会要求取消之前的临时性操作，这将对应S的一个取消操作；而当M认为全局事务应该commit时，它会放弃之前临时性操作的取消权，这对应S的一个确认操作。

每一个初步操作，最终都会被确认或取消。因此，针对一个具体的业务服务，TCC事务机制需要业务系统提供三段业务逻辑：初步操作Try、确认操作Confirm、取消操作Cancel。

\1. 初步操作（Try）
TCC事务机制中的业务逻辑（Try），从执行阶段来看，与传统事务机制中业务逻辑相同。但从业务角度来看，却不一样。TCC机制中的Try仅是一个初步操作，它和后续的确认一起才能真正构成一个完整的业务逻辑。可以认为

TCC机制将传统事务机制中的业务逻辑一分为二，拆分后保留的部分即为初步操作（Try）；而分离出的部分即为确认操作（Confirm），被延迟到事务提交阶段执行。
TCC事务机制以初步操作（Try）为中心的，确认操作（Confirm）和取消操作（Cancel）都是围绕初步操作（Try）而展开。因此，Try阶段中的操作，其保障性是最好的，即使失败，仍然有取消操作（Cancel）可以将其不良影响进行回撤。

\2. 确认操作（Confirm）
确认操作（Confirm）是对初步操作（Try）的一个补充。当TCC事务管理器决定commit全局事务时，就会逐个执行初步操作（Try）指定的确认操作（Confirm），将初步操作（Try）未完成的事项最终完成。

\3. 取消操作（Cancel）
取消操作（Cancel）是对初步操作（Try）的一个回撤。当TCC事务管理器决定rollback全局事务时，就会逐个执行初步操作（Try）指定的取消操作（Cancel），将初步操作（Try）已完成的事项全部撤回。

在传统事务机制中，业务逻辑的执行和事务的处理，是在不同的阶段由不同的部件来完成的：业务逻辑部分访问资源实现数据存储，其处理是由业务系统负责；事务处理部分通过协调资源管理器以实现事务管理，其处理由事务管理器来负责。二者没有太多交互的地方，所以，传统事务管理器的事务处理逻辑，仅需要着眼于事务完成（commit/rollback）阶段，而不必关注业务执行阶段。

而在TCC事务机制中的业务逻辑处理和事务处理，其关系就错综复杂：业务逻辑（Try/Confirm/Cancel）阶段涉及所参与资源事务的commit/rollback；全局事务commit/rollback时又涉及到业务逻辑（Try/Confirm/Cancel）的执行。其中关系，本站将另行撰文详细介绍，敬请关注！

参考文献：

1. <http://www.infoq.com/cn/interviews/soa-chengli>
2. <https://cs.brown.edu/courses/cs227/archives/2012/papers/weaker/cidr07p15.pdf>
3. <http://www.enterpriseintegrationpatterns.com/patterns/conversation/TryConfirmCancel.html>
4. <http://blog.51cto.com/robertleepeak/2083454?wx=>