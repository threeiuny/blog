---
title: Java线程池
date: 2023-07-22 20:34:48
tags: [并发编程, Java线程池, 线程池]
---

## 线程池

池化技术是一种资源管理技术，它可以在应用程序启动时创建一定数量的资源，并将它们保存在一个池中。当应用程序需要使用资源时，可以从池中获取一个空闲的资源来使用，使用完毕后资源会返回到池中，等待下一次使用。池化技术可以提高应用程序的性能和可靠性，避免资源的浪费和性能的下降。

线程池解决的核心问题就是资源管理问题。在并发环境下，系统不能够确定在任意时刻中，有多少任务需要执行，有多少资源需要投入。这种不确定性将带来以下若干问题：

* 频繁申请/销毁资源和调度资源，将带来额外的消耗，可能会非常巨大。

* 对资源无限申请缺少抑制手段，易引发系统资源耗尽的风险。

* 系统无法合理管理内部的资源分布，会降低系统的稳定性。

<!-- more -->

### JDK线程池的核心概念

Java中的线程池核心实现类是ThreadPoolExecutor，ThreadPoolExecutor实现的顶层接口是Executor，顶层接口Executor提供了一种思想：将任务提交和任务执行进行解耦。用户无需关注如何创建线程，如何调度线程来执行任务，用户只需提供Runnable对象，将任务的运行逻辑提交到执行器(Executor)中，由Executor框架完成线程的调配和任务的执行部分。

ThreadPoolExecutor核心参数如下：

* corePoolSize：核心线程数，线程池初始化时创建的线程数量，即使线程处于空闲状态，也不会被销毁。

* maximumPoolSize：最大线程数，线程池允许创建的最大线程数量。

* keepAliveTime：线程空闲时间，当线程空闲时间达到keepAliveTime时，线程会被销毁，直到线程数量等于corePoolSize。

* unit：线程空闲时间的单位。

* workQueue：工作队列，用于存放提交的等待执行任务。

* threadFactory：线程工厂，用于创建线程，可以自定义线程的名称、优先级等。

* handler：拒绝策略，当线程池中的线程数量达到最大值并且队列已满时，线程池无法执行新提交的任务，会调用拒绝策略来处理该任务。

线程池在内部实际上构建了一个生产者消费者模型，将线程和任务两者解耦，并不直接关联，从而良好的缓冲任务，复用线程。线程池的运行主要分成两部分：任务管理、线程管理。任务管理部分充当生产者的角色，当任务提交后，线程池会判断该任务后续的流转：

* 直接申请线程执行该任务。

* 缓冲到队列中等待线程执行。

* 拒绝该任务。

线程管理部分是消费者，它们被统一维护在线程池内，根据任务请求进行线程的分配，当线程执行完任务后则会继续获取新的任务去执行，最终当线程获取不到任务的时候，线程就会被回收。

![ThreadPoolExecutor运行流程](/images/ThreadPool-01.png)

### 线程池的任务调度

任务调度是线程池的主要入口，当用户提交了一个任务，接下来这个任务将如何执行都是由这个阶段决定的。

当提交一个新任务到线程池时，线程池的处理流程如下：

* 首先检测线程池运行状态，如果不是RUNNING，则直接拒绝，线程池要保证在RUNNING的状态下执行任务。

* 默认情况下，创建完线程池后并不会立即创建线程, 而是等到有任务提交时才会创建线程来进行处理（除非调用prestartCoreThread或prestartAllCoreThreads方法），当线程数小于核心线程数时，每提交一个任务就创建一个线程来执行，即使当前有线程处于空闲状态，直到当前线程数达到核心线程数。

* 当前线程数达到核心线程数时，如果这个时候还提交任务，这些任务会被放到缓冲工作队列里，等到线程处理完了手头的任务后，会来缓冲工作队列中取任务处理。

* 当前线程数达到核心线程数并且缓冲工作队列也满了，如果这个时候还提交任务，则会继续创建线程来处理，直到线程数达到最大线程数。

* 当前线程数达到最大线程数并且缓冲工作队列也满了，如果这个时候还提交任务，则会触发拒绝策略。

* 如果某个线程的空闲时间超过了keepAliveTime，那么将被标记为可回收的，并且当前线程池的当前大小超过了核心线程数时，这个线程将被终止。

![线程池的任务调度流程](/images/ThreadPool-02.png)

### 线程池的缓冲工作队列

任务缓冲模块是线程池能够管理任务的核心部分。线程池的本质是对任务和线程的管理，而做到这一点最关键的思想就是将任务和线程两者解耦，不让两者直接关联，才可以做后续的分配工作。线程池中是以生产者消费者模式，通过一个阻塞队列来实现的。阻塞队列缓存任务，工作线程从阻塞队列中获取任务。

阻塞队列(BlockingQueue)是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。阻塞队列就是生产者存放元素的容器，而消费者也只从容器里拿元素。

![缓冲队列](/images/BlockingQueue.png)

JDK中提供了多种阻塞队列的实现，如下：

|        阻塞队列       |                                                                                                                                           描述                                                                                                                                           |
|:---------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| ArrayBlockingQueue    | 一个用数组实现的有界阻塞队列，此队列按照先进先出（FIFO）的原则对元素进行排序。支持公平锁和非公平锁。                                                                                                                                                                                     |
| LinkedBlockingQueue   | 一个由链表结构组成的有界队列，此队列按照先进先出（FIFO）的原则对元素进行排序。此队列的默认长度为Integer.MAX_VALUE，所以默认创建的该队列有容量危险。                                                                                                                                      |
| PriorityBlockingQueue | 一个支持线程优先级排序的无界队列，默认自然序进行排序，也可以自定义实现compareTo()方法来指定元素的排序规则，不能保证同优先级元素的顺序。                                                                                                                                                  |
| DelayQueue            | 一个实现PriorityBlockingQueue实现延迟获取的无界队列，在创建元素时，可以指定多久才能从当前队列中获取当前元素。只有延时期满后才能从队列中获取元素。                                                                                                                                        |
| SynchronousQueue      | 一个不存储元素的阻塞队列，每一个put操作必须等待take操作，否则不能添加元素。支持公平锁和非公平锁。                                                                                                                                                                                        |
| LinkedTransferQueue   | 一个由链表结构组成的无界阻塞队列。采用预占模式，当队列为空时，消费者线程会生成一个虚拟占位节点（节点元素为null）入队，并等待在此节点上，当后续生产者线程请求添加数据时，如果发现某一个节点是预占节点，生产者线程就不入队列，直接将元素填充到该节点，并唤醒等待的消费者线程取走元素。 |
| LinkedBlockingDeque   | 一个由链表组成的双向阻塞队列。队列头部和尾部都能添加或者移除元素，多线程并发时，可以将锁的竞争最多降到一半。                                                                                                                                                                             |

### 任务拒绝策略

任务拒绝模块是线程池的保护部分，线程池有一个最大的容量，当线程池的任务缓存队列已满，并且线程池中的线程数目达到maximumPoolSize时，就需要拒绝掉该任务，采取任务拒绝策略，保护线程池。

JDK提供了多种任务拒绝策略，如下：

|  拒绝策略                              |  描述                                                                     |
|----------------------------------------|---------------------------------------------------------------------------|
| ThreadPoolExecutor.AbortPolicy         |  丢弃任务并抛出RejectedExecutionException异常。这是线程池默认的拒绝策略。 |
| ThreadPoolExecutor.DiscardPolicy       |  丢弃任务但不抛出异常。                                                   |
| ThreadPoolExecutor.DiscardOldestPolicy |  丢弃队列最前面的任务，然后重新提交被拒绝的任务。                         |
| ThreadPoolExecutor.CallerRunsPolicy    |  由调用线程（提交任务的线程）处理该任务。                                 |

如果上述拒绝策略依然不满足需求，可以自定义拒绝策略，实现RejectedExecutionHandler接口即可。

``` java
public class CustomRejectedExecutionHandler implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        // 自定义拒绝策略，将被拒绝的任务记录到日志中
        System.out.println("Task " + r.toString() + " rejected from " + executor.toString());
    }
}
```

### 线程池的运行状态

线程池运行的状态，并不是用户显式设置的，而是伴随着线程池的运行，由内部来维护。线程池内部使用一个变量维护两个值：运行状态(runState)和线程数量 (workerCount)。在具体实现中，线程池将运行状态(runState)、线程数量 (workerCount)两个关键参数的维护放在了变量ctl(AtomicInteger)中。

ctl是对线程池的运行状态和线程池中有效线程的数量进行控制的一个字段， 它同时包含两部分的信息：线程池的运行状态 (runState) 和线程池内有效线程的数量 (workerCount)，高3位保存runState，低29位保存workerCount，两个变量之间互不干扰。用一个变量去存储两个值，可避免在做相关决策时，出现不一致的情况，不必为了维护两者的一致，而占用锁资源。

线程池的运行状态有5种，分别为：

| 运行状态   | 状态描述                                                                                     |
|------------|----------------------------------------------------------------------------------------------|
| RUNNING    | 线程池处于运行状态，能接受新提交的任务，也能处理阻塞队列中的任务。                           |
| SHUTDOWN   | 线程池处于关闭状态，不再接受新提交的任务，但是会处理阻塞队列中的任务。                       |
| STOP       | 线程池处于停止状态，不再接受新提交的任务，也不会处理阻塞队列中的任务，会中断正在执行的任务。 |
| TIDYING    | 线程池正在关闭状态，并且所有任务都已经执行完毕。                                             |
| TERMINATED | 线程池已经关闭。                                                                             |

### JDK线程池模型

JDK通过对ThreadPoolExecutor的封装，提供了多种线程池，如下：

* Executors.newFixedThreadPool：创建一个固定大小的线程池，这个线程池的核心线程数=最大线程数=nThreads，阻塞队列为LinkedBlockingQueue。

* Executors.newSingleThreadExecutor：创建一个单线程运作的线程池，这个线程池的核心线程数=最大线程数=1，在该线程池上顺序按照顺序执行，在任意给定时间，最多一个任务在执行，阻塞队列为LinkedBlockingQueue。

* Executors.newCachedThreadPool：创建一个可缓存的线程池，这个线程池的核心线程数为0，最大线程数为Integer.MAX_VALUE，阻塞队列为SynchronousQueue，线程空闲时间为60s，线程池中的线程数量不固定，可以根据需求自动的更改线程池中线程的数量。

* Executors.newScheduledThreadPool：创建一个固定大小的线程池，这个线程池的核心线程数为nThreads，最大线程数为Integer.MAX_VALUE，阻塞队列为DelayQueue，这个线程池可以延迟或定时的执行任务。

上述列举的线程池底层都是通过对ThreadPoolExecutor参数的不同组装实现的，如实际过程不满足业务需求，可根据自己的业务需求自行创建对应的线程池。

### 拓展线程池

若想拓展线程池，可重写ThreadPoolExecutor的如下方法：

* beforeExecute()：在执行任务之前调用，可以用来进行一些准备工作，如记录日志、初始化线程局部变量等。

* afterExecute()：在执行任务之后调用，可以用来进行一些清理工作，如释放资源、记录日志等。

* terminated()：在线程池关闭之后调用，可以用来进行一些资源清理工作，如关闭数据库连接、关闭文件流等。

### 线程池监控

要获取Java线程池的运行情况，可通过如下方法：

* getPoolSize()：获取线程池中当前的线程数量。

* getActiveCount()：获取线程池中当前正在执行任务的线程数量。

* getQueue()：获取线程池中的任务队列，然后通过队列的size()方法获取任务队列中的任务数量。

* getCompletedTaskCount()：获取线程池中已经完成的任务数量。

* getTaskCount()：获取线程池中已经提交的任务数量。

* isShutdown()：断线程池是否已经关闭。

* isTerminated()：判断线程池是否已经终止。

* getLargestPoolSize()：获取线程池中曾经同时存在过的最大线程数，即线程池中线程数量的峰值。通过该方法可以了解线程池的最大负载情况，以及是否需要调整线程池的大小。

* getMaximumPoolSize()：获取线程池中允许的最大线程数。通过该方法可以了解线程池的最大容量，以及是否需要调整线程池的大小。

* getKeepAliveTime()：获取线程池中空闲线程的存活时间。当线程池中的线程数量超过核心线程数时，空闲线程的存活时间为keepAliveTime，超过该时间后空闲线程将被回收。通过该方法可以了解线程池中空闲线程的存活时间，以及是否需要调整该时间。

### 线程池动态调整

在项目实际运用中，线程池的大小是一个需要动态调整的参数，如果线程池的大小设置的过小，可能会导致系统无法处理大量的并发请求，从而导致系统性能下降；如果线程池的大小设置的过大，可能会导致系统消耗过多的内存资源，从而导致系统性能下降。因此，线程池的大小需要动态调整，以满足系统的实际需求。

可通过如下方法调整线程池的参数来实现动态化：

* setCorePoolSize(int corePoolSize)：设置线程池的核心线程数。

* setMaximumPoolSize(int maximumPoolSize)：设置线程池的最大线程数。

* setKeepAliveTime(long keepAliveTime, TimeUnit unit)：设置线程池中空闲线程的存活时间。

* setRejectedExecutionHandler(RejectedExecutionHandler handler)：设置线程池的拒绝策略。

* setThreadFactory(ThreadFactory threadFactory)：设置线程池的线程工厂。

### 向线程池提交任务

Java线程池可以通过以下几种方式向线程池提交任务：

* execute()：提交一个Runnable任务，用于执行不需要返回结果的任务。

* submit()：可提交一个Callable或Runnable任务，会返回一个Future对象，可以通过该对象获取任务的执行结果。

``` java
import java.util.concurrent.*;

public class ThreadPoolDemo {
    public static void main(String[] args) {
        // 创建线程池
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                2, // 核心线程数
                4, // 最大线程数
                60, // 空闲线程存活时间
                TimeUnit.SECONDS, // 时间单位
                new ArrayBlockingQueue<>(10), // 任务队列
                new ThreadPoolExecutor.AbortPolicy() // 拒绝策略
        );

        // 使用execute方法提交任务
        executor.execute(() -> {
            System.out.println("Task 1 is running.");
        });

        // 使用submit方法提交任务
        Future<String> future = executor.submit(() -> {
            System.out.println("Task 2 is running.");
            return "Task 2 result.";
        });

        try {
            // 获取任务执行结果
            String result = future.get();
            System.out.println("Task 2 result: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        // 关闭线程池
        executor.shutdown();
    }
}
```

### 另类的线程池-ForkJoinPool

“分治法（Fork-Join）”的基本思想是将一个规模为N的问题分解为K个规模较小的子问题，这些子问题的相互独立且与原问题的性质相同，求出子问题的解之后，将这些解合并，就可以得到原有问题的解。分治法的过程如下图所示：

![Fork-Join](/images/fork-join.jpg)

ForkJoinPool是Java7提供的一个用于并行执行任务的线程池，其主旨是将大任务分成若干小任务，之后再并行对这些小任务进行计算，最终汇总这些任务的结果。它的内部采用“工作窃取”模式，以避免工作线程由于拆分了任务之后的join等待过程，当执行新的任务时它可以将其拆分分成更小的任务执行，并将小任务加到线程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。ForkJoinPool的优势在于，能够充分利用多CPU、多核CPU的优势，将一个任务拆分成多个“小任务”，把多个“小任务”放到多个处理器核心上并行执行，最后将各个“小任务”的结果合并成一个“大任务”的结果。

“工作窃取”调度策略如下（即要实现“工作窃取”）：

* 每一个工作线程维护自己的调度队列中的可运行任务。

* 队列以双端队列的形式被维护，不仅支持LIFO（后进先出）的push和pop操作，还支持FIFO（先进先出）的take操作。

* 对于一个给定的工作线程来说，任务所产生的子任务将会被放入到工作者自己的双端队列中。

* 工作线程使用LIFO的顺序，通过弹出任务来处理队列中的任务。

* 当一个工作线程的本地没有任务去运行的时候，它将使用FIFO的规则尝试随机的从别的工作线程中「窃取」一个任务去运行。

* 当一个工作线程触及了join操作，如果可能的话它将处理其他任务，直到目标任务被告知已经结束（通过isDone方法）。所有的任务都会无阻塞的完成。

* 当一个工作线程无法再从其他线程中获取任务和失败处理的时候，它就会退出（通过yield、sleep或者优先级调整）并经过一段时间之后再度尝试直到所有的工作线程都被告知他们都处于空闲的状态。在这种情况下，他们都会阻塞直到其他的任务再度被上层调用。

ForkJoinPool的“工作窃取”实现如下（基于无界双端队列）：

* 每个工作线程都有自己的工作队列WorkQueue，这是一个双端队列，它是线程私有的。

* ForkJoinTask中fork的子任务，将放入运行该任务的工作线程的队头，工作线程将以LIFO的顺序来处理工作队列中的任务。

* 空闲的线程将从其它线程的队列中的队尾“窃取”任务来执行，以减少竞争。

* 双端队列的操作：push()/pop()仅在其所有者工作线程中调用，poll()是由其它线程窃取任务时调用的。

* 当只剩下最后一个任务时，还是会存在竞争，是通过CAS来实现的。

ForkJoinPool的核心类如下：

* ForkJoinPool：用来执行Task，或生成新的ForkJoinWorkerThread，执行 ForkJoinWorkerThread间的 work-stealing 逻辑。

* ForkJoinTask: 执行具体的分支逻辑，声明以同步/异步方式进行执行，一般使用其实现类类RecursiveAction（无返回值）和RecursiveTask（带返回值）。

* ForkJoinWorkerThread：是ForkJoinPool内的worker thread，执行ForkJoinTask, 内部有ForkJoinPool.WorkQueue来保存要执行的ForkJoinTask。

* ForkJoinPool.WorkQueue：保存要执行的ForkJoinTask。

ForkJoinTask的核心方法如下：

* fork()：将当前任务拆分为多个子任务，并将子任务提交到线程池中异步调度执行。

* join()：等待当前任务的子任务执行完成，并返回子任务的执行结果。如果当前任务没有子任务，则直接返回。

* invoke()：将当前任务提交到线程池中执行，并等待执行完成，并返回执行结果。

* invokeAll()：让第一个任务同步执行，其他任务异步执行(注意：其他任务先fork，第一个任务再invoke)。

* compute()：执行任务的具体逻辑，如果任务没有拆分完成，则继续拆分任务。

ForkJoinPool可以通过以下几种方式向线程池提交任务：

* execute()：提交一个Runnable/ForkJoinTask任务，用于执行不需要返回结果的任务，异步执行。

* submit()：可提交一个Runnable/Callable/ForkJoinTask任务，会返回一个Future对象，异步执行，只有在Future调用get的时候（即获取执行结果时）会阻塞。

* invoke()：提交一个ForkJoinTask任务，会返回执行结果，同步执行，会阻塞。

要监控ForkJoinPool的运行情况，可使用如下方法监控：

* getActiveThreadCount：获取当前活动线程数。活动线程数是指正在执行任务的线程数量。

* getRunningThreadCount：获取当前正在运行任务的线程数。运行中的线程是指正在执行任务的线程，并不包括已经完成或处于等待状态的线程。

* getQueuedSubmissionCount：获取提交但尚未执行的任务数量。如果任务队列中有任务等待执行，该数量将会增加。

* getQueuedTaskCount：获取当前任务队列中的任务总数，包括正在执行的任务和等待执行的任务。

* getStealCount：获取偷取任务的次数。偷取的任务是指一个线程从其他线程的任务队列中获取任务。

* getPoolSize：获取当前线程池中实际活动的线程数，即正在执行任务的线程数。

* getParallelism：获取线程池的并行级别，即线程池中允许同时运行的最大线程数。

ForJoinPool的使用示例如下：

``` java
import java.util.concurrent.RecursiveTask;
import java.util.concurrent.ForkJoinPool;

public class SumCalculator extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 100; // 阈值，控制任务拆分的大小
    private int start;
    private int end;

    public SumCalculator(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if (end - start <= THRESHOLD) {
            // 如果任务大小小于等于阈值，直接计算结果
            int sum = 0;
            for (int i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        } else {
            // 如果任务大小大于阈值，拆分任务成更小的子任务
            int mid = (start + end) / 2;
            SumCalculator leftTask = new SumCalculator(start, mid);
            SumCalculator rightTask = new SumCalculator(mid + 1, end);

            // Fork拆分出的子任务
            leftTask.fork();
            rightTask.fork();

            // Join获取子任务的结果并合并
            return leftTask.join() + rightTask.join();
        }
    }

    public static void main(String[] args) {
        int start = 1;
        int end = 10000;

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        SumCalculator task = new SumCalculator(start, end);
        int result = forkJoinPool.invoke(task);

        System.out.println("Sum from " + start + " to " + end + " is: " + result);
    }
}

```
