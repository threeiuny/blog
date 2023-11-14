---
title: 构建高并发程序-Akka（未完成，待后续学习更新）
date: 2023-08-05 22:44:33
tags: [Akka, Scala, Java, 并发]
---

## 现代并发编程模型的困境

* 面向对象（OOP）的语言中一个显著的特点是封装，然后通过对象提供的一些方法来操作其状态，但是共享内存的模型下，多线程对共享对象的并发访问会造成并发安全问题。一般会采用加锁的方式去解决。但加锁会带来一些问题：

  * 加锁的开销很大，线程上下文切换的开销大

  * 加锁导致线程block，无法去执行其他的工作，被block无法执行的线程，其实也是占据了一种系统资源。

  * 加锁在编程语言层面无法防止隐藏的死锁问题

* Java并发模型（JMM）是通过共享内存来实现。而cpu中会利用局部cache来加速主存的访问，为了解决多线程间缓存不一致的问题，在java中一般会通过使用volatile或者Atmoic来标记变量，通过JMM的happens before机制来保障多线程间共享变量的可见性。因此从某种意义上来说是没有共享内存的，而是通过cpu将cache line的数据刷新到主存的方式来实现可见。
因此与其去通过标记共享变量或者加锁的方式，依赖cpu缓存更新，倒不如每个并发实例之间只保存local的变量，而在不同的实例之间通过message来传递。

* 编程模型异步化之后，还有一个比较大的问题是调用栈转移的问题，当主线程提交了一个异步任务到队列中，Worker thread从队列提取任务执行，调用栈就变成了workthread发起的，当任务出现异常时，处理和排查就变得困难。

## Actor模型

Actor模型是一种并发计算模型，旨在解决多线程编程中的共享状态和同步问题。它是一种用于构建并发、分布式和并行系统的计算模型。Actor模型强调将计算单元视为自主的个体（称为"Actor"），这些个体之间通过消息传递进行通信和协作，而不共享内部状态。

<!-- more -->

以下是Actor模型的一些关键概念和特性：

* **Actor（演员）**：Actor是计算单元的基本单位，可以看作是一个独立的执行实体。每个Actor都有自己的状态、行为和与其他Actor通信的能力。Actor之间通过发送和接收消息来进行通信。

* **消息传递**：Actor之间通过消息传递进行通信。消息可以是任何类型的数据，包括简单的值、对象、命令等。当一个Actor向另一个Actor发送消息时，目标Actor可以根据消息的内容来执行相应的操作。

* **无共享状态**：Actor模型的一个重要特点是每个Actor都有自己的私有状态，其他Actor不能直接访问它。这消除了多线程编程中常见的竞态条件和死锁等问题。

* **并发和并行**：由于Actor之间的独立性，它们可以并发地执行。不同的Actor可以在不同的线程或处理器上同时执行，从而实现真正的并行计算。

* **响应式**：Actor模型适合构建响应式系统，即可以根据外部事件和消息动态地改变行为。Actors可以根据接收到的消息来调整自己的状态和行为。

* **监督和错误处理**：在Actor模型中，一个Actor可以监督其他Actor，并在子Actor出现错误时采取适当的措施。这种方式使系统更具弹性和健壮性。

* **位置透明性**：Actors可以在本地或远程部署，而发送消息的方式是相同的，这实现了位置透明性。这对于构建分布式系统非常有用。

* **扩展性**：由于每个Actor都是独立的计算单元，系统可以根据需要动态地扩展，增加更多的Actor来处理负载。

* **顺序保证**：在Actor模型中，每个Actor处理消息的顺序是保证的。这意味着对于同一个Actor，它会按照接收到消息的顺序依次处理，从而避免了一些并发编程中的问题。

Actor模型提供了一种有效地处理并发性和分布式性的方法。它适用于各种应用，从小型并发任务到大规模分布式系统。在实际应用中，可以使用不同的编程框架和库来实现Actor模型，如Akka（Java/Scala）、Erlang/OTP等。

Actor模型图如下：

![Actor模型](/images/actor-model.png)

**重点理解Actor与传统OOP的不同：Actor不调用方法，而是互相发送消息。发送消息不会将线程的执行权从发送方传输到目标方。Actor可以发送一条消息并继续其他操作，而不是阻塞。Actor与传统的OOP的一个重要区别是消息没有返回值。通过发送消息，Actor将工作委托给另一个Actor，如果它期望返回值，那么发送Actor要么阻塞，要么在同一线程上执行另一个 Actor的工作。**

### Actor模型是如何现代并发编程模型的困境

Actor模型中的抽象主体变为了Actor，Actor内部的执行流程是顺序的，同一时刻只有一个message在进行处理，也就是Actor的内部逻辑可以实现无锁化的编程。Actor和线程数解耦，可以创建很多Actor绑定一个线程池来进行处理，no lock，no block的方式能减少资源开销，并提升并发的性能：

* actor之间可以互相发送message。

* actor在收到message之后会将其存入其绑定的Mailbox中。

* Actor从Mailbox中提取消息，执行内部方法，修改内部状态。

* 继续给其他actor发送message。

Actor工作流程如下所示：

![Actor工作流程](/images/actor-active.jpg)

## Akka

Akka是一组用于设计跨越处理器和网络的可扩展、弹性系统的开源库， 其使用Scala语言编写，用于简化构建高并发、分布式、容错程序的开发。Akka允许你专注于满足业务需求，而不是编写初级代码来提供可靠的行为、容错性和高性能。

Akka提供：

* 不使用原子或锁之类的低级并发构造的多线程行为，甚至可以避免你考虑内存可见性问题。

* 系统及其组件之间的透明远程通信，使你不再编写和维护困难的网络代码。

* 一个集群的、高可用的体系结构，具有弹性、可按需扩展性，使你能够提供真正的反应式系统。

Akka对Actor模型的使用提供了一个抽象级别，使得编写正确的并发、并行和分布式系统更加容易。它提供了一种高层次的抽象，使得开发者可以将精力集中在业务逻辑上，而不是线程和锁上。

### Akka库和模块概述

Akka开源库包含多个模块，每个模块都提供不同的功能。下面是Akka的主要模块：

* **akka-actor**：Akka 的核心库，包含Actor模型的实现，是Akka的核心模块。

* **akka-remote**：提供远程通信功能，允许在不同的JVM中的Actor之间进行通信，使不同计算机上的 Actor 能够无缝地交换消息。

* **akka-cluster**：提供集群功能，允许将多个Actor系统组合成一个集群。

* **akka-cluster-sharding**：提供集群分片功能，允许将Actor分片到集群中的多个节点上。

* **akka-cluster-singleton**：提供集群单例功能，允许在集群中创建单例Actor，确保整个集群中只有一个服务实例在运行。

* **akka-persistence**：提供持久化功能，允许将Actor的状态持久化到存储中，可以在系统重新启动或崩溃时恢复实体/参与者的状态。

* **akka-distributed-data**：提供分布式数据功能，允许在Akka集群中的节点之间共享数据。

* **akka-stream**：提供流处理功能，允许使用流处理的方式来处理数据。

* **akka-http**：提供HTTP功能，允许使用HTTP协议来进行通信。

### 从Demo走进Akka(2.8.3版本)

我们将研究一个简单的Akka应用程序的核心逻辑，以展示如何使用Akka库来开发项目。

#### Actor的体系结构

要使用Akka创建Actor，首先需要引入如下依赖：

```xml
<dependency>
    <groupId>com.typesafe.akka</groupId>
    <artifactId>akka-actor-typed_2.13</artifactId>
    <version>2.8.3</version>
  </dependency>
```

##### Akka的Actor层级

Akka的Actor总是属于父Actor。应用程序可以通过调用ActorContext.spawn()来创建 Actor。与创建一个“独立的” Actor 不同，这会将新Actor作为一个子节点注入到已经存在的树中。事实上，在代码中创建Actor之前，Akka已经在系统中创建了三个Actor 。这些内置的Actor的名字包含guardian，因为他们守护他们所在路径下的每一个子Actor。守护者Actor包括：

* /：根守护者（root guardian）。这是系统中所有Actor的父Actor，也是系统本身终止时要停止的最后一个Actor。

* /user：守护者（user guardian）。这是用户创建的所有Actor的父Actor。使用Akka库创建的每个Actor都将有一个事先准备的固定路径/user/。

* /system：系统守护者（system guardian）。系统创建的所有 Actor 的父Actor。

Akka的Actor层级如下图所示：

![Akka的Actor层级](/images/actor_top_tree.png)

要查看Akka的Actor层级的最简单方法是打印ActorRef实例，如下示例：

``` java
package com.example;

import akka.actor.typed.ActorRef;
import akka.actor.typed.ActorSystem;
import akka.actor.typed.Behavior;
import akka.actor.typed.javadsl.AbstractBehavior;
import akka.actor.typed.javadsl.ActorContext;
import akka.actor.typed.javadsl.Behaviors;
import akka.actor.typed.javadsl.Receive;

class PrintMyActorRefActor extends AbstractBehavior<String> {

  static Behavior<String> create() {
    return Behaviors.setup(PrintMyActorRefActor::new);
  }

  private PrintMyActorRefActor(ActorContext<String> context) {
    super(context);
  }

  @Override
  public Receive<String> createReceive() {
    return newReceiveBuilder().onMessageEquals("printit", this::printIt).build();
  }

  private Behavior<String> printIt() {
    ActorRef<String> secondRef = getContext().spawn(Behaviors.empty(), "second-actor");
    System.out.println("Second: " + secondRef);
    return this;
  }
}

class Main extends AbstractBehavior<String> {

  static Behavior<String> create() {
    return Behaviors.setup(Main::new);
  }

  private Main(ActorContext<String> context) {
    super(context);
  }

  @Override
  public Receive<String> createReceive() {
    return newReceiveBuilder().onMessageEquals("start", this::start).build();
  }

  private Behavior<String> start() {
    ActorRef<String> firstRef = getContext().spawn(PrintMyActorRefActor.create(), "first-actor");

    System.out.println("First: " + firstRef);
    firstRef.tell("printit");
    return Behaviors.same();
  }
}

public class ActorHierarchyExperiments {
  public static void main(String[] args) {
    ActorRef<String> testSystem = ActorSystem.create(Main.create(), "testSystem");
    testSystem.tell("start");
  }
}
```

当代码执行时，输出包括第一个 Actor 的引用，以及匹配printit模式时创建的子 Actor 的引用。输出如下：

``` java
First: Actor[akka://testSystem/user/first-actor#623741820]
Second Actor:Actor[akka://testSystem/user/first-actor/second-actor#1835093621]
```

注意 Actor 引用的结构：

* 两条路径都以akka://testSystem/开头。因为所有Actor的引用都是有效的 URL，所以akka://是协议字段的值。

* 接下来，就像在万维网（World Wide Web）上一样，URL标识系统。在本例中，系统名为testSystem，但它可以是任何其他名称。如果启用了多个系统之间的远程通信，则URL的这一部分包括主机名，以便其他系统可以在网络上找到它。

* 因为第二个Actor的引用包含路径/first-actor/，这个标识它为第一个Actor的子Actor。

* Actor引用的最后一部分，即#1053618476或#-1544706041是一个在大多数情况下可以忽略的唯一标识符。

在了解了Actor层次结构之后，你可能会想为什么需要这个层次结构？创建这个层次结构的意义在哪里？答案是“层次结构的一个重要作用是安全地管理 Actor 的生命周期。”

##### Actor的生命周期

Actor在被创建时就会出现，每当一个 Actor 被停止时，它的所有子 Actor 也会被递归地停止。这种行为大大简化了资源清理，并有助于避免诸如由打开的套接字和文件引起的资源泄漏。

要停止Actor，推荐的模式是在Actor内部返回Behaviors.stopped以停止自身，通常作为对某些用户定义的停止消息的响应或当Actor完成其工作时。 从技术上讲，可以通过从父级调用 context.stop(childRef) 来停止子Actor，但不可能以这种方式停止任意（非子）Actor。

Akka actor API公开了一些生命周期信号(Signal)，例如在actor停止后立即发送PostStop。 此后不会处理任何消息。如下所示：

``` java
class StartStopActor1 extends AbstractBehavior<String> {

  static Behavior<String> create() {
    return Behaviors.setup(StartStopActor1::new);
  }

  private StartStopActor1(ActorContext<String> context) {
    super(context);
    context.getLog().info("first started");

    context.spawn(StartStopActor2.create(), "second");
  }

  @Override
  public Receive<String> createReceive() {
    return newReceiveBuilder()
        .onMessageEquals("stop", Behaviors::stopped)
        .onSignal(PostStop.class, signal -> onPostStop())
        .build();
  }

  private Behavior<String> onPostStop() {
    getContext().getLog().info("first stopped");
    return this;
  }
}

class StartStopActor2 extends AbstractBehavior<String> {

  static Behavior<String> create() {
    return Behaviors.setup(StartStopActor2::new);
  }

  private StartStopActor2(ActorContext<String> context) {
    super(context);
    getContext().getLog().info("second started");
  }

  @Override
  public Receive<String> createReceive() {
    return newReceiveBuilder().onSignal(PostStop.class, signal -> onPostStop()).build();
  }

  private Behavior<String> onPostStop() {
    getContext().getLog().info("second stopped");
    return this;
  }
}
```

##### Actor的失败处理

父Actor和子Actor在他们的生命周期中是相互联系的。当一个Actor失败（抛出一个异常或从接收中冒出一个未处理的异常）时，它将暂时挂起。如前所述，失败信息被传播到父Actor，然后父Actor决定如何处理由子Actor引起的异常。这样，父Actor就可以作为子Actor的监督者（supervisors）。默认的监督策略是停止并重新启动子Actor。如果不更改默认策略，所有失败都会导致重新启动。

如下所示：
  
  ``` java
  class SupervisingActor extends AbstractBehavior<String> {

  static Behavior<String> create() {
    return Behaviors.setup(SupervisingActor::new);
  }

  private final ActorRef<String> child;

  private SupervisingActor(ActorContext<String> context) {
    super(context);
    child = context.spawn(
            // 设置监督策略
            Behaviors.supervise(SupervisedActor.create()).onFailure(SupervisorStrategy.restart()),
            "supervised-actor");
  }

  @Override
  public Receive<String> createReceive() {
    return newReceiveBuilder().onMessageEquals("failChild", this::onFailChild).build();
  }

  private Behavior<String> onFailChild() {
    child.tell("fail");
    return this;
  }
}

class SupervisedActor extends AbstractBehavior<String> {

  static Behavior<String> create() {
    return Behaviors.setup(SupervisedActor::new);
  }

  private SupervisedActor(ActorContext<String> context) {
    super(context);
    context.getLog().info("supervised actor started");
  }

  @Override
  public Receive<String> createReceive() {
    return newReceiveBuilder()
        .onMessageEquals("fail", this::fail)
        .onSignal(PreRestart.class, signal -> preRestart())
        .onSignal(PostStop.class, signal -> postStop())
        .build();
  }

  private Behavior<String> fail() {
    getContext().getLog().info("supervised actor fails now");
    throw new RuntimeException("I failed!");
  }

  private Behavior<String> preRestart() {
    getContext().getLog().info("supervised will be restarted");
    return this;
  }

  private Behavior<String> postStop() {
    getContext().getLog().info("supervised stopped");
    return this;
  }
}
```

#### Akka的消息传递与消息序列

Actor之间的通信是通过消息传递来实现的，消息子系统提供的传递语义通常分为以下类别：

* 至多一次传递（At-most-once delivery）：每一条消息都是传递零次或一次；在更因果关系的术语中，这意味着消息可能会丢失，但永远不会重复。Akka采用“至多一次传递”，其开销最小，不需要将状态保持在发送端或传输机制中。

* 至少一次传递（At-least-once delivery）：可能多次尝试传递每条消息，直到至少一条成功；同样，在更具因果关系的术语中，这意味着消息可能重复，但永远不会丢失。其需要重试以抵消传输损失，这增加了在发送端保持状态和在接收端具有确认机制的开销。

* 恰好一次传递（Exactly-once delivery）：每条消息只给收件人传递一次；消息既不能丢失，也不能重复。其开销最大，除了“至少一次传递”的开销之外，还需要在接收端筛选出重复的传递。

在Akka中 ，对于一对给定的Actor，直接从第一个Actor 发送到第二个Actor 的消息不会被无序接收。此保证仅在与tell运算符直接发送到最终目的地时适用，而在使用中介时不适用。如果Actor A1向A2发送消息M1、M2和M3；Actor A3向A2发送消息M4、M5和M6，那么一定会有如下定论：

* 如果M1传递，则必须在M2和M3之前传递。

* 如果M2传递，则必须在M3之前传递。

* 如果M4传递，则必须在M5和M6之前传递。

* 如果M5传递，则必须在M6之前传递。

* A2 可以看到 A1 的消息与 A3 的消息交织在一起。

* 由于没有保证的传递，任何消息都可能丢失，即不能到达 A2。

Akka的消息传递与消息序列让一个Actor发送的消息有序到达，便于构建易于推理的系统，而另一方面，允许不同Actor发送的消息交错到达，则为Actor系统的有效实现提供了足够的自由度。
