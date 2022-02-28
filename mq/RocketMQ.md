# RocketMQ学习笔记

### 为什么要使用消息队列？

#### 解耦

传统模式的缺点：

> 系统间耦合性太强，如上图所示，系统A在代码中直接调用系统B和系统C的代码，如果将来D系统接入，系统A还需要修改代码，过于麻烦！

![img](https://img-blog.csdnimg.cn/9f21ae0bd21649cf89fa2307782b504d.png) 



中间件模式的优点：

> 将消息写入消息队列，需要消息的系统自己从消息队列中订阅，从而系统A不需要做任何修改

![img](https://img-blog.csdnimg.cn/716ab0dc553b4e009fb4ca38f5432d3f.png) 

#### 异步

传统模式的缺点：

> 一些非必要的业务逻辑以同步的方式运行，太耗费时间。

![img](https://img-blog.csdnimg.cn/9bc362f8ffa14aa18c46373dfb3dc7e5.png) 

中间件模式的的优点：

> 将消息写入消息队列，非必要的业务逻辑以异步的方式运行，加快响应速度

![img](https://img-blog.csdnimg.cn/51f54209bb87493a81d5fea3710b2f1c.png) 

#### 削峰

传统模式的缺点：

> 并发量大的时候，所有的请求直接怼到数据库，造成数据库连接异常

![img](https://img-blog.csdnimg.cn/7d94801125214595a5bd3499a3189ea5.png) 

中间件模式的的优点：

> 系统A慢慢的按照数据库能处理的并发量，从消息队列中慢慢拉取消息。在生产中，这个短暂的高峰期积压是允许的

![img](https://img-blog.csdnimg.cn/54f81313d68647cb8ceeb4fd97b92470.png) 

### 使用了消息队列会有什么缺点?

我们引入一个技术，要对这个技术的弊端有充分的认识，才能做好预防
从以下两个个角度来答

- 系统可用性降低：本来其他系统只要运行好好的，那你的系统就是正常的。现在你非要加个消息队列进去，那消息队列挂了，你的系统不是挂了。因此，系统可用性降低
- 系统复杂性增加：要多考虑很多方面的问题，比如一致性问题、如何保证消息不被重复消费，如何保证保证消息可靠传输。因此，需要考虑的东西更多，系统复杂性增大。

但是，我们该用还是要用的。

### 消息队列如何选型?

| 特性       | ActiveMQ                                                     | RabbitMQ                                                     | RocketMQ                 | kafka                                                        |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------ | ------------------------------------------------------------ |
| 开发语言   | java                                                         | erlang                                                       | java                     | scala                                                        |
| 单机吞吐量 | 万级                                                         | 万级                                                         | 10万级                   | 10万级                                                       |
| 时效性     | ms级                                                         | us级                                                         | ms级                     | ms级以内                                                     |
| 可用性     | 高(主从架构)                                                 | 高(主从架构)                                                 | 非常高(分布式架构)       | 非常高(分布式架构)                                           |
| 功能特性   | 成熟的产品，在很多公司得到应用；有较多的文档；各种协议支持较好 | 基于erlang开发，所以并发能力很强，性能极其好，延时很低;管理界面较丰富 | MQ功能比较完备，扩展性佳 | 只支持主要的MQ功能，像一些消息查询，消息回溯等功能没有提供，毕竟是为大数据准备的，在大数据领域应用广 |

综合上面的材料得出以下两点: 

1. 中小型软件公司，建议选RabbitMQ
   1. 一方面，erlang语言天生具备高并发的特性，而且他的管理界面用起来十分方便。
   2. 正所谓，成也萧何，败也萧何！他的弊端也在这里，虽然RabbitMQ是开源的，然而国内有几个能定制化开发erlang的程序员呢？所幸，RabbitMQ的社区十分活跃，可以解决开发过程中遇到的bug，这点对于中小型公司来说十分重要。
   3. 不考虑rocketmq和kafka的原因是，一方面中小型软件公司不如互联网公司，数据量没那么大，选消息中间件，应首选功能比较完备的，所以kafka排除。
   4. 不考虑rocketmq的原因是，rocketmq是阿里出品，如果阿里放弃维护rocketmq，中小型公司一般抽不出人来进行rocketmq的定制化开发，因此不推荐。 
2. 大型软件公司，根据具体使用在rocketMq和kafka之间二选一
   1. 一方面，大型软件公司，具备足够的资金搭建分布式环境，也具备足够大的数据量。针对rocketMQ,大型软件公司也可以抽出人手对rocketMQ进行定制化开发，毕竟国内有能力改JAVA源码的人，还是相当多的
   2. 至于kafka，根据业务场景选择，如果有日志采集功能，肯定是首选kafka了。具体该选哪个，看使用场景



### 如何保证消息队列是高可用的？

在第二点说过了，引入消息队列后，系统的可用性下降。

在生产中，没人使用单机模式的消息队列。

因此，作为一个合格的程序员，应该对消息队列的高可用有很深刻的了解。

如果面试的时候，面试官问，你们的消息中间件如何保证高可用的？

你的回答只是表明自己只会订阅和发布消息，面试官就会怀疑你是不是只是自己搭着玩，压根没在生产用过。 

### 如何保证消息不被重复消费？

这个问题其实换一种问法就是，**如何保证消息队列的幂等性?**

这个问题可以认为是消息队列领域的基本问题。换句话来说，是在考察你的设计能力，这个问题的回答可以根据具体的业务场景来答，没有固定的答案。 



先来说一下**为什么会造成重复消费?**

其实无论是那种消息队列，造成重复消费原因其实都是类似的。

- 正常情况下，消费者在消费消息时候，消费完毕后，会发送一个确认信息给消息队列，消息队列就知道该消息被消费了，就会将该消息从消息队列中删除。
- 只是不同的消息队列发送的确认信息形式不同,例如RabbitMQ是发送一个ACK确认消息，RocketMQ是返回一个CONSUME_SUCCESS成功标志，kafka实际上有个offset的概念，简单说一下,就是每一个消息都有一个offset，kafka消费过消息后，需要提交offset，让消息队列知道自己已经消费过了。

那**造成重复消费的原因?**

- 就是因为网络传输等等故障，确认信息没有传送到消息队列，导致消息队列不知道自己已经消费过该消息了，再次将该消息分发给其他的消费者。

**如何解决?**

这个问题针对业务场景来答分以下几点

1. 比如，你拿到这个消息做数据库的insert操作。那就容易了，给这个消息做一个唯一主键，那么就算出现重复消费的情况，就会导致主键冲突，避免数据库出现脏数据。
2. 再比如，你拿到这个消息做redis的set的操作，那就容易了，不用解决，因为你无论set几次结果都是一样的，set操作本来就算幂等操作。
3. 如果上面两种情况还不行，上大招。准备一个第三方介质,来做消费记录。
   1. 以redis为例，给消息分配一个全局id，只要消费过该消息，将<id,message>以K-V形式写入redis。那消费者开始消费前，先去redis中查询有没消费记录即可。 

### 如何保证消费的可靠性传输?

我们在使用消息队列的过程中，应该做到消息不能多消费，也不能少消费。

如果无法做到可靠性传输，可能给公司带来千万级别的财产损失。同样的，如果可靠性传输在使用过程中，没有考虑到，这不是给公司挖坑么，你可以拍拍屁股走了，公司损失的钱，谁承担？

还是那句话，认真对待每一个项目，不要给公司挖坑。

 

其实这个可靠性传输，每种MQ都要从三个角度来分析：

- ==生产者弄丢数据==
- ==消息队列弄丢数据==
- ==消费者弄丢数据==

### 如何保证消息的顺序性？

其实并非所有的公司都有这种业务需求，但是还是对这个问题要有所复习。 

针对这个问题，通过某种算法，将需要保持先后顺序的消息放到同一个消息队列中(kafka中就是partition,rabbitMq中就是queue)。

然后只用一个消费者去消费该队列。

 有的人会问：**那如果为了吞吐量，有多个消费者去消费怎么办？ **

这个问题，没有固定回答的套路。

比如我们有一个微博的操作，发微博、写评论、删除微博，这三个异步操作。

如果是这样一个业务场景，那只要重试就行。

比如你一个消费者先执行了写评论的操作，但是这时候，微博都还没发，写评论一定是失败的，等一段时间。等另一个消费者，先执行写评论的操作后，再执行，就可以成功。

 总之，针对这个问题，我的观点是**保证入队有序就行，出队以后的顺序交给消费者自己去保证，没有固定套路。 **



## 简介

### 什么是MQ？

MQ，message queue消息队列，一种提供消息队列服务的中间件，是一套提供了消息生产、储存、消费全过程的API软件系统

消息即数据

### MQ用途

#### 削峰限流

MQ可以将系统的**超量**请求暂存其中，以便系统慢慢处理，避免请求丢失或系统被压垮

![1634563603503](https://img-blog.csdnimg.cn/2896e41b79764b0096565d7bae58ace0.png)

#### 异步解耦

上游系统对下游系统的调用若为同步调用，则会大大降低系统的吞吐量与并发度，且系统耦合度太高。而异步调用则会解决这些问题。所以两层之间若要实现由同步到异步的转化，一般性做法就是，在这两层间添加一一个MQ层。

> 后台保证了数据库一定能正常修改运行，所以我无序同步等待执行结束，而是直接在用户请求后返回订单号，接着异步去处理数据库

![1634563901915](https://img-blog.csdnimg.cn/7aa4faca1883471c86c070986233c1ef.png)

### 数据收集

分布式系统会产生海量级数据流，如:业务日志、监控数据、用户行为等

针对这些数据流进行实时或批量采集汇总，然后对这些数据流进行大数据分析，这是当前互联网平台的必备技术。通过MQ完成此类数据收集是最好的选择。



## 常见的MQ协议

> MOM是什么？
>
> MOM（Message Oriented Middleware）是面向消息的中间件，使用消息传送提供者来协调消息传送操作。MOM 需要提供 API 和管理工具。客户端使用api调用，把消息发送到由提供者管理的目的地。在发送消息之后，客户端会继续执行其他工作，并且在接收方收到这个消息确认之前，提供者一直保留该消息 
>
> - PO：面向过程，Process Oriented
> - OO：面向对象，Object Oriented
> - AO：面向切面，Aspect Oriented

**JMS**

java message service，java消息服务

是java平台有关MOM的技术规范，它便于消息系统中的java应用程序进行消息交换，并且通过提供标准的产生、发送、接收消息的接口，简化企业应用的开发。

ActiveMQ是该协议的典型实现

**STOMP**

Stream Text Orientated Message Protocol，面向文本流的消息协议

是一种MOM设计的简单文本协议，STOMP提供一个可互操作的连接格式，允许客户端与任意STOMP消息代理（Broker）进行交互。

ActiveMQ是该协议的典型实现，RabbitMQ通过插件可以支持该协议

**AMQP**

Advanced Message Queuing Protocol，高级消息队列协议

一个提供统一消息服务的应用层标准，是应用层协议的一个开放标准，是一种MOM设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同开发语言等条件的限制。

RabbitMQ是该协议的典型实现。

**MQTT**

Message Queuing Telemetry Transport，消息队列遥测传输

是IBM开发的一个即时通讯协议，是一种二进制协议，主要用于服务器和低功耗IoT设备（物联网）间的通信。该协议支持所有平台，几乎可以把所有联网物品和外部连接起来，被用来当做传感器和致动器的通信协议。

RabbitMQ通过插件可以支持该协议。



**以上协议，RockerMQ都不支持**



## 基本概念

### 消息与主题

#### 消息（Message）

消息是指，消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息必须属于一个主题。

#### 主题（Topic）

topic表示一类消息的集合，每个主题包含若干个消息，每条消息只属于一个主题，是对rocketmq进行消息订阅的基本单位

一个生产者可以发送多种topic的消息，而一个消费者只对某种特定的topic感兴趣，即只可以订阅和消费一种topic的消息

![](https://img-blog.csdnimg.cn/4c478c0c3bfe437593a03a2e6fd7d76d.png)

#### 标签（Tags）

消息标签，用来进一步区分某个 Topic 下的消息分类，消息队列 RocketMQ 允许消费者按照 Tag 对消息进行过滤，确保消费者最终只消费到他关注的消息类型

Topic 与 Tag 都是业务上用来==归类==的标识，区分在于 Topic 是一级分类，而 Tag 可以说是二级分类，关系如图所示 

![在这里插入图片描述](https://img-blog.csdnimg.cn/b691cf0ebbd0496ca7a4aa3031c58989.png)

#### 队列（Queue）

储存消息的实体，一个topic中可以含有多个queue，每个queue中储存的是该topic的信息

一个topic的queue也被称为一个topic中消息的分区

一个topic的queue的消息只能被**一个消费者组中的消费者**进行读取消费，且一个queue的消息不允许同一个消费者组里多个消费者同时消费（后面说的广播模式推翻了这个说法）

> queue：消费者 => 1：1
>
> 消费者：queue => 1：n

为了提升MQ的效率，所以将topic分为多个分区去进行读取（个人感觉有点像负载均衡）

注：1号狗消费了1号骨头，就不能和2号狗去争夺2号骨头了

如果此时只有1和2号两只狗的时候，就可以让1号消费1、2号骨头，而2号狗消费3号骨头了

就是说一个消费者组里不能互相争夺分区的消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/416e6f4e1c594864a349224130f6ee2a.png)

在学习参考其它相关资料时，还会看到一个概念：分片(Sharding)

官方是没有这个概念的，但是在网上有这种说法

> 分片不同于分区。在RocketMQ中，分片指的是存放相应Topic的Broker的数量。
>
> 每个分片中会创建出相应数量的分区，即Queue,每个Queue的大小都是相同的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/a73a68a963b84f508ff102276d069ffd.png)

#### 消息标识（MessageId/Key）

- RocketMQ中**每个消息拥有唯一的MessageId**, 且**可以携带具有业务标识的Key**，以方便对消息的查询
- 不过需要注意的是，MessageId有两个: 在生产者send0消息时会自动生成一个MessageId (msgId)， 当消息到达Broker后，Broker也会自动生成一 个Messageld(称为offsetMsgId)
- msgId、 offsetMsgId与key都称为消息标识
  - msgId：由produce端生成，生成规则为
    - producerIp+进程Pid+MessageClientSetter类的ClassLoader的hashCode+当前时间+AutomiInteger自增器
  - offsetMsgId：由broker端生成，生成规则为
    - brokerIp+物理分区的offset（也就是queue的偏移量）
  - key：由用户指定的业务相关的唯一标识



## 系统架构

![在这里插入图片描述](https://img-blog.csdnimg.cn/a92129ce337a4e0d9745c3c1b3cbd6b7.png)

### 1.Producer生产者

消息生产者，负责生产消息

> 比如我将系统的日志写入MQ，就是一个消息的生成
>
> 将用户的请求写入MQ，也是一个消息的生成

Producer通过**MQ的负载均衡模块**选择相应的Broker集群队列进行消息投递，投递的过程**支持快速失败**并且**低延迟**

RocketMQ中的消息生产者都是以==生产者组==(Producer Group)的形式出现的。

生产者组是同一类生产者的集合，这类Producer发送==相同==Topic类型的消息。一个生产者组可以同时发送多个主题的消息（我的理解是可以生产多个主题的消息，但是发送时要保持消息的主题是一致的）

### 2.Consumer消费者

消息消费者，负责消费消息

> 从MQ取出日志信息进行解析，就是消息的消费
>
> 用户下单后，从MQ中读取用户的请求并进行处理的过程，也是消息的消费过程

一个消息消费者会从Broker服务器中获取到消息，并对消息进行相关业务处理

RocketMQ中的消息消费者都是以==消费者组==(Consumer Group)的形式出现的

消费者组是同一类消费者的集合，**这类Consumer消费的是同一个Topic类型的消息**，一类消费者只能消费一种topic的消息

消费者组使得在消息消费方面，实现负载均衡和容错的目标变得非常容易

- 负载均衡：将一个topic中不同的queue分配给不同的consumer，看来我前面说很像负载均衡说对了
- 容错：一个consumer挂了，还有别的consumer可以接着消费（分布式微服务容错机制）

消费者组中Consumer的数量应该小于等于订阅Topic的Queue数量。如果超出Queue数量，则多出的Consumer将不能消费消息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/125c1ba792ea48aa84c933e2fdcd87eb.png)

一个topic类型的消息可以被多个消费者组消费

注：

- 一个消费者组里消费者必须订阅**完全相同**的topic
- 一个消费者组只能消费一个topic的消息，不能同时消费多个topic的消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/1a465be963824c85aa922f360c0f7cfb.png)



### 3.Name Server注册中心

NameServer是一个Broker与Topic路由的**注册中心**，支持Broker的动态注册与发现。



主要包括两个功能:

- **Broker管理**: 
  - 接受Broker集群的注册信息并且保存下来作为路由信息的基本数据
  - 提供心跳检测机制，检查Broker是否还存活。

- **路由信息管理**:
  - 每个NameServer中都保存着Broker集群的整个路由信息和用于客户端查询的队列信息。
  - Producer和Consumer通过NameServer可以获取整个Broker集群的路由信息，从而进行消息的投递和消费。

#### 路由注册

NameServer通常也是以集群的方式部署，不过， NameServer是**无状态**的，即NameServer集群中的各个节点间是无差异的，**各节点间相互不进行信息通讯**

那各节点中的数据是如何进行数据同步的呢?

> 在Broker节点启动时，轮询NameServer列表，与**每个**NameServer节点建立长连接，发起注册请求。
>
> 在NameServer内部维护着1个Broker列表， 用来动态存储Broker的信息。

注意：

> 与其他zk、euraka、nacos等注册中心不同，别的都是注册一个然后内部通信，nameserver是所有都要注册
>
> 优点：无状态的集群搭建很简单
>
> 缺点：对于broker必须明确指出nameserver地址，未指出的不会进行注册，使得扩容、维护不方便



Broker节点为了证明自己是活着的，为了维护与NameServer间的长连接，会将最新的信息以心跳包的方式上报给NameServer,每30秒发送一次心跳。

 心跳包中包含Brokerld、Broker地址（IP+port）、 Broker名称、Broker所属集群名称等等。

NameServer在接收到心跳包后，会更新心跳时间戳，记录这个Broker的最新存活时间。

#### 路由剔除

由于Broker关机、宕机或网络抖动等原因，NameServer没有收到Broker的心跳，NameServer可能会将其从Broker列表中剔除

NameServer中有一个定时任务，每隔10秒就会扫描一次Broker表， 查看每一个Broker的最新心跳时间戳距离当前时间是否超过120秒，如果超过，则会判定Broker失效，然后将其从Broker列表中剔除。

#### 路由发现

RocketMQ的路由发现采用的是Pull模型。当Topic路由信息出现变化时，NameServer不会主动推送给客户端，而是客户端定时拉取主题最新的路由

默认客户端每30秒会拉取一次最新的路由。

> Pull模型：客户端隔一段时间去拉取服务端的数据
>
> - 可能存在“前脚拉完，后脚就改”的情况，实时性差
>
> Push模型：服务端被客户端订阅后，一旦发生改变，立马推送到客户端
>
> - 保证数据一致，实时性较好
> - 但是需要一直维护两边的长连接，占用资源
>
> Long Pulling模型：长轮询模型，客户端隔一段时间去拉取服务端的数据，且不立刻断开连接，而是服务端保持一段时间和客户端的连接，也就是Pull模型和Push模型的整合
>
> - nacos的配置中心用的就是这个，监控服务端的配置信息，一旦变更，客户端立马变更
> - 实时性较好
> - 对资源占用较少

所以push模式适合Client不多，数据变化频繁，对实时性要求高的业务需求

#### 客户端NameServer选择策略

> 这里的客户端指：producer和consumer

客户端在配置时必须要写上NameServer集群的地址，那么客户端到底连接的是哪个NameServer节点呢?

客户端首先会首先一个随机数， 然后再与NameServer节点数量取模，此时得到的就是所要连接的节点索引，然后就会进行连接。

如果连接失败，则会采用round robin轮询策略，逐个尝试着去连接其它节点。

即：首先采取**随机策略**进行选择，连接失败后再使用**轮询策略**



### 4.Broker中间人

#### 功能介绍

Broker充当着**消息中转角色**，负责**存储消息**、**转发消息**

Broker在RocketMQ系统中负责接收并存储从生产者发送来的消息，同时为后费者的拉取请求作准备

Broker同时也存储着消息相关的元数据，包括消费者组消费进度偏移量offset、主题topic、队列queue等

#### 模块构成

![在这里插入图片描述](https://img-blog.csdnimg.cn/d1d405ad92ef4355800aea7b36becf0f.png)

![img](https://img2018.cnblogs.com/blog/1090617/201906/1090617-20190626173042073-147043337.jpg)

**Remoting Module**：

整个Broker的实体，负责处理来自clients端的请求。这个Broker实体则由以下模块构成。

- **Client Manager：**客户端管理器

  - 负责接收、解析客户端(Producer/Consumer)请求， 管理客户端

  - > 例如，维护Consumer的Topic订阅信息

- **Store Service**：存储服务

  - 提供方便简单的API接口，处理**消息存储到物理硬盘**和**消息查询***功能

- **HA Service**：高可用服务

  - 提供Master Broker和Slave Broker之间的数据同步功能

- **Index Service**：索引服务

  - 根据特定的Message key，对投递到Broker的消息进行索引服务，同时也提供根据Message Key对消息进行快速查询的功能

#### 集群部署

为了增强Broker性能与吞吐量，Broker一般都是以集群形式出现的。各集群节点中可能存放着相同Topic的不同Queue。

不过，这里有个问题，如果某Broker节点宕机，如何保证数据不丢失呢？

其解决方案是，将每个Broker集群节点进行横向扩展，将Broker节点建为一个**高可用的HA集群**，解决单点问题



Broker节点集群是一个**主从集群**，即集群中具有**Master**与**Slave**两种角色。Master负责处理读写操作请求，Slave也可以负责读写操作请求（当master宕机后需要从slave中读写）

- 正常情况下都是操作master，slave只是备份服务，如果master宕机，则由slave顶上

- 一个Master可以包含多个Slave，但一个Slave只能属于一个Master。
- Master与Slave的对应关系是通过指定相同的BrokerName、不同的BrokerId 来确定的。
- BrokerId为0表示Master，非0表示Slave。
-  每个Broker与NameServer集群中的所有节点建立长连接，定时注册Topic信息到所有NameServer。

### 5.工作流程

1. 启动Name Server，开始监听端口，等待broker、producer和consumer的连接

2. 启动Broker时， Broker会与所有的NameServer建立并保持长连接，然后每30秒向NameServer定时发送心跳包

3. 发送消息前，可以先创建Topic。创建Topic时需要指定该Topic要存储在哪些Broker上， 当然，在创建Topic时也会将Topic与Broker的关系写入到NameServer中。不过，这步是可选的，也可以在发送消息时自动创建Topic

   > 手动创建Topic：
   >
   > ​	集群模式：该模式下创建的Topic在该集群中，所有Broker中的Queue数量是相同的
   >
   > ​	broker模式：该模式下创建的Topic在该集群中，每个Broker中的Queue数量可以是不同的
   >
   > 自动创建Topic：
   >
   > ​	默认采用的broker模式，默认给每个broker创建4个queue，可以在配置文件中更改

4. Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接（先随机再轮询），并从NameServer中获取路由信息，即当前发送的Topic的Queue 与Broker的地址(IP+Port) 的映射关系。然后根据算法策略从队选择-个Queue,与队列所在的Broker建 立长连接从而向Broker发消息。当然，在获取到路由信息后，Producer会首先将路由信息缓存到本地，再每30秒从NameServer更新一次路由信息

5. Consumer跟Producer类似， 跟其中一台NameServer建立长连接，获取其所订阅Topic的路由信息， 然后根据算法策略从路由信息中获取到其所要消费的Queue,然后直接跟Broker建立长连接，开始消费其中的消息。Consumer在获取到路由信息后，同样也会每30秒从NameServer更新一次路由信息。不过不同于Producer的是, Consumer还会向Broker发送心跳，以确保Broker的存活状态

#### 读/写队列问题

> 从物理上来讲，读/写队列是同一个队列。
>
> 所以，不存在读/写队列数据同步问题。读/写队列是逻辑上进行区分的概念。
>
> **一般情况下，读/写队列数量是相同的**

例如，创建Topic时设置的写队列数量为8，读队列数量为4，此时系统会创建8个Queue,分别是0 1 2 3 4 5 67。

Producer会将消息写入到这8个队列，但Consumer只会消费0 1 2 3这4个队列中的消息，4 5 6 7中的消息是不会被消费到的。

再如，创建Topic时设置的写队列数量为4，读队列数量为8，此时系统会创建8个Queue,分别是0 1 2 3 4 5 6 7。

Producer会将消息写入到0 1 2 3这4个队列，但Consumer只会消费0 1 2 3 4 5 6 7这8个队列中的消息，但是4 567中是没有消息的。此时假设Consumer Group中包含两个Consumer, Consumer1消费0 1 2 3，而Consumer2消费4567。但实际情况是，Consumer2是没有消息可消费的。



不管怎么样都会造成资源浪费等不好的情况，所以一般读写队列的数量要设计一致



## 单机的安装与启动

### 准备工作

#### 硬件

准备一台linux，我用的vm虚拟机centos，输入`ifconfig`获得`ens33`的ip为`inet 192.168.146.128`

在`C:\Windows\System32\drivers\etc\`修改host文件，将ip指向我自定义命名的centos域名

#### 环境

1. 64bit OS, Linux/Unix/Mac is recommended;(Windows user see guide below)
2. 64bit JDK 1.8+
3. Maven 3.2.x
4. Git
5. 4g+ free disk for Broker server

查看我的linux有没有jdk环境`echo $JAVA_HOME`，答案是没有

去官网下载linux的安装包，传到虚拟机



配置环境变量，输入3个值保存后，刷新profile

```bash
vi /etc/profile

# java
export JAVA_HOME=/usr/bin/java/jdk1.8.0_291
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib

source /etc/profile
```

测试java环境

```bash
javac
java -version
echo $JAVA_HOME
```

#### linux防火墙开放端口

> CentOS升级到7之后，发现无法使用iptables控制Linux的端口，因为Centos 7使用firewalld代替了原来的iptables。下面记录如何使用firewalld开放Linux端口：

**开启端口**

```bash
firewall-cmd --zone=public --add-port=9800/tcp --permanent
```

**查询端口是否开启**

```bash
firewall-cmd --query-port=9800/tcp
```

**重启防火墙**

```bash
firewall-cmd --reload
```

**查询有哪些端口是开启的**

```bash
firewall-cmd --list-port
```

这里方法先放着，等会如果出现开启控制台无法访问时，应该是端口号没打开，例如我rocketmq-console选择9800为端口，所以需要开放9800端口

### 下载RocketMQ

[官网地址](http://rocketmq.apache.org/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0660f40a81454aeebc4994ea085bed71.png)



我这里下载的4.9.1版本的，根据自己需求下载即可

[北京外国语镜像地址](https://mirrors.bfsu.edu.cn/apache/rocketmq/4.9.1/)

- Source源码版本
- Binary编译后的文件

我们下载Binary文件，zip文件，然后传到linux上

`unzip rocketmq-all-4.9.1-bin-release.zip`解压文件

### 修改启动内存

因为默认的内存太大了，2G、4G啥的，服务器的配置可能起不来，所以修改一下配置

**runserver.sh**

修改为256m

![在这里插入图片描述](https://img-blog.csdnimg.cn/da515e652d5748389d850c2e089020a0.png)

**runbroker.sh**

修改为256m

![在这里插入图片描述](https://img-blog.csdnimg.cn/3fc613c103f5422e81525e2f22a232ce.png)



### Linux下启动rocketmq



#### Start Name Server

```bash
 	# 启动nameserver服务
  > nohup sh bin/mqnamesrv &
  	# 查看日志是否启动成功
  > tail -f ~/logs/rocketmqlogs/namesrv.log
  The Name Server boot success...
  	# 查看当前java进程
  > jps
  240921 NamesrvStartup
  246169 Jps
```



#### Start Broker

```bash
  > nohup sh bin/mqbroker -n localhost:9876 -c conf/broker.conf &
  > tail -f ~/logs/rocketmqlogs/broker.log
  The broker[%s, 172.30.30.233:10911] boot success...
  > jps
  253009 BrokerStartup
  254698 Jps
  240921 NamesrvStartup
```

**端口必须开放9876和10911端口**

#### Send & Receive Messages

在发送/接收消息之前，我们需要告诉客户端名称服务器的位置。RocketMQ提供了多种方法来实现这一点。为了简单起见，我们使用环境变量`namersv_ADDR`

以下为官方提供的一个发消息和消费消息的用例

```bash
 > export NAMESRV_ADDR=localhost:9876
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
 SendResult [sendStatus=SEND_OK, msgId=7F0000011BE34AA298B76656B91103E7, offsetMsgId=C0A8928000002A9F000000000002ECD2, messageQueue=MessageQueue [topic=TopicTest, brokerName=centos, queueId=2], queueOffset=249]
 close the connection to remote address[127.0.0.1:9876] result: true
 
 > sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
 ConsumeMessageThread_2 Receive New Messages: [MessageExt [brokerName=centos, queueId=0, storeSize=192, queueOffset=145, sysFlag=0, bornTimestamp=1634734558909, bornHost=/192.168.146.128:33198, storeTimestamp=1634734558910, storeHost=/192.168.146.128:10911, msgId=C0A8928000002A9F000000000001B352, commitLogOffset=111442, bodyCRC=953417484, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='TopicTest', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=250, CONSUME_START_TIME=1634734753329, UNIQ_KEY=7F0000011BE34AA298B76656B6BD0245, CLUSTER=DefaultCluster, TAGS=TagA}, body=[72, 101, 108, 108, 111, 32, 82, 111, 99, 107, 101, 116, 77, 81, 32, 53, 56, 49], transactionId='null'}]]
 
```



#### Shutdown Servers

```bash
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```



### rocketmq控制台

[github地址](https://github.com/apache/rocketmq-externals/tags)

下载解压，是一个maven项目

#### 修改配置

进入src、resource文件夹，修改properties文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/6e49ec75ec5f439f881e2d39f009cfa4.png)

默认端口是8080，这我们当然要改动了

而且他也是相当于一个client，所以需要去获取nameserver，所以需要配置nameserver的地址

```shell
server.port=9800
rocketmq.config.namesrvAddr=localhost:9876
```

#### 导入依赖

导入JAXB依赖：Java Architecture for XML Binding 

> 允许Java开发人员将Java类映射为XML表示方式。
>
> JAXB提供两种主要特性：
>
> - 将一个Java对象序列化为XML，以及反向操作，将XML解析成Java对象。
> - 换句话说，JAXB允许以XML格式存储和读取数据，而不需要程序的类结构实现特定的读取XML和保存XML的代码。 



```xml
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-impl</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>com.sun.xml.bind</groupId>
    <artifactId>jaxb-core</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>javax.activation</groupId>
    <artifactId>activation</artifactId>
    <version>1.1.1</version>
</dependency>
```

#### 打包启动

跳过测试环节，目前不满足测试条件，会报错

```shell
mvn clean package -Dmaven.test.skip=true
```

得到结果则打包成功

```shell
[INFO] Building jar: E:\speciality_apps\RocketMQ\rocketmq-console-1.0.0\rocketmq-externals-rocketmq-console-1.0.0\rocketmq-console\target\rocketmq-console-ng-1.0.0-sources.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  54.114 s
[INFO] Finished at: 2021-10-20T21:29:12+08:00
[INFO] ------------------------------------------------------------------------
```

`rocketmq-console-ng-1.0.0-sources.jar`就是我们打包后的可执行jar，输入指令运行

```shell
java -jar rocketmq-console-ng-1.0.0.jar
```

出现运行成功和端口即可

```shell
[2021-10-20 21:41:23.209]  INFO Tomcat started on port(s): 9800 (http)
```

接下来访问我们的控制台，我这里访问的虚拟机

在OPS运维栏好像是可以添加删除和更新rocketmq的地址的

![在这里插入图片描述](https://img-blog.csdnimg.cn/c5029069b50b40b3b405bf2a26bc16e9.png)

#### 测试访问message

我们重新发送消息测试样例

```bash
export NAMESRV_ADDR=localhost:9876
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```

然后在导航栏选中`Message`，选中`TopicTest`点击search

查询出来刚刚发送过去但是还未消费的消息

![在这里插入图片描述](https://img-blog.csdnimg.cn/6cb3dc5f1c2e4b5c843980a49a424dac.png)



## 集群的搭建

![在这里插入图片描述](https://img-blog.csdnimg.cn/d58f3cf047ce40c1ab1d2aa174b05c4c.png)

producer和consumer集群只需要写入相同的group_name即可成为集群，启动一个就会增加一个节点，这里暂且不论

nameserver呢相互之间没有通讯，所以也是启动即节点，这里也不论

所以本章讲解的是**broker集群**

### 数据复制与刷盘策略

![在这里插入图片描述](https://img-blog.csdnimg.cn/da2f4280888a42f2a2de264555785f1b.png)

#### 复制策略

复制策略是Broker的Master与Slave间的数据同步方式

分为同步复制与异步复制：

- 同步复制:消息写入master后，master会等待slave同步数据成功后才向producer返回成功ACK
- 异步复制:消息写入master后，master立即向producer返回成功ACK， 无需等待slave同步数据成功



#### 刷盘策略

由内存到磁盘的过程，就叫刷盘（持久化？）

刷盘策略指的是broker中消息的落盘方式，即**消息发送到broker内存后消息持久化到磁盘的方式**

分为同步刷盘与异步刷盘:

- 同步刷盘:当消息持久化到broker的磁盘后才算是消息写入成功
- 异步刷盘:当消息写入到broker的内存后即表示消息写入成功，无需等待消息持久化到磁盘

> 1. 异步策略当然会降低延迟，提高吞吐量
> 2. 消息写入到Broker的内存，一般是写入了PageCache
> 3. 对于异步策略来说，写入到PageCache后不会立马进行刷盘操作，而是等待数据达到一定量的时候再进行落盘（应该也是提高效率，减少频繁的io）

### Broker集群模式

#### 单master

只有一个broker (其本质上就不能称为集群)。这种方式也只能是在测试时使用，生产环境下不能使用

因为存在**单点问题**

> 什么是单点问题？
>
> 指系统中一点失效，就会让整个系统无法运作的部件，换句话说，单点故障即会整体故障 

#### 多master

broker集群仅由多个master构成，不存在Slave。 同一Topic的各个Queue会平均分布在各个master节点上

- 优点：配置简单，单个Master宕机或重启维护对应用无影响，在磁盘配置为RAID10时，即使机器宕机不可恢复情况下，由于RAID10磁盘非常可靠，消息也不会丢(异步刷盘丢失少量消息，同步刷盘一条不丢)，性能最高

- 缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅(不可消费)，消息实时性会受到影响

> 什么是raid10？
>
> Raid 10是一个Raid 1与Raid0的组合体，它是利用奇偶校验实现条带集镜像，所以它继承了Raid0的快速和Raid1的安全。我们知道，RAID 1在这里就是一个冗余的备份阵列，而RAID 0则负责数据的读写阵列。其实，概述图只是一种RAID 10方式，更多的情况是从主通路分出两路，做Striping操作，即把数据分割，而这分出来的每一路则再分两路，做Mirroring操作，即互做镜像



#### 多master多slave模式——异步复制

broker集群由多个master构成，每个master又配置了多个slave 

注：在配置了RAID磁盘阵列的情况下，一个master一般配置一个slave即可

master 与slave的关系是主备关系，即**master负责处理消息的读写请求**，而**slave仅负责消息的备份与master宕机后的角色切换**



异步复制即前面所讲的复制策略中的异步复制策略，即消息写入master成功后，master立即向producer返回成功ACK，无需等待slave同步数据成功

该模式的最大特点之一是：**当master宕机后，slave能够自动切换为master**

不过由于slave从master的同步具有短暂的延迟(毫秒级)，所以当master宕机后，这种异步复制方式可能会存在少量消息的丢失问题

#### 多master多slave模式——同步双写

该模式是多Master多slave模式的同步复制实现

所谓同步双写，指的是消息写入master成功后，master会等待slave同步数据成功后才向producer返回成功ACK，即**master 与slave都要写入成功后才会返回成功ACK**，也即双写

该模式与异步复制模式相比，优点是消息的安全性更高，不存在消息丢失的情况。但单个消息的RT略高，从而导致性能要略低(大约低10%)

该模式存在一个大的问题：**对于目前的版本，Master宕机后， Slave不能自动切换到Master**

#### 最佳方案

> 一般会为Master配置RAID10磁盘阵列，然后再为其配置一个Slave
>
> 即利用了RAID10磁盘阵列的高效、安全性，又解决了可能会影响订阅的问题



但是RAID10磁盘阵列价格较高且可能会影响订阅的问题，所以个人认为还是多Master+slave+异步策略为最佳



## 集群搭建实践

### 配置

将linux系统克隆一份，选用两台机器进行集群的搭建



进入conf文件夹，查看文件夹，有系统默认给出的3个方案

![在这里插入图片描述](https://img-blog.csdnimg.cn/13f92076583342ef929b9d1578cd061c.png)

- 2m-2s-async：2主2从异步策略
- 2m-2s-sync：2主2从同步策略
- 2m-noslave：2主策略

我们就进去2m2s异步策略文件夹

```bash
[root@centos conf]# cd 2m-2s-async/
[root@centos 2m-2s-async]# ll
总用量 16
-rwxr-xr-x 1 root root 929 8月  16 06:20 broker-a.properties
-rwxr-xr-x 1 root root 922 8月  16 06:20 broker-a-s.properties
-rwxr-xr-x 1 root root 929 8月  16 06:20 broker-b.properties
-rwxr-xr-x 1 root root 922 8月  16 06:20 broker-b-s.properties
```

很明显的可以看出，-s是slave配置文件，正常的就是master配置文件

我们这是选用

- broker-a.properties+broker-b-s.properties
- broker-b.properties+broker-a-s.properties

即

- master1+slave2
- master2+slave1



vim修改配置文件broker-a.properties

```properties
#指定整个broker集群的名称，或者说是RocketMQ集群的名称
brokercluste rName=Defaultcluster
#指定master-slave集群的名称。一个RocketMQ集群可以包含多个master-slave集群，所以m和s保持一致
brokerName=broker-a
# master的brokerId为0，0代表master，1代表slave
brokerId=0
#指定删除消息存储过期文件的时间为凌晨4点
deletewhen=04
#指定未发生更新的消息存储文件的保留时长为48小时，48小时后过期，将会被删除
fileReservedTime=48
#指定当前broker为异步复制master
brokerRole=ASYNC_MASTER
#指定刷盘策略为异步刷盘
f1ushDi skType=ASYNC_FLUSH
```

我们需要配置，指定Name Server的地址

```properties
#指定Name Server的地址
namesrvAddr=192.168.59.164:9876;192.168.59.165:9876
```



修改配置文件broker-b-s.properties

```properties
brokercluste rName=Defaultcluster
#指定这是另外个master-slave集群
brokerName=broker-b
# slave的brokerId为非0
brokerId=1
deletewhen=04
fileReservedTime=48
#指定当前broker为slave后
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH
namesrvAddr=192.168.59.164:9876;192.168.59.165:9876
#指定Broker对外提供服务的端口，即Broker与producer与consumer通信的端口。默认10911。由于当前主机同时充当着master1与s]ave2，而前面的master1使用的是默认端口。这里需要将这两个端口加以区分，以区分出master1与s1ave2
listenPort=11911
#指定消息存储相关的路径。 默认路径为~/store目录。由于当前主机同时充当着master1与slave2, master1使用的是默认路径，这里就需要再指定一个不同路径
storePathRootDir=~/store-s
storePathcommi tLog=~/store-s/commit1og
storepathconsumeQueue=~/store-s/consumequeue
storepathIndex=~/store-s/ i ndex
storeCheckpoint=~/store-s/checkpoint
abortFile=~/store-s/abort
```



另一台机器同理设置另外两个配置



### 启动

#### 启动nameserver

上文有写

```bash
nohup sh bin/mqnamesrv &
tail -f ~/logs/rocketmqlogs/namesrv.log
```



#### 启动两个master

与上文不同，启动时需要指定文件启动

分别启动两个机器的broker master

```bash
nohup sh bin/mqbroker -C conf/2m-2s-async /broker-a.properties &
tail -f ~/logs/rocketmqlogs/broker.log 
```

 

```bash
nohup sh bin/mqbroker -C conf/2m-2s-async /broker-b.properties &
tail -f ~/logs/rocketmqlogs/broker.log
```





#### 启动两个slave

```bash
nohup sh bin/mqbroker -C conf/2m-2s-async /broker-b-s.properties &
tail -f ~/logs/rocketmqlogs/broker.log 
```



```bash
nohup sh bin/mqbroker -C conf/2m-2s-async /broker-a-s.properties &
tail -f ~/logs/rocketmqlogs/broker.log 
```



#### 修改控制台配置

```properties
rocketmq.config.namesrvAddr=ipA:9876;ipB:9877
```

应该也可以直接在控制台添加一个nameserver



## mqadmin命令

在mq的bin目录有一个mqadmin命令，该命令是一个运维命令，用于对mq的主题、集群、broker等信息进行管理

这个命令能够完成的功能在控制台都可以完成

这里就不再阐述了，贴一个[博客地址](https://blog.csdn.net/wulitaot/article/details/79551499)



## RocketMQ工作原理

这个是一个重点、难点、面试高频考点

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201220194632102.png)

### 消息的生产

#### 消息的生产过程

Producer可以将消息写入到某Broker中的某Queue中，其经历了如下过程：

- Producer发送消息之前，会先向NameServer发出获取**消息Topic的路由信息**的请求

- NameServer返回该Topic的**路由表**及**Broker列表**

  - > 路由表：实际是一个Map, key为Topic名称，value是一个QueueData实例列表
    >
    > QueueData并不是一个Queue, 对应一个QueueData, 而是一个Broker中该Topic的所有Queue对应一个QueueData即，只要涉及到该Topic的Broker，一个Broker对应一 个QueueData，QueueData中包含BrokerName
    >
    > 简单来说，路由表的key为Topic名称，value则为所有涉及该Topic的BrokerName列表
    >
    > 
    >
    > Broker列表：实际上也是一个Map，key为BrokerName，value是BrokerData
    >
    > 一个BrokerData对应了一套BrokerName相同的Master-Slave的小集群
    >
    > BrokerData中包含了一个BrokerName和一个map
    >
    > map的key为BrokerId（0-master，1-slave），value为broker的地址

- Producer根据代码中指定的Queue选择策略，从Queue列表中选出一个队列，用于后续存储消息

- Produer对消息做一些特殊处理，例如，消息本身超过4M,则会对其进行压缩

- Producer向选择出的Queue所在的Broker发出RPC请求，将消息发送到选择出的Queue



#### Queue选择算法

有序消息可以指定queue，无序消息则由系统自己选择

对于无序消息，其Queue选择算法也称为消息投递算法，常见的有两种：

**轮询算法**：

默认选择算法。该算法保证了每个Queue中可以均匀的获取到消息

> 该算法存在一个问题:由于某些原因，在某些Broker. 上的Queue可能投递延迟较严重
>
> 从而导致Producer的缓存队列中出现较大的消息积压，影响消息的投递性能

**最小投递延迟算法**：

该算法会统计每次消息投递的时间延迟，然后根据统计出的结果将消息投递到时间延迟最小的Queue

如果延迟相同，则采用轮询算法投递。

> 如果存在一个queue一直延迟最低，则所有消息都会堆积到该queue中，分配不均，导致压力暴增



### 消息的储存

RocketMQ中的消息存储在本地文件系统中，这些相关文件默认在当前用户主目录下的store目录中。

![在这里插入图片描述](https://img-blog.csdnimg.cn/b8e439630af84459b77041cfe7dfd7cd.png)



- abort：该文件在Broker启动后会自动创建，正常关闭Broker,该文件会自动消失。若在没有启动Broker的情况下，发现这个文件是存在的，则说明之前Broker的关闭是非正常关闭。
- checkpoint：其中存储着commitlog、 consumequeue、 index文件的最后刷盘时间戳
- commitlog：其中存放着commitlog文件，而消息是写在commitlog文件中的
- config：存放着Broker运行期间的一些配置数据
- consumequeue：其中存放着consumequeue文件，队列就存放这个目录中
- index：其中存放着消息索引文件indexfile
- lock：运行期间使用到的全局资源锁



RocketMQ中，无论是消息本身还是消息索引，都是存储在磁盘上的。其不会影响消息的消费吗?

当然不会。其实RocketMQ的性能在目前的MQ产品中性能是非常高的。因为系统通过一系列相关机制大大提升了性能。

首先，RocketMQ对文件的读写操作是通过mmap零拷贝进行的，将对文件的操作转化为直接对内存地址进行操作，从而极大地提高了文件的读写效率。

其次，consumequeue中的数据是顺序存放的，还引入了PageCache的预读取机制，使得对consumequeue文件的读取几乎接近于内存读取，即使在有消息堆积情况下也不会影响性能。

RocketMQ中可能会影响性能的是对commitlog文件的读取。因为对commitlog文件来说， 读取消息时会产生大量的随机访问，而随机访问会严重影响性能。

不过，如果选择合适的系统IO调度算法，比如设置调度算法为Deadline (采用SSD固态硬盘的话)，随机读的性能也会有所提升。



### indexFile

除了通过通常的**指定Topic进行消息消费**外，RocketMQ还提供了**根据key进行消息查询**的功能。

该查询是通过store目录中的index子目录中的indexFile进行索引实现的快速查询。

当然，这个indexFile中的索引数据是在包含了key的消息被发送到Broker时写入的。如果消息中没有包含key,则不会写入。

每个Broker中会包含一组indexFile, 每个indexFile都是以一 个时间戳命名的(这个indexFile被创建时的时间戳)。每个indexFile文件由三部分构成：

- indexHeader
- slots槽位
- indexes索引数据

每个indexFile文件中包含500w个slot槽。而每个slot槽又可能会挂载很多的index索引单元。

![在这里插入图片描述](https://img-blog.csdnimg.cn/67a40b4309924dbb982b1dccd5e84de0.png)

indexHeader固定40个字节，其中存放着如下数据：

![在这里插入图片描述](https://img-blog.csdnimg.cn/f9076767c48f4fb081ec3d41cc292f41.png)

- beginTimestamp:该indexFile中第一 条消息的存储时间
- endTimestamp:该indexFile中最后一 -条消息存储时间
- beginPhyoffset:该indexFile中第一 条消息在commitlog中的偏移量commitlog offset
- endPhyoffset:该indexFile中最后一 条消息在commitlog中的偏移量commitlog offset
- hashSlotCount:已经填充有index的slot数量（并不是每个slot都有挂载的index索引单元，这里统计的是所有已经挂载了的index索引单元的数量）
- indexCount:该indexFile中包含的索引个数



indexFile中最复杂的是Slots与Indexes间的关系。在实际存储时，Indexes 是在Slots后面的，但为了便于理解，将它们的关系展示为如下形式:

### 消息的消费

消费者从Broker中获取消息的方式有两种：

- pull拉取方式
- push推动方式

消费者组对于消息消费的模式又分为两种：

- 集群消费Clustering
- 广播消费Broadcasting



#### 推拉获取消费类型



**pull拉取模式**：

Consumer主动从Broker中拉取消息，主动权由Consumer控制。一旦获取了批量消息，就会启动消费过程

不过，该方式的实时性较弱，即Broker中有了新的消息时消费者并不能及时发现并消费



**push推送模式**：

该模式下Broker收到数据后会主动推送给Consumer。该消费模式一般实时性较高。

该消费类型是典型的**发布-订阅**模式，即Consumer向其关联的Queue注册了监听器，一旦发现有新的消息到来就会触发回调的执行，回调方法是Consumer去Queue中拉取消息。

而这些都是基于Consumer与Broker间的长连接的。长连接的维护是需要消耗系统资源的。



**对比**：

- pull：需要应用去实现对关联Queue的遍历，实时性差；但便于应用控制消息的拉取（想拉就拉）
- push：封装了对关联Queue的遍历，实时性强，但会占用较多的系统资源



#### 消费模式



**广播消费**

![在这里插入图片描述](https://img-blog.csdnimg.cn/c396eb77f6c649009e3f048bdb34c35f.png)

广播消费模式下，相同Consumer Group的每个Consumer实例都接收同一个Topic的全量消息

即：每条消息都会被发送到Consumer Group中的==每个==Consumer



**集群消费**

集群消费模式下，相同Consumer Group的每个Consumer实例**平均分摊**同一个Topic的消息。

即：每条消息只会被发送到Consumer Group中的==某个==Consumer



**广播和集群模式的对比**

- 广播模式：

  - 消费进度保存在**consumer端**。因为广播模式下consumer group中每个consumer都会消费所有消息，但它们的消费进度是不同。所以consumer各自**保存各自的消费进度**。

- 集群模式：

  - 消费进度保存在**broker**中。consumer group中的所有consumer共同消费同一个Topic中的消息，同一条消息只会被消费一次。消费进度会参与到了消费的负载均衡中，故消费进度是需要**共享**的。

  

#### Rebalance机制



**什么是Rebalance**：

Rebalance即**再均衡**，指的是，将一个Topic 下的多个Queue在同一个Consumer Group中的多个Consumer间进行重新分配的过程

Rebalance机制的本意是为了**提升消息的并行消费能力**

例如，一个Topic下5个队列，在只有1个消费者的情况下，这个消费者将负责消费这5个队列的消息。如果此时我们增加一一个消费者，那么就可以给其中一个消费者分配2个队列，给另一个分配3个队列，从而提升消息的并行消费能力

![在这里插入图片描述](https://img-blog.csdnimg.cn/d48b0e428365477385f4c0c79833daab.png)



**rebalance的限制**：

由于一个队列最多分配给一个消费者， 因此当某个消费者组下的消费者实例数量大于队列的数量时，多余的消费者实例将分配不到任何队列

**rebalance的缺点**：

- 消费暂停：

  - 在只有一一个Consumer时， 其负责消费所有队列；在新增了一个Consumer后会触发Rebalance的发生。此时原Consumer就需要**暂停队列的消费**,等到这些队列分配给新的Consumer后，这些暂停消费的队列才能继续被消费。

- 消费重复：

  - Consumer 在消费新分配给自己的队列时，必须接着之前Consumer提交的消费进度的offset继续消费。然而默认情况下，offset是异步提交的，这个异步性导致提交到Broker的offset与Consumer实际消费的消息并不一致。这个不一致的差值就是可能会**重复消费消息**。

  - > 同步提交: consumer提交了其消费完毕的一批消息的offset给broker后， 需要等待broker的成功ACK。当收到ACK后，consumer才会继续获取并消费下一批消息。在等待ACK期间，consumer是阻塞的。
    > 异步提交: consumer提交了其消费完毕的一批消息的offset给broker后， 不需要等待broker的成功ACK。consumer 可以直接获取并消费下一批消息

- 消费突刺：

  - 由于Rebalance可能导致重复消费， 如果需要重复消费的消息过多，或者因为Rebalance暂停时间过长从而导致积压了部分消息。那么有可能会导致在Rebalance结束之后**瞬间需要消费很多消息**。

**Rebalance产生的原因**
导致Rebalance产生的原因，无非就两个：

- 消费者所订阅的Queue数量发生变化
- 消费者组中消费者的数量发生变化。

**Rebalance过程**

在Broker中维护着多个Map集合，这些集合中动态存放着当前Topic中Queue的信息、Consumer Group中Consumer实例的信息。

一旦发现消费者所订阅的Queue数量发生变化，或消费者组中消费者的数量发生变化，立即向Consumer Group中的每个实例发出Rebalance通知

> TopicConfigManager: key是topic 名称，value是TopicConfig. TopicConfig 中维护着该Topic中所有Queue的数据。
> ConsumerManager: key是Consumer Group Id, value 是ConsumerGroupInfo。ConsumerGroupInfo中维护着该Group中所有Consumer实例数据。
>
> ConsumeroffsetManager: key为 Topic与订阅该Topic c的Group的组合，value是一 个内层Map。内层Map的key为QueueId,内层Map的value 为该Queue的消费进度offset。

Consumer实例在接收到通知后会采用Queue分配算法自己获取到相应的Queue,即由Consumer实例自主进行Rebalance



#### Queue分配算法

一个Topic中的Queue只能由Consumer Group中的一个Consumer进行消费，而一个Consumer可以同时消费多个Queue中的消息。那么Queue与Consumer间的配对关系是如何确定的

即Queue要分配给哪个Consumer进行消费，也是有算法策略的。

常见的有四种策略，这些策略是通过在创建Consumer时的构造器传进去的。



**平均分配策略**

![在这里插入图片描述](https://img-blog.csdnimg.cn/fd90f00c8f7c45119203d41db23c253b.png)

该算法是要根据`avg = QueueCount / ConsumerCount`的计算结果进行分配的。如果能够整除，则按顺序将avg个Queue逐个分配Consumer.如果不能整除，则将多余出的Queue按照Consumer顺序逐个分配。



**环形平均策略**

![在这里插入图片描述](https://img-blog.csdnimg.cn/557c7acf3901481197993ded94a8d1e0.png)

环形平均算法是指，根据消费者的顺序，依次在由queue队列组成的环形图中逐个分配。

即：直接轮询一个一个来



**一致性hash策略**

![在这里插入图片描述](https://img-blog.csdnimg.cn/408fd1a288674adc83fbdddf4c38bca9.png)

该算法会将consumer的hash值作为Node节点存放到hash环上，然后将queue的hash值也放到hash环上，
通过顺时针方向，距离queue最近的那个consumer就是该queue要分配的consumer。

> 存在问题：分配不均



**同机房策略**

![在这里插入图片描述](https://img-blog.csdnimg.cn/cdcb4939e0d442ae9ae8d6512b50b23a.png)

该算法会根据queue的部署机房位置和consumer的位置，过滤出当前consumer相同机房的queue

然后按照**平均分配策略**或**环形平均策略**对同机房queue进行分配

如果没有同机房queue，则按照平均分配策略或环形平均策略对所有queue进行分配



#### 至少一次原则

RocketMQ有一个原则：**每条消息必须要被成功消费一次**
那么什么是成功消费呢？

 Consumer在消费完消息后会向其**消费进度记录器**提交其消费消息的offset，offset被成功记录到记录器中，那么这条消费就被成功消费了。

> 什么是消费进度记录器？
>
> 对于广播模式来说，Consumer本身就是消费进度记录器
>
> 对于集群模式来说，Broker是消费进度记录器



![在这里插入图片描述](https://img-blog.csdnimg.cn/c2425e9e1e2444c3a81b899bd22f0a97.png)



### 订阅关系的一致性

订阅关系的一致性指的是

> 同一个消费者组(Group ID相同)下所有Consumer实例所订阅的Topic与Tag及对消息的处理逻辑必须完全一致。否则，消息消费的逻辑就会混乱，甚至导致消息丢失



#### 正确订阅关系

多个消费者组订阅了多个Topic，并且每个消费者组里的多个消费者实例的订阅关系保持了一致

![在这里插入图片描述](https://img-blog.csdnimg.cn/01ce8e010bb44dfdb0759fee24a11971.png)



#### 错误订阅关系

一个消费者组订阅了多个Topic，但是该消费者组里的多个Consumer实例的订阅关系并没有保持一致。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d0b5fa9015f74192a8701884e0b295b3.png)

- **订阅了不同的Topic**
  - 同一个消费者组中的两个Consumer实例订阅了不同的Topic
- **订阅了不同的Tag**
  - 同一个消费者组中的两个Consumer订阅了相同Topic的不同Tag。
- **订阅了不同数量的Topic**
  - 同一个消费者组中的两个Consumer订阅了不同数量的Topic。



### offset管理

> 这里的offset指的是Consumer中的消费进度offset

消费进度offset是用来`记录每个Queue的不同消费组中每个consumer的消费进度`的

根据消费进度记录器的不同，可以分为两种模式：**本地模式**和**远程模式**

广播模式下，同消费组的消费者相互独立，消费进度要单独存储

集群模式下，同一条消息只会被同一个消费组消费一次，消费进度会参与到负载均衡中，故消费进度是需要共享的 

#### offset本地管理模式

当消费模式为广播消费时，offset使用本地模式存储。因为每条消息会被所有的消费者消费，每个消费者管理自己的消费进度，各个消费者之间不存在消费进度的交集。

相关数据以json形式，储存在Consumer的本地磁盘下

#### offset远程管理模式

当消费模式为集群消费时，offset使用远程模式管理。因为所有Consumer实例对消息采用的是均衡消费,所有Consumer共享Queue的消费进度。

相关数据以json形式，储存在Broker的本地磁盘下

当发生Rebalance时， 新的Consumer会从Broker中获取到相应的数据来继续消费

#### offset用途

消费者是如何从最开始持续消费消息的?消费者要消费的第一条消息的起始位置是用户自 己通过consumer. setConsumeFromWhere()方法指定的。
在Consumer启动后，其要消费的第一条消息的起始位置常用的有三种，这三种位置可以通过枚举类型常量设置。这个枚举类型为ConsumeFromWhere。

![在这里插入图片描述](https://img-blog.csdnimg.cn/12b1b477833f4dee98dcaf91b023edb0.png)

当消费完一-批消息后，Consumer会提交其消费进度offset给Broker, Broker在收到消费进度后会将其更新到那个双层Map及consumerOffset.json文件中，然后向该Consumer进行ACK

而ACK内容中包含三项数据：当前消费队列的最小offset (minOffset) 、最大offset (minOffset) 、及下次消费的起始offset (nextBeginOffset) 

> CONSUME_FROM_LAST_OFFSET：从queue的当前最后一条消息开始消费
> CONSUME_FROM_FIRST_ OFFSET：从queue的第一条消息开始消费
> CONSUME_FROM_TIMESTAMP：从指定的具体时间戳位置的消息开始消费
>
> 这个具体时间戳是通过另外一个语句指定的。
> consumer.setConsumeTimestamp(“20210701080000") yyyMMddHHmmss

#### 重试队列

当rocketMQ对消息的消费出现异常时,会将发生异常的消息的offset提交到Broker中的重试队列

系统在发生消息消费异常时会为当前的topic@group创建一个重试队列， 该队列以%RETRY%开头，到达重试时间后进行消费重试

#### 集群模式下offset的同步提交与异步提交

集群消费模式下，Consumer消费完消息后会向Broker提交消费进度offset

提交了offset后，需要继续向broker拿取下一批消息的位置nextBeginOffset

其提交方式分为两种

- **同步提交**：

  消费者在消费完一批消息后会向broker提交这些消息的offset,然后等待broker的成功响应。若在等待超时之前收到了成功响应，则继续读取下一批消息进行消费。若没有收到响应，则会重新提交，直到获取到响应。而在这个等待过程中，消费者是阻塞的。其严重影响了消费者的吞吐量。

- **异步提交**：

  消费者在消费完一-批消息后向broker提交offset ，但无需等待Broker的成功响应，可以继续读取并消费下一批消息。这种方式增加了消费者的吞吐量。但需要注意，broker在收到提交的offset后，还是会向消费者进行响应的。



### 消费幂等

#### 什么是消费幂等？

> 当出现消费者对某条消息重复消费的情况时，重复消费的结果与消费一次的结果是相同的，并且多次消费并未对业务系统产生任何负面影响，那么这个消费过程就是消费幂等的
>
> 幂等：若某操作执行多次与执行次对系统产生的影响是相同的，则称该操作是幂等的。

在互联网应用中，尤其在网络不稳定的情况下，消息很有可能会出现重复发送或重复消费。如果重复的消息可能会影响业务处理，那么就应该对消息做**幂等处理**

#### 消息重复的场景分析

什么情况可能会发生消息的重复消费呢？

**发送消息时重复：应答失败导致重复发送**

当-条消息已被成功发送到Broker并完成持久化，此时出现了网络闪断，从而导致Broker对Producer应答失败

如果此时Producer意识到消息发送失败并尝试再次发送消息，此时Broker中就可能会出现两条内容相同并且Message ID也相同的消息，那么后续Consumer就一定会消费两次该消息。

**消费时消息重复：应答失败导致重复投递消费**

消息已投递到Consumer并完成业务处理,当Consumer给Broker反馈应答时网络闪断， Broker没有接收到消费成功响应。为了保证消息至少被消费一次的原则， Broker将在网络恢复后再次尝试投递之前已被处理过的消息。此时消费者就会收到与之前处理过的内容相同、Message ID也相同的消息。

**rebalance时消息重复**

当Consumer Group中的Consumer数量发生变化时，或其订阅的Topic的Queue数量发生变化时，会触发Rebalance,此时Consumer可能会收到曾经被消费过的消息。

#### 通用解决方案

**两要素**
幂等解决方案的设计中涉及到两项要素：**幂等令牌**与**唯一性处理**。

只要充分利用好这两要素，就可以设计出好的幂等解决方案。

- 幂等令牌：是生产者和消费者两者中的既定协议，**通常指具备唯一业务标识**的字符串。（订单号、流水号）一般由生产者Producer传消息时一起传递
- 唯一性处理：服务端通过采用一定的**算法策略**，保证同一个业务逻辑不会被重复执行成功多次。例如：一个订单的多次支付操作，只会成功一次



**解决方案**
对于常见的系统，幂等性操作的通用性解决方案是:

1. 首先通过**缓存去重**。在缓存中如果已经存在了某幂等令牌，则说明本次操作是重复性操作；若缓存没有命中，则进入下一步
2. 在唯一性处理之前， 先在数据库中**查询幂等令牌作为索引的数据是否存在**。若存在，则说明本次操作为重复性操作；若不存在，则进入下一一步
3. 在**同一事务**中完成项操作: 唯一性处理后，将幂等令牌写入到缓存，并将幂等令牌作为唯一索引的数据写入到DB中

因为缓存具有有效期，所以当然需要加一层db的判断

**以支付场景举例**：

1. 当支付请求到达后，首先在Redis缓存中却获取key为支付流水号的缓存value。若value不空， 则说明本次支付是重复操作，业务系统直接返回调用侧重复支付标识;若value为空，则进入下一步操作
2. 到DBMS中根据支付流水号查询是否存在相应实例。若存在，则说明本次支付是重复操作，业务系统直接返回调用侧重复支付标识:若不存在，则说明本次操作是首次操作，进入下一步完成唯一性处理
3. 在分布式事务中完成三项操作: 

- 完成支付任务（唯一性处理）
- 将当前支付流水号作为key,任意字符串作为value,通过set(key, value, expireTime)将数据写入到Redis缓存

- 将当前支付流水号作为主键，与其它相关数据共同写入到DBMS



#### 消费幂等的实现

消费幂等的解决方案很简单：**为消息指定不会重复的唯一标识**

因为Message ID有可能出现重复的情况，所以真正安全的幂等处理，不建议以Message ID作为处理依据。

最好的方式是**以业务唯一标识作为幂等处理的关键依据**，而业务的唯一标识可以通过消息Key设置。

![在这里插入图片描述](https://img-blog.csdnimg.cn/0463266e063e488f86194f336d6ac133.png)

处理消息的时候就根据业务id去进行缓存和db判断即可

### 消息堆积与消费延迟

#### 概念

消息处理流程中，如果Consumer的消费速度跟不上Producer的发送速度，MQ中未处理的消息会越来越多(进的多出的少)，这部分消息就被称为**堆积消息**

**消息出现堆积，进而会造成消息的消费延迟**

以下场景需要重点关注消息堆积和消费延迟问题：

- 业务系统上下游能力不匹配造成的持续堆积，且无法自行恢复。
- 业务系统对消息的消费实时性要求较高，即使是短暂的堆积造成的消息延迟也无法接受。

一般只需要解决了消息堆积，就不会有消息延迟了

#### 消息拉取

Consumer通过长轮询Pull模式批量拉取的方式从服务端获取消息，将拉取到的消息缓存到本地缓冲队列中。

对于拉取式消费，在内网环境下会有很高的吞吐量,所以这一阶段一般不会成为消息堆积的瓶颈。

**消息消费**

Consumer将本地缓存的消息提交到消费线程中，使用业务消费逻辑对消息进行处理，处理完毕后获取到一个结果。这是真正的消息消费过程。此时Consumer的消费能力就完全依赖于消息的**消费耗时**和**消费并发度**了

如果由于业务处理逻辑复杂等原因，导致处理单条消息的耗时较长，则整体的消息吞吐量肯定不会高，此时就会导致Consumer本地缓冲队列达到上限，停止从服务端拉取消息



**消息堆积的主要瓶颈在于客户端的消费能力，而消费能力由消费耗时和消费并发度决定**

优先解决消费耗时的问题（降低处理时间），再考虑消费并发度（增加线程等）



#### 消费耗时

影响消息处理时长的主要因素是**代码逻辑**

而代码逻辑中可能会影响处理时长的主要有两种类型的代码：**CPU内部计算型代码**和**外部I/O操作型代码**

通常情况下代码中如果没有复杂的递归和循环的话，内部计算耗时相对外部I/0操作来说几乎可以忽略

所以**外部IO型代码是影响消息处理时长的主要症结所在**

#### 消费并发度

一般情况下，消费者端的消费并发度由单节点线程数和节点数量共同决定，其值为单节点线程数节点数量

不过，通常需要优先调整单节点的线程数，若单机硬件资源达到了上限，则需要通过横向扩展来提高消费并发度



### 消息的清理

消息被消费完后会被清理嘛？不会的

消息是被顺序存储在commitlog文件的，且消息大小不定长，所以消息的清理是不可能以消息为单位进行清理的，而是以commitlog文件 为单位进行清理的。否则会极具下降清理效率，并实现逻辑复杂

commitlog文件存在一个过期时间， 默认为72小时，即三天。

除了用户手动清理外，在以下情况下也会被自动清理，无论文件中的消息是否被消费过:

- 文件过期，且到达清理时间点(默认为凌晨4点)后，自动清理过期文件
- 文件过期，且磁盘空间占用率已达过期清理警戒线(默认75%)后，无论是否达到清理时间点，都会自动清理过期文件
- 磁盘占用率达到清理警戒线(默认85%)后，开始按照设定好的规则清理文件，无论是否过期。默认会从最老的文件开始清理

- 磁盘占用率达到系统危险警戒线(默认90%)后，Broker将拒绝消息写入



## RocketMQ应用

切记切记开启防火墙，9876和10911端口！不然会连接不上

### 一、普通消息

**消息发送分类**

Producer对于消息的发送也有许多的选择，不同的方法会产生不同的效果

#### 同步发送消息

同步发送消息是指，Producer发出一条消息后， 会在收到MQ返回的ACK之后才发下一条消息

该方式的消息可靠性最高，但消息发送效率太低

![在这里插入图片描述](https://img-blog.csdnimg.cn/f8c49cc927cf41dca486f4c48a324484.png)

同步消息发送示例

```java
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

/**
 * @Author if
 * @Description: 同步生产者
 * @Date 2021-10-24 下午 05:52
 */
public class SyncProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        //创建一个producer，参数为producerGroup名称
        DefaultMQProducer producer=new DefaultMQProducer("pg");
        //设置name server地址
        producer.setNamesrvAddr("centos:9876");
        //启动生产者
        producer.start();

        //当发送失败时重试发送的次数，默认2次
        producer.setRetryTimesWhenSendFailed(3);
        //设置发送超时时间5000ms（5s），默认3000ms
        producer.setSendMsgTimeout(5000);

        //测试生产并发送100条消息
        for(int i=0;i<100;i++){
            byte[] body=("Hi,"+i).getBytes();
            //创建Message，指定topic、tag和具体内容byte[]类型的body
            Message msg=new Message("someTopic","someTag",body);
            //发送消息并获得result
            SendResult sendResult = producer.send(msg);
            System.out.println(sendResult);
        }
        //关闭producer
        producer.shutdown();
    }
}
```

SendResult类是一个枚举类，有如下状态

```java
public enum SendStatus {
    SEND_OK,			//发送成功
    FLUSH_DISK_TIMEOUT,  //刷盘超时（Broker刷盘策略设置同步刷盘可能产生这个状态）
    FLUSH_SLAVE_TIMEOUT, //slave同步超时（Broker集群设置同步复制可能会导致）
    SLAVE_NOT_AVAILABLE; //从站slave不可用（Broker集群设置同步复制可能会导致）

    private SendStatus() {
    }
}
```

出现下列文字则是消息发送成功`sendStatus=SEND_OK`

> SendResult [sendStatus=SEND_OK, msgId=7F00000112D818B4AAC27A729CB90062, offsetMsgId=C0A8E98000002A9F00000000000667D4, messageQueue=MessageQueue [topic=someTopic, brokerName=centos, queueId=2], queueOffset=49]





#### 异步发送消息

异步发送消息是指，Producer发出消息后无需等待(异步发送响应)MQ返回ACK，直接发送下一条消息

该方式的消息可靠性可以得到保障，消息发送效率也可以

![在这里插入图片描述](https://img-blog.csdnimg.cn/4ddff85ab0034e7d9b6d9799d8075ccb.png)



异步和同步只是发送时选用的方法不同`public void send(Message msg, SendCallback sendCallback)`

而且发送的时候需要TimeUnit.SECONDS.sleep(3);一下，不能让他立即关闭了

需要一个回调类SendCallback

```java
public interface SendCallback {
    void onSuccess(SendResult var1);

    void onException(Throwable var1);
}
```



异步发送消息代码示例

```java
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;
import java.util.concurrent.TimeUnit;

/**
 * @Author if
 * @Description: 异步生产者
 * @Date 2021-10-24 下午 05:52
 */
public class AsyncProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        //创建一个producer，参数为producerGroup名称
        DefaultMQProducer producer=new DefaultMQProducer("pg");
        //设置name server地址
        producer.setNamesrvAddr("centos:9876");
        //启动生产者
        producer.start();

        //设置异步发送不重试
        producer.setRetryTimesWhenSendAsyncFailed(0);
        //设置发送超时时间5000ms（5s），默认3000ms
        producer.setSendMsgTimeout(5000);
        //设定新创建的topic的queue数量为2，默认4
        producer.setDefaultTopicQueueNums(2);

        //测试生产并发送100条消息
        for(int i=0;i<100;i++){
            byte[] body=("Hi,"+i).getBytes();
            //创建Message，指定topic、tag和具体内容byte[]类型的body
            Message msg=new Message("AsyncTopic","AsyncTopic",body);
            //发送消息并获得result
            producer.send(msg, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.println(sendResult);
                }

                @Override
                public void onException(Throwable throwable) {
                    throwable.printStackTrace();
                }
            });
        }
        //这一步的休眠很重要！因为是异步发送，还没发送完就关闭producer了
        TimeUnit.SECONDS.sleep(3);
        //关闭producer
        producer.shutdown();
    }
}
```





#### 单向发送消息

单向发送消息是指，Producer仅负责发送消息， 不等待、不处理MQ的ACK。该发送方式时MQ也不返回ACK

该方式的消息发送效率最高，但消息可靠性较差

![在这里插入图片描述](https://img-blog.csdnimg.cn/eaf2e5d1cdc54663a183a191354416e5.png)



单向发送消息代码示例

```java
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

/**
 * @Author if
 * @Description: 单向消息生产者
 * @Date 2021-10-24 下午 07:21
 */
public class OneWayProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException {
        DefaultMQProducer producer=new DefaultMQProducer("pg");
        producer.setNamesrvAddr("centos:9876");
        producer.start();

        for(int i=0;i<100;i++){
            byte[] body=("Hi,"+i).getBytes();
            Message msg=new Message("OneWayTopic","OneWayTopic",body);
            //单向发送
            producer.sendOneway(msg);
        }
        producer.shutdown();
        System.out.println("producer shutdown");
    }
}
```



#### 消费者消费代码示例

```java
import org.apache.rocketmq.client.consumer.DefaultMQPullConsumer;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;
import java.util.List;

/**
 * @Author if
 * @Description: 消费者
 * @Date 2021-10-24 下午 07:25
 */
public class SomeConsumer {
    public static void main(String[] args) throws MQClientException {
        //定义一个Pull消费者
//        DefaultMQPullConsumer pullConsumer=new DefaultMQPullConsumer("cg");
        //定义一个Push消费者
        DefaultMQPushConsumer consumer=new DefaultMQPushConsumer("cg");
        consumer.setNamesrvAddr("centos:9876");
        //指定从第一条消息开始消费
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
        //订阅消费的topic与tag（此处tag为通配符表示所有）
        consumer.subscribe("someTopic","*");

        //指定消费模式：广播模式，默认为集群模式
        consumer.setMessageModel(MessageModel.BROADCASTING);


        //注册消息监听器
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            //消费消息代码，一旦broker中有了其订阅的消息就会触发该代码的执行
            //返回值为当前消费的状态
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for (MessageExt messageExt : list) {
                    System.out.println(messageExt);
                }
                //CONSUME_SUCCESS消费成功
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        //开启消费
        consumer.start();
        System.out.println("consumer start");
    }
}
```

结果

> MessageExt [brokerName=centos, queueId=1, storeSize=182, queueOffset=43, sysFlag=0, bornTimestamp=1635071931511, bornHost=/192.168.233.1:53685, storeTimestamp=1635071929343, storeHost=/192.168.233.128:10911, msgId=C0A8E98000002A9F000000000006560E, commitLogOffset=415246, bodyCRC=1980703418, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='someTopic', flag=0, properties={MIN_OFFSET=0, MAX_OFFSET=75, CONSUME_START_TIME=1635075197904, UNIQ_KEY=7F00000112D818B4AAC27A729C770049, CLUSTER=DefaultCluster, TAGS=someTag}, body=[72, 105, 44, 55, 51], transactionId='null'}]



我们清楚控制台内容，不关闭消费者，然后开启同步生产者生成消息发送，消费者就会处理刚刚生产的消息

我们可以rocket-console看到所以运行的消费者及其信息状态

![在这里插入图片描述](https://img-blog.csdnimg.cn/0b1677f8a28b4a0fb4b500d7f1bc9f8a.png)

### 二、顺序消息

#### 什么是顺序消息？

顺序消息指的是，严格按照消息的**发送顺序**进行消费的消息

默认情况下生产者会把消息以Round Robin轮询方式发送到不同的Queue分区队列

而消费消息时会从多个Queue上拉取消息，这种情况下的发送和消费是不能保证顺序的



如果将消息仅发送到同一个Queue中，消费时也只从这个Queue上拉取消息，就严格保证了消息的顺序性

#### 为什么需要顺序消息？

用例子说明：假设我们一个订单topic，有4个queue队列，该topic中不同的消息用于描述订单不同的状态

假设订单有4个状态：未支付、已支付、发货中、发货成功、发货失败



根据以上订单状态，生产者从时序上可以生成如下几个消息: 

订单T0000001:未支付-->订单T0000001:已支付-->订单T0000001:发货中-->订单T0000001:发货失败

消息发送到MQ中之后，Queue的选择如果采用轮询策略，消息在MQ的存储可能如下:

![在这里插入图片描述](https://img-blog.csdnimg.cn/5353efa47d9840e4a1a2fe3310a22a93.png)

这种情况下，我们希望Consumer消费消息的顺序和我们发送是一致的， 然而上述MQ的投递和消费方式，我们无法保证顺序是正确的

对于顺序异常的消息，Consumer即使设置有一定的状态容错，也不能完全处理好这么多种随机出现组合情况



基于上述的情况，可以设计如下方案:对于相同订单号的消息，通过-定的策略，将其放置在一个Queue中，然后消费者再采用一定的策略(例如，一个线程独立处理一个queue, 保证处理消息的顺序性)，能够保证消费的顺序性。

![在这里插入图片描述](https://img-blog.csdnimg.cn/7ea687e153804d39b861cad057ce4bb8.png)

#### 消息的有序性分类

根据有序范围的不同，RocketMQ可以严格地保证两种消息的有序性：**分区有序**与**全局有序**

**分区有序**

如果有多个Queue参与，其仅可保证在该Queue分区队列上的消息顺序，则称为分区有序

也就是一个订单选用一个queue

> 如何实现Queue的选择?
>
> 在定Producer时我们可以指定消息队列选择器，而这个选择器是我们自己实现了MessageQueueSelector接口定义的。
>
> 在定义选择器的选择算法时，一般需要使用选择key。 这个选择key可以是消息key也可以是其它数据。但无论谁做选择key，都不能重复，都是唯一的，一般可以用业务id，即例子中的订单id
>
> 一般性的选择算法是， 让选择key（或其hash值）与该Topic所包含的Queue的数量取模，其结果即为选择出的Queue的Queueld
>
> 取模算法存在一个问题：
>
> 不同选择key与Queue数量取模结果可能会是相同的，即不同选择key的消息可能会出现在相同的Queue,即同一个Consumer可能会消费到不同选择key的消息。
>
> 这个问题如何解决? 
>
> 一般性的作法是，从消息中获取到选择key，对其进行判断。
>
> 若是当前Consumer需要消费的消息，则直接消费，否则，什么也不做。这种做法要求选择key要能够随着消息一起被Consumer获取到。此时使用消息key作为选择key是比较好的做法
>
> 以上做法会不会出现如下新的问题呢?
>
> 不属于那个Consumer的消息被拉取走了，那么应该消费该消息的Consumer是否还能再消费该消息呢?同一个Queue中的消息不可能被同一个Group中的不同Cosuner同时消费所以，消费现一个Queue的不同选择key的消息的Consumer一定属于不同的Group。 而不同的Group中的Consumer间的消费是相互隔离的，互不影响的。



![在这里插入图片描述](https://img-blog.csdnimg.cn/e6019f40573549c7bc047f980d8b44ab.png)

分区有序代码示例：两种选择算法

```java
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.MessageQueueSelector;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageQueue;
import org.apache.rocketmq.remoting.exception.RemotingException;
import java.util.List;

/**
 * @Author if
 * @Description: 顺序消息生产者
 * @Date 2021-10-24 下午 08:25
 */
public class OrderedProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        DefaultMQProducer producer=new DefaultMQProducer("pg");
        producer.setNamesrvAddr("centos:9876");
        producer.start();
        for(int i=0;i<100;i++){
            //oderId表示订单id
            int orderId=i;
            byte[] body=("Hi,"+i).getBytes();
            Message msg=new Message("OrderedTopic","OrderedTag",body);
            //将oderId当做消息key并充当选择算法中的选择key
            msg.setKeys(orderId+"");
            //send()方法的第三个参数Object arg我们这使用业务id，即订单id
            //这个arg会传递给选择器
            //该send为同步发送，当然也可以加一个SendCallback将其变为异步发送
            SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                //订单选择器的选择算法
                @Override
                public MessageQueue select(List<MessageQueue> list, Message message, Object o) {
                    //以下是使用消息key作为选择key的选择算法
                    String key=message.getKeys();
                    int id=Integer.parseInt(key);

                    //以下是使用arg作为选择key
//                    int id=(int) o;

                    int index=id % list.size();
                    return list.get(index);
                }
            },orderId);
            System.out.println(sendResult);
        }
        producer.shutdown();
    }
}
```



**全局有序**

当发送和消费参与的Queue只有**一个**时所保证的有序是整个Topic中消息的顺序，称为全局有序

3个方式指定topic的queue数量

1. producer发送时可以指定数量`producer.setDefaultTopicQueueNums(2);`

2. 可视化控制台里手动创建topic时指定queue数量
3. mqadmin命令手动创建topic时指定queue数量



![在这里插入图片描述](https://img-blog.csdnimg.cn/dc4f03c0f25541ef9cfb665286caebd8.png)



全局有序的实现就指定数量就行了



### 三、延时消息

#### 什么是延时消息？

当消息写入到Broker后，在**指定的时长后才可被消费处理的消息**，称为延时消息

采用RocketMQ的延时消息可以实现`定时任务`的功能，而无需使用定时器

典型的应用场景是，电商交易中**超时未支付关闭订单**的场景，12306平台订票超时未支付取消订票的场景。

#### 延时等级

延时消息的延迟时长不支持随意时长的延迟，是通过特定的延迟等级来指定的

延时等级定义在RocketMQ服务端的MessageStoreConfig类中的如下变量中：

![在这里插入图片描述](https://img-blog.csdnimg.cn/5442bf50150440ea95f357388ea596da.png)

若指定延时等级为3，则表示延时为10s（从1开始计数）

如果需要自定义延时等级，可以去conf里修改broker的配置

> messageDelayLevel=1s 5s10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h 1d



#### 延时消息原理

![在这里插入图片描述](https://img-blog.csdnimg.cn/c7bbb4830e274da89ec65e8ae5290ee3.png)

**修改消息**

Producer将消息发送到Broker后，Broker会 首先将消息写入到commitlog文件，然后需要将其分发到相应的consumequeue。

不过，在分发之前，系统会先判断消息中是否带有延时等级`msg.setDelayTimeLevel(3);`

若没有，则直接正常分发

若有，则需要经历一个复杂的过程：

- 修改消息的Topic为SCHEDULE _TOPIC_XXXX
- 根据延时等级，在consumequeue目录中SCHEDULE_TOPIC_XXXX主题下创建出相应的queueId目录与consumequeue文件(如果没有这些目录与文件的话)
- 修改消息索引单元内容。索引单元中的Message Tag HashCode部分原本存放的是消息的Tag的Hash值。现修改为**消息的投递时间**。投递时间是指该消息被重新修改为原Topic后再次被写入到commitlog中的时间。**投递时间=消息存储时间+延时等级时间**。消息存储时间指的是消息被发送到Broker时的时间戳
- 将消息索引写入到SCHEDULE_TOPIC_XXXX主题下相应的consumequeue中

**投递消息服务类**

Broker内部有一个延迟消息服务类ScheuleMessageService,其会消费SCHEDULE TOPIC XXXX中的消息，即按照每条消息的投递时间，将延时消息投递到目标Topic中。

不过，在投递之前会从commitlog中将原来写入的消息再次读出，并将其原来的延时等级设置为0，即原消息变为了一条不延迟的普通消息。然后再次将消息投递到目标Topic中。

**将消息重新写入commitlog**
延迟消息服务类将延迟消息再次发送给了commitlog,并再次形成新的消息索引条目，分发到相应Queue

#### 原理总结

判断有无延迟等级->有则修改原topic为SCHEDULE _TOPIC并填入时间->时间到了将其变为普通消息并投到原topic



#### 延时消费代码示例

DelayConsumer表示消费者，创建并运行

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

/**
 * @Author if
 * @Description: What is it
 * @Date 2021-10-25 下午 05:32
 */
public class DelayConsumer {
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer=new DefaultMQPushConsumer("cg");
        consumer.setNamesrvAddr("centos:9876");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
        consumer.subscribe("DelayTopic","*");
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for (MessageExt messageExt : list) {
                    //输出消费时的时间
                    System.out.println(new SimpleDateFormat("mm:ss").format(new Date()));
                    System.out.println(messageExt);
                }
                //CONSUME_SUCCESS消费成功
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.println("consumer start");
    }
}
```

DelayProducer为延时消息生产者，创建并运行

```java
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @Author if
 * @Description: What is it
 * @Date 2021-10-25 下午 05:29
 */
public class DelayProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        DefaultMQProducer producer = new DefaultMQProducer("pg");
        producer.setNamesrvAddr("centos:9876");
        producer.start();

        for(int i=0;i<100;i++){
            byte[] body=("Hi,"+i).getBytes();
            Message msg=new Message("DelayTopic","DelayTag",body);
            //指定消息延迟等级为3，即10s
            msg.setDelayTimeLevel(3);
            SendResult sendResult = producer.send(msg);
            //输出消息发送的时间
            System.out.println(new SimpleDateFormat("mm:ss").format(new Date()));
            //输出消息返回结果
            System.out.println(" , "+sendResult);
        }
        producer.shutdown();
    }
}
```

查看输出结果，举例

先是生产者发送消息，可以看见时间为34分46秒

> 34:46
>  , SendResult [sendStatus=SEND_OK, msgId=7F000001258818B4AAC27F5E4D880063, offsetMsgId=C0A8E98000002A9F000000000008342E, messageQueue=MessageQueue [topic=DelayTopic, brokerName=centos, queueId=0], queueOffset=99]

然后是消费者消费消息，这时已经是延时结束了，时间为35分16秒

> 35:16
> MessageExt [brokerName=centos, queueId=2, storeSize=235, queueOffset=4, sysFlag=0, bornTimestamp=1635154486499, bornHost=/192.168.233.1:5949, storeTimestamp=1635154495928, storeHost=/192.168.233.128:10911, msgId=C0A8E98000002A9F00000000000844B3, commitLogOffset=541875, bodyCRC=657998117, reconsumeTimes=0, preparedTransactionOffset=0, toString()=Message{topic='DelayTopic', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=DelayTopic, MAX_OFFSET=25, CONSUME_START_TIME=1635154516164, UNIQ_KEY=7F000001258818B4AAC27F5E4CE30011, CLUSTER=DefaultCluster, DELAY=3, WAIT=true, TAGS=DelayTag, REAL_QID=2}, body=[72, 105, 44, 49, 55], transactionId='null'}]



### 四、事务消息

#### 问题引入

这里的一个需求场景是：工行用户A向建行用户B转账1万元

我们可以使用同步消息来处理该需求场景：

![在这里插入图片描述](https://img-blog.csdnimg.cn/994a70a674b84eb39a75891195f565ee.png)

1. 工行系统发送一个给B增款1万元的同步消息M给Broker
2. 消息被Broker成功接收后，向工行系统发送成功ACK
3. 工行系统收到成功ACK后从用户A中扣款1万元
4. 建行系统从Broker中获取到消息M
5. 建行系统消费消息M,即向用户B中增加1万元

> 如果扣款失败，可是消息已经到了mq，只要消息写入成功就可以被消费，就会出现问题

解决思路是，让第1、2、3步具有原子性，要么全部成功，要么全部失败

即消息发送成功后，必须要保证扣款成功。如果扣款失败，则回滚发送成功的消息。而该思路即使用**事务消息**



#### 解决思路

这里要使用**分布式事务**解决方案

![在这里插入图片描述](https://img-blog.csdnimg.cn/1b56f612b6114a5dad774b64b1fd237c.png)

1. 事务管理器TM向事务协调器TC发起指令，开启**全局事务**
2. 工行系统发一个给B增款1万元的事务消息M给TC
3. TC会向Broker发送**半事务消息prepareHalf**，将消息M**预提交**到Broker。此时的建行系统是看不到Broker中的消息M的
4. Broker会将预提交执行结果Report给TC。
5. 如果预提交失败，则TC会向TM上报预提交失败的响应，全局事务结束;如果预提交成功，TC会调用工行系统的回调操作，去完成工行用户A的预扣款1万元的操作
6. 工行系统会向TC发送预扣款执行结果，即**本地事务**的执行状态
7. TC收到预扣款执行结果后，会将结果上报给TM。
8. TM会根据_上报结果向TC发出不同的确认指令
   1. 若预扣款成功(本地事务状态为COMMIT MESSAGE)，则TM向TC发送Global Commit指令
   2. 若预扣款失败(本地事务状态为ROLLBACK_ MESSAGE)，则TM向TC发送Global Rollback指令
   3. 若现未知状态(本地事务状态为UNKNOW)，则会触发工行系统的本地事务状态回查操作。回查操作会将回查结果，即COMMIT MESSAGE或ROLLBACK MESSAGE Report给TC。TC将结果上报给TM, TM会再向TC发送最终确认指令Global Commit或Global Rollback
9. TC在接收到指令后会向Broker与工行系统发出确认指令
   1. TC接收的若是Global Commit指令，则向Broker与工行系统发送Branch Commit指令。此时Broker中的消息M才可被建行系统看到;此时的工行用户A中的扣款操作才真正被确认
   2. TC接收到的若是Global Rollback指令，则向Broker与工行系统发送Branch Rollback指令。此时Broker中的消息M将被撤销;工行用户A中的扣款操作将被回滚



#### 基础

**分布式事务**

分布式事务通俗地说就是，一次操作由若干分支操作组成，这些分支操作分属不同应用，分布在不同服务器上

分布式事务需要保证这些分支操作要么全部成功，要么全部失败

分布式事务与普通事务一样，就是为了保证操作结果的一致性

**事务消息**

RocketMQ提供了类似X/Open XA的分布式事务功能，通过事务消息能达到分布式事务的最终一致

XA是一种分布式事务解决方案，一 种分布式事务处理模式

**半事务消息**

暂不能投递的消息，发送方已经成功地将消息发送到了Broker,但是Broker未收到最终确认指令，此时该消息被标记成‘暂不能投递”状态，即不能被消费者看到

处于该种状态下的消息即半事务消息

**本地事务状态**

Producer回调操作执行的结果为本地事务状态，其会发送给TC,而TC会再发送给TM

TM会根据TC发送来的本地事务状态来决定全局事务确认指令

**消息回查**

消息回查，即重新查询本地事务的执行状态。本例就是重新到DB中查看预扣款操作**是否执行成功**

![在这里插入图片描述](https://img-blog.csdnimg.cn/0cdae779f79540a68680d5741dead3c3.png)

> 常见的两个引发消息回查的原因：
>
> 1. 回调操作返回UNKNWON
> 2. TC没有接收到TM的最终全局事务确认指令



#### XA三剑客

**XA协议**

XA (Unix Transaction)是一种**分布式事务解决方案**，一种分布式事务处理模式，是基于XA协议的

XA协议由Tuxedo (Transaction for Unix has been Extended for Distributed Operation,分布式操作扩展之后的Unix事务系统)首先提出的，并交给X/Open组织，作为资源管理器与事务管理器的接口标准

XA模式中有三个重要组件: TC、TM、RM

**TC**

`事务协调者`，Transaction Coordinator

维护全局和分支事务的状态，驱动全局事务提交或回滚

> RocketMQ中Broker充当着TC事务协调者



**TM**

`事务管理器`，Transaction Manager

定义全局事务的范围：开始全局事务、提交或回滚全局事务。它实际是全局事务的发起者

> RocketMQ中Producer充当着TM事务管理器



**RM**

`资源管理器`，Resource Manager

管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚

> RocketMQ中Broker和Producer均充当着RM资源管理器



#### 注意事项

- 事务消息不支持延时消息
- 对于事务消息要做好幂等性检查，因为事务消息可能不止一次被消费(因为存在回滚后再提交的情况)



#### 事务消息代码示例

创建事务监听器`IcbcTransactionListener`类

```java
import org.apache.commons.lang3.StringUtils;
import org.apache.rocketmq.client.producer.LocalTransactionState;
import org.apache.rocketmq.client.producer.TransactionListener;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.common.message.MessageExt;

/**
 * @Author if
 * @Description: 爱存不存事务监听器
 * @Date 2021-10-25 下午 09:56
 */
public class IcbcTransactionListener implements TransactionListener {
    /**
     * 回调操作
     * 消息预提交成功就会触发该方法的执行，用于执行本地事务
     * 这里模拟一下
     * tagA就是扣款成功
     * tagB就是扣款失败
     * tagC就是不知道成没成功，要执行消息回查
     * @param message
     * @param o
     * @return
     */
    @Override
    public LocalTransactionState executeLocalTransaction(Message message, Object o) {
        System.out.println("预提交消息成功："+message);
        if(StringUtils.equals("TAGA",message.getTags())){
            return LocalTransactionState.COMMIT_MESSAGE;
        }else if(StringUtils.equals("TAGB",message.getTags())){
            return LocalTransactionState.ROLLBACK_MESSAGE;
        }else if(StringUtils.equals("TAGC",message.getTags())){
            return LocalTransactionState.UNKNOW;
        }
        return LocalTransactionState.UNKNOW;
    }

    /**
     * 消息回查方法
     * 这里模拟出来表示回查就返回成功状态
     *
     * 常见的两个引发消息回查的原因：
     *  1. 回调操作返回UNKNWON
     *  2. TC没有接收到TM的最终全局事务确认指令
     * @param messageExt
     * @return
     */
    @Override
    public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {
        System.out.println("执行消息回查："+messageExt.getTags());
        return LocalTransactionState.COMMIT_MESSAGE;
    }
}
```

创建生产者`TransactionProducer`类

```java
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.TransactionMQProducer;
import org.apache.rocketmq.client.producer.TransactionSendResult;
import org.apache.rocketmq.common.message.Message;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * @Author if
 * @Description: What is it
 * @Date 2021-10-25 下午 09:44
 */
public class TransactionProducer {
    public static void main(String[] args) throws MQClientException {
        TransactionMQProducer producer=new TransactionMQProducer("tpg");
        producer.setNamesrvAddr("centos:9876");

        /**
         * 定义一个线程池
         * @param corePoolSize，核心线程数
         * @param maximumPoolSize，最多线程数
         * @param keepAliveTime，保持活跃的时间：当线程池中线程数量大于核心线程数量时，多余空闲线程的存活时长
         * @param unit，时间单位
         * @param workQueue，临时存放任务的工作队列
         * @param threadFactory，线程工厂
         */
        ThreadPoolExecutor executorService = new ThreadPoolExecutor(
                2,
                5,
                100,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(200),
                new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r);
                thread.setName("client-transaction-msg-check-thread");
                return thread;
            }
        });

        //为生产者指定刚刚定义的线程池
        producer.setExecutorService(executorService);
        //为生产者指定任务监听器
        producer.setTransactionListener(new IcbcTransactionListener());
        
        producer.start();
        String[] tags={"TAGA","TAGB","TAGC"};
        for(int i=0;i<3;i++){
            byte[] body=("Hi"+i).getBytes();
            Message msg=new Message("TransactionTopic",tags[i],body);
            //发送事务消息
            //第二个参数用于指定在执行本地事务时需要使用的业务参数
            TransactionSendResult result = producer.sendMessageInTransaction(msg, null);
            System.out.println("发送结果为："+result.getSendStatus());
        }
    }
}
```



启动生产者得到结果

> 预提交消息成功：Message{topic='TransactionTopic', flag=0, properties={TRAN_MSG=true, UNIQ_KEY=7F0000011DC018B4AAC280588D660000, WAIT=true, PGROUP=tpg, TAGS=TAGA}, body=[72, 105, 48], transactionId='7F0000011DC018B4AAC280588D660000'}
> 发送结果为：SEND_OK
>
> 预提交消息成功：Message{topic='TransactionTopic', flag=0, properties={TRAN_MSG=true, UNIQ_KEY=7F0000011DC018B4AAC280588DF80001, WAIT=true, PGROUP=tpg, TAGS=TAGB}, body=[72, 105, 49], transactionId='7F0000011DC018B4AAC280588DF80001'}
> 发送结果为：SEND_OK
>
> 预提交消息成功：Message{topic='TransactionTopic', flag=0, properties={TRAN_MSG=true, UNIQ_KEY=7F0000011DC018B4AAC280588E210002, WAIT=true, PGROUP=tpg, TAGS=TAGC}, body=[72, 105, 50], transactionId='7F0000011DC018B4AAC280588E210002'}
> 发送结果为：SEND_OK
> 执行消息回查：TAGC

可以看到TAGC时，监听器返回了`LocalTransactionState.UNKNOW`表示不知道成没成功，所以执行了消息回查方法



此时我们使用消费者去消费`TransactionTopic`消息

> consumer start
>
> MessageExt [brokerName=centos, queueId=0, storeSize=284, queueOffset=0, sysFlag=8, bornTimestamp=1635170887201, bornHost=/192.168.233.1:18864, storeTimestamp=1635170920413, storeHost=/192.168.233.128:10911, msgId=C0A8E98000002A9F0000000000089736, commitLogOffset=562998, bodyCRC=2099282948, reconsumeTimes=0, preparedTransactionOffset=562706, toString()=Message{topic='TransactionTopic', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=TransactionTopic, TRANSACTION_CHECK_TIMES=1, MAX_OFFSET=1, TRAN_MSG=true, CONSUME_START_TIME=1635171216788, UNIQ_KEY=7F0000011DC018B4AAC280588E210002, CLUSTER=DefaultCluster, PGROUP=tpg, WAIT=true, TAGS=**TAGC**, REAL_QID=0}, body=[72, 105, 50], transactionId='7F0000011DC018B4AAC280588E210002'}]
>
> MessageExt [brokerName=centos, queueId=2, storeSize=258, queueOffset=0, sysFlag=8, bornTimestamp=1635170887015, bornHost=/192.168.233.1:18864, storeTimestamp=1635170886680, storeHost=/192.168.233.128:10911, msgId=C0A8E98000002A9F0000000000089402, commitLogOffset=562178, bodyCRC=321840424, reconsumeTimes=0, preparedTransactionOffset=561380, toString()=Message{topic='TransactionTopic', flag=0, properties={MIN_OFFSET=0, REAL_TOPIC=TransactionTopic, MAX_OFFSET=1, TRAN_MSG=true, CONSUME_START_TIME=1635171216902, UNIQ_KEY=7F0000011DC018B4AAC280588D660000, CLUSTER=DefaultCluster, PGROUP=tpg, WAIT=true, TAGS=**TAGA**, REAL_QID=2}, body=[72, 105, 48], transactionId='7F0000011DC018B4AAC280588D660000'}]

可以看见只消费了正常的TAGA和执行了消息回查的TAGC，而TAGB因为在监听器里触发时返回了失败状态，所以在整体回滚之后，消息就没有被提交到broker，自然就没有被消费者消费



### 五、批量消息

#### 批量发送

查看`DefaultMQPushConsumer`生产者类可以看见，send消息不仅是参数可以为单条Message，也可以说集合类型

`public SendResult send(Collection<Message> msgs)`

**发送限制**

生产者进行消息发送时可以一次发送多条消息，这可以大大提升Producer的发送效率

不过需要注意以下几点:

- 批量发送的消息必须具有**相同的Topic**
- 批量发送的消息必须具有**相同的刷盘策略**
- 批量发送的消息**不能是延时消息与事务消息**



**批量发送大小**

默认情况下，一批发送的消息总大小不能超过4MB字节。如果想超出该值，有两种解决方案：

- 方案一：将批量消息进行拆分，拆分为若干不大于4M的消息集合分多次批量发送
- 方案二：在Producer端与Broker端修改属性
  - Producer端需要在发送之前设置Producer的maxMessageSize属性
  - Broker端需要修改其加载的配置文件中的maxMessageSize属性



生产者通过send()方法发送的Message,并不是直接将Message序列化后发送到网络上的，而是通过这个Message生成了一个字符串发送出去的

这个字符串由四部分构成：**Topic**、 **消息Body**、**消息日志**(占20字节)，及用于**描述消息的一堆属性key-value**

这些属性中包含例如生产者地址、生产时间、要发送的Queueld等。最终写入到Broker中消息 单元中的数据都是来自于这些属性



#### 批量消费

Consumer的MessageListenerConcurrently监听接口的consumeMessage(方法的第一个参数为消息列表，但默认情况下每次只能消费一条消息。

```java
        //注册消息监听器
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            //消费消息代码，一旦broker中有了其订阅的消息就会触发该代码的执行
            //返回值为当前消费的状态
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for (MessageExt messageExt : list) {
                    System.out.println(messageExt);
                }
                //CONSUME_SUCCESS消费成功
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
```





若要使其一次可以消费多条消息， 则可以通过修改Consumer的consumeMessageBatchMaxSize属性来指定

`consumer.setConsumeMessageBatchMaxSize(10);`

默认值为1，,该值不能超过32，即消费者每次可以拉取的消息最多是32条

```java
	/**
     * Batch consumption size 批量消费规模
     */
    private int consumeMessageBatchMaxSize = 1;
```



若要修改一次拉取的最大值，则可通过修改Consumer的pullBatchSize属性来指定

`consumer.setPullBatchSize(32);`

默认值为32

```java
    /**
     * Batch pull size 批量拉取大小
     */
    private int pullBatchSize = 32;
```



**存在的问题**

Consumer的pullBatchSize属性与consumeMessageBatchMaxSize属性是否设置的越大越好？当然不是

- pullBatchSize值设置的越大， Consumer每拉取一-次需要的时间就会越长，且在网络上传输出现问题的可能性就越高。若在拉取过程中若出现了问题， 那么本批次所有消息都需要全部重新拉取。
- consumeMessageBatchMaxSize值设置的越大， Consumer的消息并发消费能力越低，且这批被消费的消息具有相同的消费结果。因为consumeMesageBatchMaxSize指定的一批消息只会使用一个线程进行处理，且在处理过程中只要有一 个消息处理异常，则这批消息需要全部重新再次消费处理。



### 六、消息过滤

消息者在进行消息订阅时，除了可以指定要订阅消息的Topic外，还可以对指定Topic中的消息根据指定条件进行过滤，即可以订阅比Topic更加细粒度的消息类型

对于指定Topic消息的过滤有两种过滤方式：**Tag过滤**与**SQL过滤**



#### Tag过滤

通过consumer的subscribe()方法指定要订阅消息的Tag.如果订阅多个Tag的消息，Tag间使用或运算符`双竖线||`连接。

```java
        DefaultMQPushConsumer consumer=new DefaultMQPushConsumer("cg");
        consumer.subscribe("Topic","TAGA || TAGB || TAGC");
```



#### SQL过滤

SQL过滤是一种通过特定表达式对事先埋入到消息中的**用户属性**进行筛选过滤的方式。

通过SQL过滤，可以实现对消息的复杂过滤。

不过，只有使用**PUSH模式**的消费者才能使用SQL过滤。

SQL过滤表达式中支持多种常量类型与运算符。



支持的常量类型：

- 数值:比如: 123, 3.1415
- 字符:必须用单引号包裹起来，比如: 'abc'
- 布尔: TRUE或FALSE
- NULL:特殊的常量，表示空



支持的运算符：

- 数值比较: >, >=, <，<=，BETWEEN, =
- 字符比较: =，<>, IN
- 逻辑运算: AND, OR, NOT
- NULL判断: IS NULL或者IS NOT NULL



#### 代码示例

先来个**Tag过滤**

定义消费者`FilterConsumer`

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;
import java.util.List;

/**
 * @Author if
 * @Description: What is it
 * @Date 2021-10-25 下午 10:43
 */
public class FilterConsumer {
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer=new DefaultMQPushConsumer("cg");
        consumer.setNamesrvAddr("centos:9876");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
        //消费FilterTopic中的TagA和TagB（TagC过滤）
        consumer.subscribe("FilterTopic","TagA || TagB");
        consumer.setMessageModel(MessageModel.BROADCASTING);

        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for (MessageExt messageExt : list) {
                    System.out.println("消费的tag为："+messageExt.getTags());
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.println("consumer start");
    }
}
```

定义生产者`FilterProducer`

```java
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

/**
 * @Author if
 * @Description: What is it
 * @Date 2021-10-25 下午 10:43
 */
public class FilterProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        DefaultMQProducer producer=new DefaultMQProducer("pg");
        producer.setNamesrvAddr("centos:9876");
        producer.start();

        //创建3中tag，让消费者过滤掉TagC
        String[] tags={"TagA","TagB","TagC"};
        for(int i=0;i<10;i++){
            byte[] body=("Hi,"+i).getBytes();
            String tag=tags[i % tags.length];
            Message msg=new Message("FilterTopic",tag,body);
            SendResult sendResult = producer.send(msg);
            System.out.println(sendResult);
        }
        producer.shutdown();
    }
}
```

可以看见消费者消费的结果为

> consumer start
> 消费的tag为：TagB
> 消费的tag为：TagB
> 消费的tag为：TagB
> 消费的tag为：TagA
> 消费的tag为：TagA
> 消费的tag为：TagA
> 消费的tag为：TagA

也就是说通过`consumer.subscribe("FilterTopic","TagA || TagB");`将TagC过滤掉了（只消费A和B）



再来个**SQL过滤**

创建消费者`SQLFilterConsumer`

```java
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.MessageSelector;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;
import org.apache.rocketmq.common.protocol.heartbeat.MessageModel;

import java.util.List;

/**
 * @Author if
 * @Description: What is it
 * @Date 2021-10-25 下午 10:43
 */
public class SQLFilterConsumer {
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer=new DefaultMQPushConsumer("cg");
        consumer.setNamesrvAddr("centos:9876");
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        //消费SQLFilterTopic，根据MessageSelector.bySql()方法过滤出age在0到6的消息
        consumer.subscribe("SQLFilterTopic", MessageSelector.bySql("age between 0 and 6"));

        consumer.setMessageModel(MessageModel.BROADCASTING);
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
                for (MessageExt messageExt : list) {
                    System.out.println("消费的用户属性为："+messageExt.getProperty("age"));
                }
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        consumer.start();
        System.out.println("consumer start");
    }
}
```

创建生产者`SQLFilterProducer`

```java
import org.apache.rocketmq.client.exception.MQBrokerException;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.exception.RemotingException;

/**
 * @Author if
 * @Description: What is it
 * @Date 2021-10-25 下午 10:43
 */
public class SQLFilterProducer {
    public static void main(String[] args) throws MQClientException, RemotingException, InterruptedException, MQBrokerException {
        DefaultMQProducer producer=new DefaultMQProducer("pg");
        producer.setNamesrvAddr("centos:9876");
        producer.start();

        for(int i=0;i<10;i++){
            byte[] body=("Hi,"+i).getBytes();
            Message msg=new Message("SQLFilterTopic","SQLFilterTopic",body);
            //在message中put用户属性
            msg.putUserProperty("age",i + "");
            SendResult sendResult = producer.send(msg);
            System.out.println(sendResult);
        }
        producer.shutdown();
    }
}
```

查看结果

> consumer start
> 消费的用户属性为：5
> 消费的用户属性为：6
> 消费的用户属性为：0
> 消费的用户属性为：1
> 消费的用户属性为：4
> 消费的用户属性为：3
> 消费的用户属性为：2

证明sql过滤`MessageSelector.bySql("age between 0 and 6")`成功



如果运行报错

> CODE: 1 DESC: The broker does not support consumer to filter message by SQL9

可能是broker不支持属性过滤，需要在conf中修改broker.conf文件，加上`enablePropertyFilter=true `，且在运行broker时用`-c`命令指定conf文件，即可解决



### 七、消息发送重试机制

#### 什么是消息发送重试机制

Producer对发送失败的消息进行重新发送的机制，称为**消息发送重试机制**，也称为**消息重投机制**

对于消息重投，需要注意以下几点：

- 生产者在发送消息时，若采用**同步**或**异步**发送方式，发送失败**会重试**，但**单向消息**发送方式发送失败是**没有**重试机制的
- 只有普通消息具有发送重试机制，顺序消息是没有的
- 消息重投机制可以保证消息尽可能发送成功、不丢失，但可能会造成消息重复。消息重复在RocketMQ中是无法避免的问题
- 消息重复在一般情况下不会发生I当出现消息量大、网络抖动，消息重复就会成为大概率事件
- producer主动重发、consumer负载变化也会导致重复消息
- 消息重复无法避免，但要避免消息的重复消费。
- 避免消息重复消费的解决方案是，为消息添加唯一标识，使消费者对消息进行消费判断来避免重复消费
- 消息发送重试有三种策略可以选择:同步发送失败策略、异步发送失败策略、消息刷盘失败策略



#### 同步发送失败策略

对于普通消息，消息发送**默认采用轮询round robin策略**来选择所发送到的队列。

如果发送失败，默认重试2次。但在重试时是不会选择上次发送失败的Broker，而是选择其它Broker。

```java
        //当发送失败时重试发送的次数，默认2次
        producer.setRetryTimesWhenSendFailed(3);
```

同时，Broker还具有失败隔离功能，使Producer尽量选择未发生过发送失败的Broker作为目标Broker。

如果超过重试次数，则抛出异常，由Producer去保证消息不丢。 当然当生产者出现RemotingException、MQClientException和MQBrokerException时，Producer会自动重投消息。



#### 异步发送失败策略

异步发送失败重试时，异步重试不会选择其他broker, 仅在**同一个**broker上做重试，所以该策略无法保证消息不丢。

```java
        //设置异步发送不重试
        producer.setRetryTimesWhenSendAsyncFailed(0);
```



#### 消息刷盘发送失败策略

消息刷盘超时（master或slave不可用）或slave不可用(返回状态非SEND OK)时，默认是不会将消息尝试发送到其他Broker的。

不过，对于重要消息可以通过在Broker的配置文件设置retryAnotherBrokerWhenNotStoreOK属性为true来开启。



### 八、消息消费重试机制

#### 顺序消息的消费重试

对于顺序消息，当Consumer消费消息失败后，为了保证消息的顺序性，其会自动**不断地**进行消息重试，直到消费成功。重试期间应用会出现消息消费**被阻塞**的情况。

> 由于对顺序消息的重试是无休止的，不间断的，直至消费成功，所以对于顺序消息的消费，务必要保证应用能够及时监控并处理消费失败的情况，避免消费被永久性阻塞

```java
        //顺序消息消费失败的消费重试时间间隔，默认为1000亳秒，其取值范围为[10,30000]毫秒
        consumer.setSuspendCurrentQueueTimeMillis(100);
```



#### 无序消息的消费重试

对于无序消息，当Consumer消费消息失败时，可以通过设置返回状态达到消息重试的效果。不过需要注意，无序消息的重试**只对集群消费方式生效**，**广播消费方式不提供失败重试**特性

即对于广播消费，消费失败后，失败消息不再重试，继续消费后续消息。



#### 消费重试次数与间隔

对于**无序消息集群消费**下的重试消费，每条消息**默认最多重试16次**

```java
        //设定消息重试最多10次
        consumer.setMaxReconsumeTimes(10);
```

对于这个自定义次数：

> 若修改值小于16，则按照指定间隔进行重试
> 若修改值大于16，则超过16次的重试时间间隔均为2小时
>
> 对于Consumer Group,若仅修改了-个Consumer的消费重试次数，则会应用到该Group中所有其它Consumer实例
>
> 若出现多个Consumer均做了修改的情况，则采用覆盖方式生效。即最后被修改的值会覆盖前面设置的值

但每次重试的间隔时间是不同的，会逐渐变长。每次重试的间隔时间如下表。

![在这里插入图片描述](https://img-blog.csdnimg.cn/1d2fbc9d41f94420b7b80f6f671cb529.png)



#### 重试队列

对于需要重试消费的消息，并不是Consumer在等待了指定时长后再次去拉取原来的消息进行消费

而是将这些需要重试消费的消息放入到了一个特殊Topic的队列中，而后进行再次消费的

这个特殊的队列就是重试队列

当出现需要进行重试消费的消息时Broker会为每个消费组都设置一个Topic名称为`%RETRY%consumerGroup@consumerGroup`的重试队列



#### 消费重试的配置方式

集群消费方式下，消息消费失败后若希望消费重试，则需要在消息监听器接口的实现中明确进行如下三种方式之一的配置：

- 方式1:返回ConsumeConcurrentlyStatus.RECONSUME_LATER
  - `return ConsumeConcurrentlyStatus.RECONSUME_LATER;`
- 方式2:返回Null
- 方式3:抛出异常



### 消费不重试配置文件


集群消费方式下，消息消费失败后若不希望消费重试，则在捕获到异常后同样也返回与消费成功后的相同的结果，即ConsumeConcurrentlyStatus.CONSUME_SUCCESS，则不进行消费重试。

`return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;`



### 九、死信队列

#### 什么是死信队列

当一条消息初次消费失败，消息队列会自动进行消息重试

达到最大重试次数后，若消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时消息队列不会立刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中



这个队列就是**死信队列**(Dead-Letter Queue, DLQ)

其中的消息则称为**死信消息**(Dead-Letter Message, DLM)



#### 死信队列的特征

- 死信队列中的消息不会再被消费者正常消费
- 死信存储有效期与正常消息相同，均为3天，3天后会被自动删除
- 死信队列就是一个特殊的Topic, 名称为%DLQ%consumerGroup@consumerGroup
- 如果一个消费者组未产生死信消息，则不会为其创建相应的死信队列



#### 死信队列的处理

实际上，当一条消息进入死信队列，就意味着系统中某些地方出现了问题，从而导致消费者无法正常消费该消息，比如代码中原本就存在Bug

因此，对于死信消息，通常需要开发人员进行特殊处理。最关键的步骤是要排查可疑因素，解决代码中可能存在的Bug.然后再将原来的死信消息再次进行投递消费



