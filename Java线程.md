---
title: Java线程
date: 2023-06-10 11:47:21
tags: Java 多线程
---

## 并发与并行

并发与并行都是多任务处理的方式，他们都可以表示两个或者多个任务一起执行。但并发是指多个任务交替执行，并发是通过CPU时间片轮转的方式实现；而并行是指多个任务同时执行，需要多CPU核心或者多台计算机协同工作才能实现。

## 关于CPU并发分片

CPU并发分片是指在单核CPU上实现多任务处理的一种方式。操作系统会将所有需要执行的任务按照一定的优先级排序，然后将CPU时间分成若干个时间片，每个时间片的长度通常是几十毫秒到几百毫秒不等。然后，操作系统会按照任务的优先级依次将时间片分配给不同的任务，让它们交替执行。当一个任务的时间片用完后，操作系统会将它挂起，然后将CPU时间片分配给下一个任务，以此类推。

通过CPU并发分片，单核CPU可以同时处理多个任务，看起来像是同时执行，但实际上是交替执行的。这种方式可以提高CPU的利用率，让多个任务在单核CPU上得到合理的处理。

## Java内存模型（JMM）

Java内存模型（Java Memory Model，JMM）是Java虚拟机规范中定义的一种抽象的计算机内存模型，它屏蔽了各种硬件和操作系统的内存访问差异，为Java程序在各种平台上的内存访问行为提供了统一的规范。通过这些规则、规范定义了程序中各个变量的访问方式。

jvm运行的程序的实体是线程，而每个线程运行时，都会创建一个工作内存（也叫栈空间），来保存线程所有的私有变量。而JMM内存模型规范中规定所有的变量都存储在主内存中（一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。），而主内存中的变量是所有的线程都可以共享的，而对主内存中的变量进行操作时，必须在线程的工作内存进行操作，首先将主内存的变量copy到工作内存，进行操作后，再将变量刷回到主内存中。所有线程只有通过主内存来进行通信。

### JMM的8种内存交互操作

* lock(锁定)：作用于主内存中的变量，一个变量在同一时间只能被一个线程锁定，即把变量标识为线程独占状态。

* read(读取)：作用于主内存变量，表示把一个变量值从主内存传输到线程的工作内存中，以便下一步的 load 操作使用。

* load(载入)：作用于线程的工作内存的变量，表示把 read 操作从主内存中读取的变量值放到工作内存的变量副本中(副本是相对于主内存的变量而言的)。

* use(使用)：作用于线程的工作内存中的变量，表示把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时就会执行该操作。

* assign(赋值)：作用于线程的工作内存的变量，表示把执行引擎返回的值赋值给工作内存中的变量，每当虚拟机遇到一个给变量赋值的字节码指令时就会执行该操作。

* store(存储)：作用于线程的工作内存中的变量，把工作内存中的一个变量的值传递给主内存，以便下一步的 write 操作使用。

* write(写入)：作用于主内存的变量，表示把 store 操作从工作内存中得到的变量的值放入主内存的变量中。

* unlock(解锁)：作用于主内存的变量，表示把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。

JMM内存交互操作可用下图概括

![JMM内存交互图](/images/JMM.png)

JMM 还规定了以上八种操作需按照如下规则进行：

* 不允许read 和 load、store 和 write 操作之一单独出现，也就是 read 操作后必须 load，store 操作后必须 write。即不允许一个变量从主内存读取了但工作内存不接受，或者从工作内存发起回写了但主内存不接受的情况出现。

* 不允许线程丢弃它最近的 assign 操作，即变量在工作内存中改变了之后必须把该变化同步回主内存。

* 不允许线程将没有 assign 的数据从工作内存同步到主内存。

* 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。也就是对变量实施 use 和 store 操作之前，必须经过 load 和 assign 操作。

* 一个变量同一时间只能有一个线程对其进行 lock 操作。但 lock 操作可以被同一条线程重复执行多次，多次 lock 之后，必须执行相同次数 unlock 才可以解锁。

* 如果对一个变量进行 lock 操作，会清空所有工作内存中此变量的值。在执行引擎使用这个变量前，必须重新 load 或 assign 操作初始化变量的值。

* 如果一个变量没有被 lock，就不能对其进行 unlock 操作。也不能 unlock 一个被其他线程锁住的变量。

* 一个线程对一个变量进行 unlock 操作之前，必须先把此变量同步回主内存。

### JMM三大特征

* 原子性：一个操作是不可中断的。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程打断。

* 可见性：多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

* 有序性：程序执行的顺序按照代码的先后顺序执行。

### 指令重排

为了最大的限度发挥机器性能，jvm根据处理器特性（cpu多级缓存、多核处理器等）适当的对机器指令进行排序，使机器指令能更符合CPU的执行特性，只要程序的最终结果与顺序执行的结果保持一致，则虚拟机就可以进行排序，此过程就叫指令重排序。指令重排遵循as-if-serial语义和happens-before 原则。

#### as-if-serial

as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器、runtime和处理器都必须遵守as-if-serial语义。

为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序。

#### happens-before

* 程序顺序原则：即在一个线程内必须保证语义串行性，也就是说按照代码顺序执行。

* 锁规则：解锁(unlock)操作必然发生在后续的同一个锁的加锁(lock)之前，也就是说，如果对于一个锁解锁后，再加锁，那么加锁的动作必须在解锁动作之后(同一个锁)。

* volatile规则：volatile变量的写，先发生于读，这保证了volatile变量的可见性，简单的理解就是，volatile变量在每次被线程访问时，都强迫从主内存中读该变量的值，而当该变量发生变化时，又会强迫将最新的值刷新到主内存，任何时刻，不同的线程总是能够看到该变量的最新值。

* 线程启动规则：线程的start()方法先于它的每一个动作，即如果线程A在执行线程B的start方法之前修改了共享变量的值，那么当线程B执行start方法时，线程A对共享变量的修改对线程B可见

* 传递性 A先于B ，B先于C 那么A必然先于C

* 线程终止规则：线程的所有操作先于线程的终结，Thread.join()方法的作用是等待当前执行的线程终止。假设在线程B终止之前，修改了共享变量，线程A从线程B的join方法成功返回后，线程B对共享变量的修改将对线程A可见。

* 线程中断规则：对线程 interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过Thread.interrupted()方法检测线程是否中断。

* 对象终结规则：对象的构造函数执行，结束先于finalize()方法。

## 线程不安全

多线程编程最直观的问题就是线程安全问题，当多个线程同时访问共享资源时，可能会出现数据不一致或者其它问题的情况，导致数据出现错误或者不一致的情况。

具体到JMM来看，每个线程都有自己的工作内存（也称之为栈帧，当方法执行完成后，对应的栈帧会被弹出），工作内存中保存了线程局部变量与需要访问的主内存变量的副本，每个线程只能在自己的工作内存中操作变量副本（所以局部变量不存在线程安全问题）。当线程需要访问共享变量时，它会先从主内存中读取变量的值，然后将变量的值赋值到自己的工作内存中。当线程修改变量的值后，需要将修改后的值写回到主内存中，已便其它线程可以看到变量的最新值。由于每个线程都有自己的工作内存，因此多个线程同时访问共享变量时，可能会出现数据不一致的情况。

在多线程编程中，为了避免出现线程不安全的问题，可以使用锁机制控制共享资源的访问，从而避免数据不一致的情况。

## 临界区

多线程编程中，需要对共享变量的访问和操作进行控制。
在Java中，锁机制可以用来控制共享资源的访问，从而避免线程不安全的问题。锁机制可以将多个线程对共享资源的访问串行化，保证在同一时间只有一个线程可以访问共享资源，从而避免了多个线程同时修改共享资源的情况。

在锁机制中，被保护的代码块称为临界区。临界区是指一段代码或方法，在同一时间只能被一个线程执行。当一个线程进入临界区时，它会尝试获取锁，如果锁没有被其它线程占用，那么该线程就可以执行临界区的代码。当线程执行完临界区的代码后释放锁，已便其它线程可以获取锁并执行临界区代码。

## 临界区带来的死锁、饥饿和活锁

Java中的临界区可以保证共享资源的访问是线程安全的，但是也会带来一些问题，例如死锁、饥饿和活锁等。

* 死锁是指两个或多个线程互相持有对方需要的锁，从而导致它们都无法继续执行的情况。例如，线程A持有锁1，等待获取锁2，而线程B持有锁2，等待获取锁1，这样就会导致两个线程都无法继续执行，从而出现死锁的情况。

* 饥饿是指某个线程无法获取所需的资源，从而无法继续执行的情况。例如，如果一个线程一直无法获取所需的锁，那么它就会一直等待，从而无法继续执行，这样就会出现饥饿的情况。

* 活锁是指多个线程在竞争资源时，由于某些原因导致它们一直在重试，但是却无法取得进展的情况。例如，如果多个线程都在等待对方释放锁，那么它们就会一直重试，但是却无法取得进展，这样就会出现活锁的情况。

## 并发级别

由于临界区的存在，多线程之间的并发必须受到控制。根据控制并发的策略，大致上可以分为阻塞、无饥饿、无障碍、无锁、无等待几类。

* 阻塞：线程会被阻塞，直到获取到锁或者资源才能继续执行。synchronized关键字和ReentrantLock类就是阻塞的典型代表。

* 无饥饿：线程不会被饥饿，即每个线程都有机会获取锁或者资源。公平锁就是无饥饿的典型代表。

* 无障碍：线程可以自由地读写共享变量，而不需要加锁。无障碍算法就是无障碍的典型代表。

* 无锁 线程不需要加锁，而是通过CAS（Compare and Swap）等机制来实现同步。Atomic类和ConcurrentHashMap类就是无锁的典型代表。

* 无等待 线程不需要等待其他线程释放锁或者资源，而是通过一些算法来实现同步。无等待算法就是无等待的典型代表。

## Java线程状态

线程是操作系统可调度任务的最小单位，是Java程序中的最小执行单元。Java线程是建立在操作系统线程之上的，一个Java线程对应一个操作系统线程，但是Java线程的调度和管理是由JVM完成的。

Java中线程状态大致分为以下六种状态：

* NEW（初始状态）：线程被创建，但还没有调用start()方法。
* RUNNABLE（就绪（可运行）状态）：处于此状态的线程并不一定处于运行中。Java多线程使用的线程调度策略是抢占式调度，每个可运行线程轮着获取CPU时间片。Java中线程的RUNNABLE状态对应操作系统线程状态中的两种状态，分别是 Running 和 Ready
* BLOCKED（阻塞状态）：线程正等待监视器锁，该锁目前被其它线程持有。
* WAITING（等待状态）：线程不会被分配CPU执行时间，无限期等待，需要被显示的唤醒。
* TIMED_WAITING（超时等待状态）：与WAITING一样处于等待状态，不过不会无限期的等待，在到达指定时间后会自动唤醒。
* TERMINATED（终止状态）：表示当前线程已经执行完毕。

Java线程的状态并不是一层不变的，随着程序的运行，线程的状态会随时切换，线程状态的切换可见下图

![线程状态切换](/images/thread-01.jpg)

## Java线程的基本操作

Java提供了一系列基本方法让我们可以对线程状态进行变更

### 新建线程

Java中可以通过两种方式来新建一个线程

* 继承Thread类并重写run()方法，然后创建该类的实例并调用start()方法来启动线程。

``` java
public class MyThread extends Thread {
    public void run() {
        // 线程执行的代码
    }
}

// 创建线程并启动
MyThread thread = new MyThread();
thread.start();
```

* 实现Runnable接口并重写run()方法，然后创建该类的实例并其作为参数传递给Thread类的构造方法，最后调用start()方法来启动线程。

``` java
public class MyRunnable implements Runnable {
    public void run() {
        // 线程执行的代码
    }
}

// 创建线程并启动
MyRunnable runnable = new MyRunnable();
Thread thread = new Thread(runnable);
thread.start();
```

当然也可以使用lambda表达式简化上述代码

``` java
Thread thread = new Thread(() -> {
    // 线程执行的代码
});
thread.start();
```

需要注意：

* **不要用Thread的run()方法来开启新线程。它只会在当前线程中，串行执行run()方法中的代码**
* **Thread类的run()方法就是直接调用其内部Runnable接口中的run()方法，由于Java是单继承的，因此更推荐使用Runnable接口来开启新线程**

### 终止线程stop（已废弃，不推荐使用）

如果需要提前人为终止正在执行中的线程，可以使用Thread类的stop()方法，但是stop()方法是强制终止线程的执行，可能会导致线程正在执行的任务没有完成，从而导致数据不一致或者其他线程安全问题。因此JDK已经废弃掉stop()方法，不推荐使用。

``` java
public class StopThreadUnSafe {

    public static User user = new User();
    @Getter
    @Setter
    public static class User {
        private int id;
        private String name;
        public User() {
            this.id = 0;
            this.name = "0";
        }
        @Override
        public String toString() {
            return "User{id=" + id + ", name=" + name + "}";
        }
    }

    // 写线程
    public static class ChangeThread implements Runnable {
        @Override
        public void run() {
            while (true) {
                synchronized (user) {
                    int v = (int) (System.currentTimeMillis() / 1000);
                    user.setId(v);
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    user.setName(String.valueOf(v));
                }
                Thread.yield();
            }
        }
    }

    // 读线程，如果持有锁资源，打印ID与name不一致的数据
    public static class ReadThread implements Runnable {
        @Override
        public void run() {
            while (true) {
                synchronized (user) {
                    if (user.getId() != Integer.parseInt(user.getName())) {
                        System.out.println(user);
                    }
                }
                Thread.yield();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        // 创建一个读线程用于监控
        new Thread(new ReadThread()).start();
        // 创建多个写线程
        while (true) {
            Thread change = new Thread(new ChangeThread());
            change.start();
            Thread.sleep(150);
            change.stop();
        }
    }
}
```

强制终止线程可能会导致数据不一致或者其他线程安全问题，因此强烈不推荐在程序中使用stop()方法终止线程，可以使用下文的线程中断来通知线程退出执行。

### 线程中断interrupt

线程中断是一种协作式的线程终止机制，通过向目标线程发送中断信号来通知目标线程退出执行，从而避免了使用stop()方法引起数据不一致或者其它线程安全问题。线程中断并不会使线程立即退出，而是通过协作的方式来通知线程退出执行。

线程中断主要与如下三个方法相关：

* interrupt()：中断线程

* isInterrupted()：判断是否被中断

* interrupted(): 判断是否被中断，并清除当前状态

线程中断机制主要包括以下几个方面：

* 中断标志位：每个线程都有一个中断标志位，当线程被中断时，中断标志位会被设置为true。

* 中断方法：Java提供了两个方法来中断线程，分别是interrupt()方法和isInterrupted()方法。interrupt()方法用于向目标线程发送中断信号，isInterrupted()方法用于检查目标线程的中断标志位是否被设置为true。

* 中断异常：当一个线程被中断时，它可能会抛出InterruptedException异常，可以在catch块中处理该异常来退出线程。

* 中断状态的清除：当一个线程被中断时，它的中断标志位会被设置为true，但是如果线程在执行过程中没有检查中断标志位，那么中断状态就会一直保持，直到线程执行完毕。为了避免这种情况，可以在catch块中调用Thread.interrupted()方法来清除中断状态。

使用线程中断改进线程终止代码

``` java
public class InterruptThreadSafe {

    public static User user = new User();
    @Getter
    @Setter
    public static class User {
        private int id;
        private String name;
        public User() {
            this.id = 0;
            this.name = "0";
        }
        @Override
        public String toString() {
            return "User{id=" + id + ", name=" + name + "}";
        }
    }

    public static class ChangeThread implements Runnable {
        @Override
        public void run() {
            while (true) {
                // 判定是否被中断
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("Interrupted!");
                    break;
                }
                synchronized (user) {
                    System.out.println("执行写线程");
                    int v = (int) (System.currentTimeMillis() / 1000);
                    user.setId(v);
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        System.out.println("Interrupted when sleep! changeThread! isInterrupted = " + Thread.currentThread().isInterrupted());
                        Thread.currentThread().interrupt();
                    }
                    user.setName(String.valueOf(v));
                }
                Thread.yield();
            }
        }
    }

    public static class ReadThread implements Runnable {
        @Override
        public void run() {
            while (true) {
                synchronized (user) {
                    System.out.println("执行读线程");
                    if (user.getId() != Integer.parseInt(user.getName())) {
                        System.out.println(user);
                    }
                }
                Thread.yield();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ReadThread()).start();
        while (true) {
            Thread change = new Thread(new ChangeThread());
            change.start();
            Thread.sleep(150);
            change.interrupt();
        }
    }
}
```

在主线程使用interrupt()向变更线程发送中断信号，在变更线程中使用isInterrupt()判断当前线程是否被中断，如果已经被中断，则线程退出执行。

如果你仔细查看代码，你会发现在捕获线程睡眠抛出的异常时，**使用了interrupt()再次向当前线程发送中断信号**。你可能会好奇，为什么需要在此处再次使用interrupt()向当前线程发送中断信息？为了解答这个问题，我们需要先了解线程睡眠。

### 线程睡眠sleep

线程睡眠可以让当前线程暂停执行一段时间，在Java中，使用Thread.sleep()方法让**当前线程**休眠若干毫秒，如果在睡眠过程中线程被中断，sleep()方法会抛出InterruptedException异常。sleep()方法签名如下：

``` java
public static native void sleep(long millis) throws InterruptedException;
```

而当sleep()方法抛出InterruptedException异常时，此时它会**清除中断标志**（即将中断标志位置为false,在catch代码块中调用Thread.currentThread().isInterrupted()判定时是否中断时，会判定为未中断），如果不进行处理，那么在下一次循环时，就无法正常中断线程，所以在异常处理中再次设置中断标志位。

**当使用Thread.sleep()方法让当前线程休眠时，不会释放持有的锁。**

### 线程等待wait和通知notify

线程等待和通知是一种线程间的协作机制，它可以让一个线程等待另一个线程的通知，从而实现线程间的同步。wait()方法用于让当前线程等待，直到另一个线程调用notify()方法或notifyAll()方法来通知它继续执行。需要注意的是，这两个方法并不是在Thread类中，而是在Object类中。这也意味着任意对象都可以调用这两个方法。

**在执行线程等待和通知时，必须在同步代码块中进行，即都必须先获得目标对象的监视器（共享资源锁）。**

* wait()：永久等待，使调用该方法的线程释放掉持有的共享资源锁，然后从运行状态退出，进入等待队列，直到其它线程唤醒。

* wait(long)：同wait()，但不会无限等待，如果在指定时间内无线程唤醒，则主动唤醒自己，进入可运行状态。

* notify()：随机唤醒等待队列中等待同一共享资源的一个线程，并使该线程退出等待队列，进入可运行状态。

* notifyAll(): 使所有正在等待队列中等待同一共享资源的全部线程退出等待队列，进入可运行状态。

**注意线程变为可运行状态并不一定会马上执行后续操作，因为要执行后续操作，都需要先取得共享资源锁，且一次只能有一个线程取得共享资源锁。如果无法获取到共享资源锁，那么线程将会被阻塞。**

``` java
public class WaitNotifyExample {
    public static void main(String[] args) {
        final Object lock = new Object();
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock) {
                    System.out.println("Thread 1 is waiting...");
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println("Thread 1 is notified.");
                }
            }
        });
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock) {
                    System.out.println("Thread 2 is notifying...");
                    lock.notify();
                    // 此处加入睡眠是为了验证调用wait线程要继续执行，必须要获得共享资源锁
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        thread1.start();
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        thread2.start();
    }
}
```

### 线程挂起suspend和继续执行resume（已废弃，不推荐使用）

Java中的suspend和resume是一种线程挂起和恢复的机制，由于被挂起的线程不会释放其持有的锁资源，容易导致死锁和其它线程安全问题，所以已经被废弃。

当一个线程调用suspend()时，它会被挂起，线程暂停执行，**并且不会释放其持有的对象锁。**

当一个线程调用resume()时，被挂起的线程才能继续执行，其它阻塞在相关锁上的线程也可以继续执行。

``` java
public class BadSuspend {

    public static Object u = new Object();

    public static ChangeObjectThread t1 = new ChangeObjectThread("t1");
    public static ChangeObjectThread t2 = new ChangeObjectThread("t2");

    public static class ChangeObjectThread extends Thread {

        public ChangeObjectThread(String name) {
            super.setName(name);
        }
       @Override
       public void run() {
            synchronized (u) {
                System.out.println("in " + getName());
                // 挂起线程，不释放持有的锁
                Thread.currentThread().suspend();
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        t1.start();
        Thread.sleep(100);
        t2.start();
        t1.resume();
        t2.resume();
        // 等待t1、t2线程执行完
        t1.join();
        t2.join();
    }
}
```

如果执行上述程序，那么就会发现控制台将会先后输出“in 线程信息”，这表明两个线程先后进入了临界区。**但是最终程序并不会停止，而是挂起。如果使用jstack就会发现，线程t2其实是被挂起的，但是t2的线程状态仍然为RUNNABLE。**虽然已经在主函数中调用resume，但是resume并未生效，这就导致了线程t2被永远挂起，并且永远占用对象锁u。

### 等待线程结束join和谦让yield

Java中的join()和yield()方法可以让我们更加灵活的控制线程的执行顺序和时间片分配。

如果一个线程需要等待另一个线程执行完毕后再继续执行，则可以使用join()，**join()的本质实际上是让调用线程wait()在当前线程对象实例上。**

* join()/join(0)：当有新的线程加入时，“主线程”会进入等待状态，直到调用join()方法的线程执行结束为止。
* join(long)：同join()，但是“主线程”不会无限等待，至多等待传入参数的毫秒数。

``` java
public class JoinExample {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Thread 1 is running...");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Thread 1 is finished.");
            }
        });
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("Thread 2 is running...");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("Thread 2 is finished.");
            }
        });
        thread1.start();
        thread2.start();
        System.out.println("Waiting for threads to finish...");
        thread1.join();
        thread2.join();
        System.out.println("All threads have finished.");
    }
}
```

当一个线程所做事的优先级并不高时，可以调用yield()方法时，它会让出CPU时间片，让其他线程（一般是同优先级或更高优先级的线程）有机会执行。**需要注意的是，yield()方法只是让出CPU时间片，是把当前线程的执行权交出去，把当前线程变成RUNNABLE状态，并不会让线程进入等待状态，因此它并不能保证其他线程一定会执行。**

``` java
public class YieldExample {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 5; i++) {
                    System.out.println("Thread 1 is running...");
                    Thread.yield();
                }
                System.out.println("Thread 1 is finished.");
            }
        });
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 5; i++) {
                    System.out.println("Thread 2 is running...");
                    Thread.yield();
                }
                System.out.println("Thread 2 is finished.");
            }
        });
        thread1.start();
        thread2.start();
    }
}
```

### volatile

volatile关键字用于修饰变量，是Java提供的一种轻量级的同步机制（相对于synchronized），**它不会引起线程上下文的切换和调度。** volatile是采用“内存屏障（内存栅栏）”来实现的，volatile修饰的变量，会多出一个lock前缀指令，内存屏障提供三个功能：

* 强制将对缓存的修改操作立即写入主内存。

* 写操作会导致其他CPU中对应的缓存行无效。

* 确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成。

JMM规定所有的变量都存储在主内存中，当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量的值立即刷新回主内存中，**同时导致其他线程中的volatile变量缓存无效；** 当读一个volatile变量时，如果此变量发生过写操作，重新回到主内存中读取最新的共享变量。

volatile可以保证线程可见性且提供了一定的有序性，**volatile无法保证原子性，** volatile主要是保证变量的可见性和禁止指令重排序优化。

``` java
public class VolatileDemo {
    private volatile boolean flag = false;

    public void setFlag(boolean flag) {
        this.flag = flag;
    }

    public void printFlag() {
        System.out.println("flag = " + flag);
    }

    public static void main(String[] args) throws InterruptedException {
        VolatileDemo demo = new VolatileDemo();

        new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            demo.setFlag(true);
            System.out.println("flag has been set to true");
        }).start();

        while (!demo.flag) {
            Thread.sleep(100);
        }

        demo.printFlag();
    }
}
```

### 守护线程

守护线程是一种特殊的线程，它的作用是为其它线程提供服务，当所有的非守护线程都执行完毕后，JVM会自动退出。

``` java
public class DaemonThreadExample {
    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    System.out.println("Daemon thread is running...");
                }
            }
        });
        // 设置守护线程，必须要在start()方法之前设置
        thread.setDaemon(true);
        thread.start();
        System.out.println("Main thread is finished.");
    }
}
```

### 线程优先级

现代操作系统基本采用时分的形式调度运行的线程，操作系统会分出一个个时间片，线程会分配到若干时间片，当线程的时间片用完了就会发生线程调度，并等待着下次分配。线程分配到的时间片多少也就决定了线程使用处理器资源的多少，而线程优先级就是决定线程需要多或者少分配一些处理器资源的线程属性。线程的优先级决定了线程在竞争CPU时间片时的优先级，优先级高的线程会获得更多的CPU时间片，优先级低的线程会获得较少的CPU时间片。

在Java线程中，通过一个整型成员变量priority来控制优先级，优先级的范围从1~10，在线程构建的时候可以通过setPriority(int)方法来修改优先级，默认优先级是5，优先级高的线程分配时间片的数量要多于优先级低的线程。设置线程优先级时，针对频繁阻塞（休眠或者I/O操作）的线程需要设置较高优先级，而偏重计算（需要较多CPU时间或者偏运算）的线程则设置较低的优先级，确保处理器不会被独占。在不同的JVM以及操作系统上，线程规划会存在差异， 有些操作系统甚至会忽略对线程优先级的设定。

**程序正确性不能依赖线程的优先级高低，线程的优先级只是优先级高的线程分配时间片的数量要多于优先级低的线程。**

``` java
public class ThreadPriorityExample {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    System.out.println("Thread 1 is running...");
                }
            }
        });
        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    System.out.println("Thread 2 is running...");
                }
            }
        });
        thread1.setPriority(Thread.MAX_PRIORITY);
        thread2.setPriority(Thread.MIN_PRIORITY);
        thread1.start();
        thread2.start();
    }
}
```

## JDK并发包

为了更好的支持并发程序，JDK内部提供了大量实用的API和框架。

### synchronized

synchronized关键字用于实现线程的同步，它可以保证在同一时刻只有一个线程可以执行某个方法或代码块，从而避免多个线程同时访问共享资源时可能出现的数据不一致问题。

synchronized关键字是一种基于Monitor对象实现的线程同步机制。Monitor对象是Java中的一种同步原语，每个Java对象都可以作为Monitor对象来使用。

当一个线程进入synchronized代码块时，它会尝试获取Monitor对象的锁。如果该锁没有被其他线程占用，那么该线程就会获取到锁，并进入临界区执行代码。如果该锁已经被其他线程占用，那么该线程就会进入阻塞状态，直到该锁被其他线程释放为止。

当一个线程退出synchronized代码块时，它会释放Monitor对象的锁。如果此时有其他线程在等待该锁，那么它们中的一个线程会获取到锁，并进入临界区执行代码。如果没有其他线程在等待该锁，那么该锁就会变为可用状态，其他线程可以继续竞争该锁。

基于进入和退出Monitor对象实现的线程同步机制，可以保证同一时刻只有一个线程进入临界区执行代码，从而避免了线程间的竞争和冲突。同时，该机制还可以保证线程间的可见性和有序性，从而保证了程序的正确性和可靠性。

![synchronized Monitor同步机制](/images/synchronized.jpg)

synchronized关键字可用于修饰方法和代码块，当一个线程访问synchronized修饰的方法和代码块时，需要取得对应的共享资源锁，而其他线程则必须等待该线程释放共享资源锁后才能访问被synchronized关键字修饰的方法或者代码块。**需要注意的是，如果多线程使用的共享资源锁不是同一个对象，那么是无法保证线程安全的。**

当synchronized关键字作用于方法时：

* 实例方法：作用于当前实例加锁，进入同步代码前要先获得当前实例的锁。

* 静态方法：作用于当前类对象（即类名.class对象，并非实例对象，一个类只有一个类对象）加锁，进入同步代码前要先获得当前类对象的锁。使用类对象加锁解决了多个实例对象加锁带来的线程不安全问题。

当synchronized关键字作用于代码块时：

* 锁定指定的对象synchronized(object)：多个线程访问同步代码块时，需要获得该对象的锁才能执行代码块。如果多个线程访问的是同一个对象，那么它们就会互相竞争该对象的锁，从而保证同一时刻只有一个线程可以执行代码块。

* 锁定当前对象synchronized(this)：多个线程访问同步代码块时，需要获得当前对象的锁才能执行代码块。如果多个线程访问的是同一个对象，那么它们就会互相竞争该对象的锁，从而保证同一时刻只有一个线程可以执行代码块。**同synchronized作用于实例方法时。**

* 锁定类对象synchronized(Object.class)：多个线程访问同步代码块时，需要获得该类对象的锁才能执行代码块。由于每个类只有一个类对象，因此使用类对象作为锁可以保证同一时刻只有一个线程可以执行代码块。**同synchronized作用于静态方法时。**

``` java
public class SynchronizedExample {
    private int count = 0;
    private Object lock1 = new Object();

    //锁定指定对象 
    public void increment1() {
        synchronized (lock1) {
            count++;
            System.out.println("Thread 1: " + count);
        }
    }

    // 锁定当前对象
    public synchronized void increment2() {
        count++;
        System.out.println("Thread 2: " + count);
    }

    // 锁定类对象
    public void increment3() {
        synchronized (SynchronizedExample.class) {
            count++;
            System.out.println("Thread 3: " + count);
        }
    }

    public static void main(String[] args) {
        SynchronizedExample example = new SynchronizedExample();

        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    example.increment1();
                }
            }
        });

        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    example.increment2();
                }
            }
        });

        Thread thread3 = new Thread(new Runnable() {
            @Override
            public void run() {
                while (true) {
                    example.increment3();
                }
            }
        });

        thread1.start();
        thread2.start();
        thread3.start();
    }
}
```

使用synchronized关键字时，锁对象的选择非常重要，不同的锁对象会影响到线程同步的效果。如果使用了不恰当的锁对象，可能会导致线程安全问题或者性能问题。同时JVM也对synchronized关键字进行了多种优化，主要包含以下几个方面：

* 偏向锁：在程序启动后，虚拟机会将对象头中的标志位设置为“无锁”状态，表示该对象没有被任何线程锁定。当某个线程第一次访问该对象时，虚拟机会将对象头中的标志位设置为“偏向锁”状态，并将该线程的ID记录在对象头中。之后，该线程再次访问该对象时，无需竞争锁，直接进入临界区执行代码。偏向锁的优化适用于只有一个线程访问对象的情况，可以减少锁竞争的开销。

* 轻量级锁：当多个线程访问同一个对象时，虚拟机会使用轻量级锁来避免重量级锁的开销。轻量级锁的实现方式是，在对象头中记录锁对象的指针和线程ID，当某个线程访问该对象时，虚拟机会将对象头中的锁指针复制到线程栈中，并将对象头中的锁指针指向线程栈中的锁记录。之后，该线程再次访问该对象时，无需竞争锁，直接进入临界区执行代码。如果有多个线程访问同一个对象，那么它们之间会竞争锁，如果竞争成功，就进入临界区执行代码，否则就使用重量级锁。

* 自旋锁：当线程竞争锁失败时，虚拟机会使用自旋锁来避免线程进入阻塞状态。自旋锁的实现方式是，在竞争锁失败后，线程会循环尝试获取锁，而不是进入阻塞状态。如果在一定时间内获取到了锁，就进入临界区执行代码，否则就使用重量级锁。

* 锁消除：在某些情况下，虚拟机会对锁进行消除，从而避免不必要的锁竞争。比如，当虚拟机检测到某个锁对象只被一个线程访问时，就可以将该锁消除掉，从而避免锁竞争的开销。

### 重入锁ReentrantLock

ReentrantLock是一种可重入的互斥锁，与synchronized关键字类似，也可以用来实现线程同步，同时提供了更加灵活和强大的线程同步机制，可以实现更加复杂的线程同步需求。

ReentrantLock主要特点如下：

* 可重入性：ReentrantLock是可重入的互斥锁，同一个线程可以多次获取该锁，而不会导致死锁。

* 锁超时：ReentrantLock支持锁超时机制，即当一个线程在等待锁时，可以设置等待的最长时间，如果超过了该时间仍然没有获取到锁，就会放弃锁的获取。

* 中断响应：ReentrantLock支持中断响应，即当一个线程在等待锁时，可以响应中断信号，从而提高程序的可靠性。

* 条件变量：ReentrantLock提供了条件变量（Condition）的机制，可以让线程在满足某个条件时才能继续执行。条件变量可以用来实现线程间的通信和协作。

ReentrantLock的主要方法如下：

* lock()：获取锁。如果锁已经被其它线程占用，当前线程就会**进入阻塞状态**，直到获取到锁为止。

* tryLock()：尝试获取锁。如果锁没有被其它线程占用，当前线程就会获取到锁，并返回true；否则，当前线程就会立即返回false，**不会进入阻塞状态**。

* tryLock(long timeout, TimeUnit unit)：尝试获取锁，并设置超时时间。如果在指定的时间内获取到了锁，当前线程就会获取到锁，并返回true；否则，当前线程就会立即返回false，**不会进入阻塞状态**。

* lockInterruptibly()：获取锁，但优先响应中断。**在等待锁的过程中没有被中断，那么它会一直阻塞等待获取锁，直到获取到锁为止；如果当前线程在等待锁的过程中被中断，那么它不会一直阻塞等待获取锁，而是会立即响应中断信号，抛出InterruptedException异常**。

* unlock()：释放锁。

``` java
public class ReentrantLockDemo {
    private static ReentrantLock lock1 = new ReentrantLock(false);
    private static ReentrantLock lock2 = new ReentrantLock(false);

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            while (true) {
                if (lock1.tryLock()) {
                    try {
                        System.out.println("Thread 1 got lock1");
                        if (lock2.tryLock()) {
                            try {
                                System.out.println("Thread 1 got lock2");
                                Thread.sleep(2000);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            } finally {
                                lock2.unlock();
                                break;
                            }
                        } else {
                            System.out.println("Thread 1 failed to get lock2");
                        }
                    } finally {
                        lock1.unlock();
                    }
                } else {
                    System.out.println("Thread 1 failed to get lock1");
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });

        Thread t2 = new Thread(() -> {
            while (true) {
                if (lock2.tryLock()) {
                    try {
                        System.out.println("Thread 2 got lock2");
                        if (lock1.tryLock()) {
                            try {
                                System.out.println("Thread 2 got lock1");
                                Thread.sleep(2000);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            } finally {
                                lock1.unlock();
                                break;
                            }
                        } else {
                            System.out.println("Thread 2 failed to get lock1");
                        }
                    } finally {
                        lock2.unlock();
                    }
                } else {
                    System.out.println("Thread 2 failed to get lock2");
                    try {
                        Thread.sleep(500);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

ReentrantLock提供了两种获取锁的方式：**公平锁与非公平锁**。

公平锁是指多个线程按照申请锁的顺序来获取锁，即先到先得的原则。在公平锁的情况下，线程会按照申请锁的顺序排队等待，当锁被释放时，等待时间最长的线程会优先获取锁。公平锁可以避免线程饥饿现象，但是由于需要维护一个有序队列，因此性能相对较低。

非公平锁是指多个线程获取锁的顺序是不确定的，可能会出现后申请的线程比先申请的线程先获取到锁的情况。在非公平锁的情况下，线程会直接尝试获取锁，如果锁没有被占用，就会获取到锁；否则，就会进入阻塞状态等待。非公平锁的性能相对较高，因为不需要维护一个有序队列，但是可能会导致线程饥饿现象。

**ReentrantLock默认是非公平锁**，可以通过构造函数来指定是否使用公平锁。在构造函数中，可以传入一个boolean类型的参数fair，如果fair为true，就表示使用公平锁；否则，就表示使用非公平锁。

``` java
ReentrantLock lock = new ReentrantLock(true); // 使用公平锁
ReentrantLock lock = new ReentrantLock(false); // 使用非公平锁
```

### synchronized与ReentrantLock的区别

ReentrantLock和synchronized都是Java中的线程同步机制，它们的主要区别如下：

* 可重入性：ReentrantLock是可重入的互斥锁，同一个线程可以多次获取该锁，而不会导致死锁；而synchronized也是可重入的互斥锁，同一个线程可以多次获取该锁，而不会导致死锁。

* 锁的获取方式：ReentrantLock提供了公平锁和非公平锁两种获取锁的方式，而synchronized只支持非公平锁。

* 锁的释放方式：ReentrantLock需要手动释放锁，即在finally块中调用unlock方法释放锁；而synchronized会在代码块执行完毕后自动释放锁。

* 性能：在低并发的情况下，synchronized的性能比ReentrantLock要好，因为synchronized是JVM内置的机制，不需要额外的开销；而在高并发的情况下，ReentrantLock的性能比synchronized要好（**synchronized的重量级锁是通过操作系统的互斥量（Mutex）来实现的，操作系统的互斥量是一种特殊的锁，它可以保证同一时刻只有一个线程能够访问共享资源。当一个线程获取到互斥量时，其他线程就无法获取到该互斥量，只能等待当前线程释放互斥量后才能继续执行。因此，使用互斥量可以保证锁的互斥访问，但是会涉及到操作系统的系统调用，开销比较大，因此synchronized的重量级锁性能相对较低**），同时ReentrantLock还提供了锁超时、中断响应等高级特性，能提供更多方式处理锁等待和超时的情况。

### Condition

Condition是一个与锁相关的等待/通知机制，它可以让线程在等待某个条件时进入休眠状态，并在条件满足时被唤醒,**等待线程被唤醒后，依然需要重新获取锁才能继续执行**，与wait()和notify()类似。Condition通常与重入锁一起使用，Condition是由Lock接口提供的，通过newCondition()可以获得一个与当前重入锁绑定的Condition实例，每个Lock对象都可以有多个Condition对象，用于不同的等待/通知场景。

Condition主要方法如下：

* await()：使当前线程进入等待状态，并释放锁，直到被通知或中断。当线程被通知时，它会重新尝试获取锁，并继续执行。

* await(long timeout, TimeUnit unit)：同await()，但不会无限等待，如果在指定时间内无线程唤醒，则主动唤醒自己，进入可运行状态。

* awaitUninterruptibly()：同await()，但是不会响应中断。

* signal()：唤醒一个等待在该Condition上的线程，使其重新尝试获取锁。

* signalAll()：唤醒所有等待在该Condition上的线程，使它们重新尝试获取锁。

``` java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionDemo {
    private static Lock lock = new ReentrantLock();
    private static Condition condition = locl.newCondition();
    private static int count = 0;

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            lock.lock();
            try {
                while (count < 10) {
                    System.out.println("Thread 1: " + count);
                    count++;
                    if (count == 5) {
                        condition.signal();
                        condition.await();
                    }
                }
                condition.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        });

        Thread t2 = new Thread(() -> {
            lock.lock();
            try {
                while (count < 10) {
                    if (count < 5) {
                        condition.await();
                    }
                    System.out.println("Thread 2: " + count);
                    count++;
                }
                condition.signal();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        });

        t1.start();
        t2.start();
    }
}
```

### Semaphore

Semaphore是一种计数信号量，用于控制同时访问某个资源的线程数量。Semaphore内部维护了一个计数器，初始值为指定的数量，每当一个线程访问该资源时，计数器就会减1，当计数器为0时，所有访问该资源的线程都会被阻塞，直到有一个线程释放了该资源，计数器才会加1，唤醒一个或多个等待的线程。

Semaphore主要方法如下：

* acquire()：获取一个许可证，如果没有可用的许可证，则阻塞线程。

* acquireUninterruptibly()：同acquire()，但是不会响应中断。

* tryAcquire()：尝试获取一个许可证，如果没有可用的许可证，则立即返回false，不会阻塞线程。

* tryAcquire(long timeout, TimeUnit unit)：同tryAcquire()，但是可以指定等待时间。

* release()：释放指定数量的许可证，使计数器加上指定的数量。

``` java
public class SemaphoreDemo {
    private static Semaphore semaphore = new Semaphore(2);

    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            try {
                semaphore.acquire();
                System.out.println("Thread 1 acquired a permit");
                Thread.sleep(2000);
                semaphore.release();
                System.out.println("Thread 1 released a permit");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        Thread t2 = new Thread(() -> {
            try {
                semaphore.acquire();
                System.out.println("Thread 2 acquired a permit");
                Thread.sleep(2000);
                semaphore.release();
                System.out.println("Thread 2 released a permit");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        Thread t3 = new Thread(() -> {
            try {
                semaphore.acquire();
                System.out.println("Thread 3 acquired a permit");
                Thread.sleep(2000);
                semaphore.release();
                System.out.println("Thread 3 released a permit");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        t1.start();
        t2.start();
        t3.start();
    }
}
```

### ReadWriteLock

在高并发的的业务场景中，大多都是读数据，少量的写数据。而synchronized和ReentrantLock都是独占锁，即在同一时刻只能有一个线程获取到锁。，如果此时仍然使用独占锁，那么效率将会极其低下。ReadWriteLock就是Java提供的读写锁，它允许多个线程同时读取共享资源，但是只允许一个线程写入共享资源。

ReentrantReadWriteLock读写阻塞如下：

|    |   读   |  写  |
|:--:|:------:|:----:|
| 读 | 非阻塞 | 阻塞 |
| 写 | 阻塞   | 阻塞 |

***

读写锁的特点如下：

* 公平性：支持非公平（默认）和公平的锁获取方式，吞吐量还是非公平优于公平。

* 可重入：读锁和写锁都支持线程重进入。在线程获取读锁之后能够再次获取读锁，但是不能获取写锁，而线程在获取写锁之后能够再次获取写锁，同时也能获取读锁。

* 锁降级：遵循获取写锁、获取读锁再释放写锁的次序，写锁能够降级成为读锁。

``` java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockDemo {

    private static final Lock lock = new ReentrantLock();

    private static final ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private static final Lock readLock = readWriteLock.readLock();

    private static final Lock writeLock = readWriteLock.writeLock();

    private int value;

    public Object handleRead(Lock lock){
        try {
            lock.lock();
            Thread.sleep(1000);
            System.out.println(System.currentTimeMillis() + "read");
            return value;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
        return null;
    }

    public void handleWrite(Lock lock, int index){
        try {
            lock.lock();
            Thread.sleep(1000);
            System.out.println(System.currentTimeMillis() + "write");
            value = index;
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        ReadWriteLockDemo readWriteLockDemo = new ReadWriteLockDemo();
        Runnable readRunnable = () -> {
            // 使用独占锁
            // readWriteLockDemo.handleRead(lock);
            // 使用读锁
            readWriteLockDemo.handleRead(readLock);
        };

        Runnable writeRunnable = () -> {
            // 使用独占锁
            // readWriteLockDemo.handleWrite(lock, 1);
            // 使用写锁
            readWriteLockDemo.handleWrite(writeLock, 1);
        };
        for (int i = 0; i < 18; i++) {
            new Thread(readRunnable).start();
        }
        for (int i = 0; i < 2; i++) {
            new Thread(writeRunnable).start();
        }
    }

}
```

### StampedLock

`StampedLock`是Java8中新增的一种读写锁，用于替代`ReentrantReadWriteLock`，`StampedLock`引入了乐观读锁，实现了在读操作时不会阻塞写操作，解决了读写锁中写线程饥饿的问题。其核心思想是：在读操作的时候如果发生了写操作，应该通过重试的方式来获取新的值，而不应该阻塞写操作。这种模式也就是典型的无锁编程思想。

`StampedLock`有如下三种模式：

* 读模式（Read Lock）：类似于读锁，在读模式下，多个线程可以同时获取读锁，读取共享数据，不会互斥。

* 写模式（Write Lock）：类似于写锁，写模式下，只允许一个线程获取写锁，进行写操作，其他线程无法获取读锁和写锁。

* 乐观读模式（Optimistic Read）：乐观读是一种特殊的读模式，它不会阻塞其他线程的写操作。乐观读尝试获取一个乐观读标记（stamp），并返回该标记。如果在获取标记后没有发生写操作，那么乐观读是有效的，可以进行读操作。否则，乐观读需要回退到悲观读，重新获取读锁。

`StampedLock`的主要方法包括：

* `tryOptimisticRead()`：尝试获取乐观读标记，如果当前没有写操作，返回标记值，否则返回0。

* `validate(long stamp)`：校验乐观读标记，如果在获取标记后没有发生写操作，则返回true，否则返回false。

* `readLock()`：获取读锁，返回一个读锁标记。

* `tryReadLock()`：尝试获取读锁，如果当前没有线程持有写锁，则返回读锁标记，否则返回0。

* `writeLock()`：获取写锁，返回一个写锁标记。

* `tryWriteLock()`：尝试获取写锁，如果当前没有线程持有读锁或写锁，则返回写锁标记，否则返回0。

* `unlockRead(long stamp)`：释放读锁。

* `unlockWrite(long stamp)`：释放写锁。

使用`StampedLock`时需要特别注意的是：

* 乐观读并不是真正意义上的读锁，它不会阻塞其他线程的写操作，因此在使用乐观读时，需要通过`validate()`方法来验证乐观读是否有效。

* 写操作是互斥的，任何一个线程在执行写操作时，其他线程都无法获取读锁和写锁。

* 不支持重入：同一个线程不能多次获取同一个锁标记。`StampedLock`不基于AQS实现，其自身维护了一个状态变量`state`，`state`分为三段：

  * 高24位存储版本号，只有写锁增加其版本号，而读锁不会增加其版本号；

  * 低7位存储读锁被获取的次数；

  * 第8位存储写锁被获取的次数，因为只有一位用于表示写锁，**所以StampedLock不是可重入锁**。

``` java
import java.util.concurrent.locks.StampedLock;

class Point {
    private double x, y;
    private final StampedLock stampedLock = new StampedLock();

    // 使用乐观读模式读取点的坐标
    public double distanceFromOrigin() {
        long stamp = stampedLock.tryOptimisticRead(); // 尝试获取乐观读标记
        double currentX = x;
        double currentY = y;
        if (!stampedLock.validate(stamp)) { // 校验乐观读标记是否有效
            stamp = stampedLock.readLock(); // 获取悲观读锁
            try {
                currentX = x;
                currentY = y;
            } finally {
                stampedLock.unlockRead(stamp); // 释放悲观读锁
            }
        }
        return Math.sqrt(currentX * currentX + currentY * currentY);
    }

    // 使用写模式更新点的坐标
    public void move(double deltaX, double deltaY) {
        long stamp = stampedLock.writeLock(); // 获取写锁
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            stampedLock.unlockWrite(stamp); // 释放写锁
        }
    }
}

public class StampedLockDemo {
    public static void main(String[] args) {
        Point point = new Point();

        // 创建多个线程读取点的坐标
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                double distance = point.distanceFromOrigin();
                System.out.println(Thread.currentThread().getName() + ": Distance from origin: " + distance);
            }).start();
        }

        // 创建一个线程更新点的坐标
        new Thread(() -> {
            point.move(3, 4);
            System.out.println(Thread.currentThread().getName() + ": Point moved to (3, 4)");
        }).start();
    }
}
```

### CountDownLatch

CountDownLatch是一种同步工具，它允许一个或多个线程等待其他线程完成操作后再继续执行。CountDownLatch 定义了一个计数器和一个阻塞队列， 当计数器的值递减为0之前，阻塞队列里面的线程处于挂起状态，当计数器递减到0时会唤醒阻塞队列所有线程。

CountDownLatch主要方法如下：

* countDown()：将计数器的值减1。

* await()：阻塞当前线程，直到计数器的值为0。

* await(long timeout, TimeUnit unit)：同await()，但是增加了超时时间，如果超过了指定的时间，计数器的值仍然不为0，则会唤醒阻塞队列所有线程。

``` java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo {
    private static final int THREAD_COUNT = 5;
    private static CountDownLatch countDownLatch = new CountDownLatch(THREAD_COUNT);

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < THREAD_COUNT; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " started");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " finished");
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
        System.out.println("All threads finished");
    }
}
```

### CyclicBarrier

CyclicBarrier是一种同步工具，它允许多个线程在一个屏障处等待，直到所有线程都到达屏障处后再继续执行。CyclicBarrier内部维护了一个计数器和一个屏障点，当所有线程都到达屏障点后，计数器的值会被重置，并且所有线程都会被释放。

CyclicBarrier与CountDownLatch的区别在于，**CyclicBarrier可以重复使用，而CountDownLatch只能使用一次。另外，CyclicBarrier还可以指定一个Runnable对象，在所有线程都到达屏障点后执行该对象的run()方法**。

CyclicBarrier主要方法如下：

* await()：阻塞当前线程，直到所有线程都到达屏障点。

* await(long timeout, TimeUnit unit)：同await()，但是增加了超时时间，如果超过了指定的时间，计数器的值仍然不为0，则会唤醒阻塞队列所有线程。

* reset()：重置屏障。

``` java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    private static final int THREAD_COUNT = 5;
    private static CyclicBarrier cyclicBarrier = new CyclicBarrier(THREAD_COUNT, () -> {
        System.out.println("All threads have reached the barrier");
    });

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 3; i++) {
            System.out.println("Round " + (i + 1));
            for (int j = 0; j < THREAD_COUNT; j++) {
                new Thread(() -> {
                    System.out.println(Thread.currentThread().getName() + " started");
                    try {
                        Thread.sleep(1000);
                        cyclicBarrier.await();
                    } catch (InterruptedException | BrokenBarrierException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + " finished");
                }).start();
            }
            Thread.sleep(5000);
            // 如果每一轮所有线程都正常执行，不必调用reset()方法，此处只是为了给出reset()使用示例
            cyclicBarrier.reset();
        }
    }
}
```

### LockSupport

LockSupport是一种同步工具，它可以阻塞当前线程，也可以唤醒指定被阻塞的线程。LockSupport与每个使用它的线程都会关联一个许可证，许可证只有0和1两个值，默认是0；当调用park()时，许可证减1，当调用unpark()时，许可证加1，但许可证的最大值为1。在默认情况下调用LockSupport的线程是不持有许可证的，所以调用park()方法会被阻塞，并且线程状态为WAITING（**也就是当许可证的值为0时，调用park会阻塞线程**）。而调用unpark(Thread thread)方法可以使得一个持有许可证的线程被唤醒，**如果调用unpark()方法时，线程并没有持有许可证，则该线程会先持有许可证，然后再释放许可证（也就是不会阻塞），注意许可不可重入，也就是说只能调用一次park()方法，否则会一直阻塞。**

LockSupport主要方法如下：

* park()：阻塞当前线程，如果调用unpark方法或者当前线程被中断，从能从park()方法中返回。

* park(blocker)：功能同方法1，入参增加一个Object对象，用来记录导致线程阻塞的阻塞对象。

* parkNanos(Object blocker, long nanos)：功能同方法2，增加了超时返回，最长不超过nanos纳秒。

* parkUntil(Object blocker, long deadline)：阻塞当前线程，直到deadline时间。

* unpark(Thread thread)：唤醒处于阻塞状态的指定线程。

``` java
public class LockSupportDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println("Thread started");
            LockSupport.park();
            System.out.println("Thread unparked");
        });
        thread.start();
        Thread.sleep(1000);
        LockSupport.unpark(thread);
    }
}
```

### ThreadLocal

除了使用锁控制共享资源的访问外，还可以通过增加资源来保证线程安全。它提供了一种简单的方式来实现线程封闭（Thread confinement）和线程本地变量（Thread-local variables）的概念。它允许每个线程拥有自己独立的变量副本，互不干扰。每个线程可以读写自己的副本，而不会影响其他线程的副本。ThreadLocal通常用于在多线程环境下保存线程私有的数据，从而避免线程间的数据共享和竞争条件。

ThreadLocal的具有如下特点：

* 线程封闭：ThreadLocal可以将某个对象与当前线程绑定，使得每个线程都拥有自己独立的对象副本，从而实现线程封闭，避免数据竞争。

* 线程本地变量：ThreadLocal允许将变量与当前线程关联起来，使得每个线程都可以独立地修改自己的变量副本，而不会影响其他线程的副本。

* 避免线程同步：通过使用ThreadLocal，可以避免使用显式的线程同步手段（如synchronized）来保护共享数据，从而简化了多线程编程。

**需要注意的是，ThreadLocal并不能解决多线程并发访问共享资源的问题，它只是提供了一种线程内部的数据隔离机制。如果多个线程同时访问同一个共享资源，仍然需要使用同步机制来保证线程安全。**

ThreadLocal的实现原理：

* ThreadLocal的线程安全实现是基于每个线程都维护了一个ThreadLocalMap（实际上是一个Entry[]数组）来保存变量的副本。

* 在ThreadLocal的set()方法中，会获取当前线程，并将变量副本存储在该线程的ThreadLocalMap中。

* 每个线程都有自己独立的ThreadLocalMap，因此可以保证线程之间的变量互不干扰。

ThreadLocal的常用方法包括：

* get()：获取当前线程的变量副本。

* set()：设置当前线程的变量副本。

* remove()：移除当前线程的变量副本。在使用ThreadLocal时，要确保在不需要的时候调用remove()方法将ThreadLocal对象从当前线程中删除，以避免内存泄漏。

```java
public class ThreadLocalDemo {
    private static ThreadLocal<Integer> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        for (int i = 0; i < 2; i++) {
            executorService.execute(() -> {
                int num = new Random().nextInt(100);
                threadLocal.set(num);
                System.out.println(Thread.currentThread().getName() + " set num to " + num);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " get num: " + threadLocal.get());
                threadLocal.remove();
            });
        }
        executorService.shutdown();
    }
}
```

## 线程池

池化技术是一种资源管理技术，它可以在应用程序启动时创建一定数量的资源，并将它们保存在一个池中。当应用程序需要使用资源时，可以从池中获取一个空闲的资源来使用，使用完毕后资源会返回到池中，等待下一次使用。池化技术可以提高应用程序的性能和可靠性，避免资源的浪费和性能的下降。

线程池解决的核心问题就是资源管理问题。在并发环境下，系统不能够确定在任意时刻中，有多少任务需要执行，有多少资源需要投入。这种不确定性将带来以下若干问题：

* 频繁申请/销毁资源和调度资源，将带来额外的消耗，可能会非常巨大。

* 对资源无限申请缺少抑制手段，易引发系统资源耗尽的风险。

* 系统无法合理管理内部的资源分布，会降低系统的稳定性。

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

## 无锁

对于并发控制而言，锁是一种悲观的策略。它总是假设每一次的临界区操作会产生冲突，因此当多个线程同时需要访问临界区资源，就宁可牺牲性能让线程等待，因此锁会阻塞线程执行。而无锁则是一种乐观的策略，它总是假设每一次的临界区操作不会产生冲突，因此当多个线程同时需要访问临界区资源时，无锁会让线程不断重试，直到没有冲突为止，因此无锁不会阻塞线程执行。

### 比较交换-CAS(Compare and Swap)

无锁的策略使用一种叫做比较交换的技术（CAS Compare and Swap），CAS是一种无锁算法，它的实现不需要使用传统的锁机制来保证线程安全。CAS是一种原子操作，这意味着这个操作是不可中断的，要么全部执行成功，要么全部执行失败。CAS操作的实现原理是利用了CPU的原子性操作指令，它可以保证在同一时刻只有一个线程能够成功地更新内存位置V的值。如果多个线程同时尝试更新同一个内存位置V的值，只有一个线程能够成功，其他线程会失败并重新尝试，直到成功为止。

相比于传统的锁机制，CAS操作具有以下优点：

* 无需等待锁的释放，因此可以避免锁带来的性能损失。

* 不会引起死锁，因为CAS操作不会阻塞线程。

* 可以支持更高的并发度，因为多个线程可以同时进行CAS操作。

CAS操作包含三个操作数：内存位置V、旧的预期值A和新值B。当且仅当预期值A和内存位置V的值相同时，CAS操作才会将内存位置V的值更新为新值B，否则不会进行任何操作。

**CAS操作不需要使用传统的锁机制，因此它被称为无锁算法。需要注意的是，虽然CAS操作不需要使用传统的锁机制，但它并不是完全没有锁。在CAS操作的实现中，仍然需要使用一些原子性操作指令来保证线程安全。因此，CAS操作被称为无锁算法，是相对于传统的锁机制而言的。**

CAS如何实现线程安全：

* 原子性：CAS操作是原子的，不会被其他线程中断，因此在多线程环境下，CAS操作可以确保共享变量的原子性。如果两个线程同时执行CAS操作，并且内存位置的值与期望值相等，只有其中一个线程能够成功地修改内存位置的值，而另一个线程将会失败，并需要重新尝试。

* 比较与交换：CAS操作涉及两个主要步骤：读取当前内存位置的值和写入新值。在这两个步骤之间，CAS会先检查内存位置的值是否与期望值相等。如果相等，表示没有其他线程在此期间修改过该内存位置，CAS可以继续将新值写入内存位置。如果不相等，表示在此期间有其他线程修改了该内存位置，CAS操作失败，需要重新尝试。

* 循环重试：为了确保CAS操作的成功，CAS一般会使用循环来不断重试。这是因为在多线程环境中，其他线程可能会竞争修改共享变量，导致CAS操作失败。因此，CAS会不断地尝试读取内存位置的当前值并与期望值进行比较，直到CAS操作成功为止。

* 无锁算法：CAS是一种无锁算法，相比于传统的锁机制，它不需要进入或退出临界区，从而减少了线程切换和上下文切换的开销。这使得CAS在一些低竞争情况下具有较好的性能表现。

### JDK CAS初探：AtomicInteger

`AtomicInteger`是`java.util.concurrent.atomic`包下的原子类，用于提供对整型变量的原子操作。它使用CAS（Compare and Swap）操作来实现线程安全，确保对共享整型变量的操作是原子的，即在多线程环境下不会出现竞争条件和数据不一致的问题。其内部使用`value`变量来保存整型变量的值，使用`volatile`关键字修饰的`value`变量来保证对该变量的修改对其他线程是可见的。

``` java
private volatile int value;
```

`AtomicInteger`主要方法如下：

* `compareAndSet(int expect, int update)`：这个方法是CAS操作的核心方法，它用于比较当前`AtomicInteger`对象的值与期望值（`expect`）是否相等。如果相等，就将其值更新为`update`，并返回`true`表示操作成功；如果不相等，则不进行更新，直接返回`false`表示操作失败。CAS操作期间，会保证操作的原子性，即其他线程无法干扰该操作。

* `get()`：这个方法用于获取当前`AtomicInteger`对象的值，类似于普通的读取操作。由于`AtomicInteger`的内部值是使用`volatile`关键字修饰的，所以在调用`get()`方法时，可以保证对其他线程对该值的修改对当前线程是可见的。

* `set(int newValue)`：这个方法用于设置`AtomicInteger`对象的值为指定的`newValue`。与普通的写操作不同的是，`AtomicInteger`的`set()`方法也是原子的，不会被其他线程中断。

* `incrementAndGet()`：这个方法用于对`AtomicInteger`的值进行自增操作并返回自增后的结果。它等价于执行`getAndIncrement()`后再执行`get()`。

* `getAndIncrement()`：这个方法用于获取当前`AtomicInteger`对象的值，并将该值自增1。与`compareAndSet`方法类似，这个方法也是原子的，不会被其他线程中断。

下面就incrementAndGet()方法的具体实现来看看JDK中的CAS是如何实现的：

``` java
public final int incrementAndGet() {
    for (;;) {
        int current = get(); // 获取当前AtomicInteger对象的值
        int next = current + 1; // 计算自增后的值
        if (compareAndSet(current, next)) { // 使用CAS操作尝试更新值
            return next; // 如果CAS操作成功，则返回自增后的值
        }
        // 如果CAS操作失败，则继续循环尝试
    }
}
```

> 基于JDK 1.7源码解析

解析`incrementAndGet()`方法的核心步骤：

* `current = get()`: `incrementAndGet()`方法会调用`get()`方法来获取当前的`AtomicInteger`对象的值。由于`value`内部使用了`volatile`关键字修饰，所以可以保证在多线程环境下对`value`的读取是可见的，即可以读取到其他线程对`value`的修改。

* `next = current + 1`: `incrementAndGet()`方法会计算自增后的值`next`，即将当前值`current`加1。

* `compareAndSet(current, next)`: `incrementAndGet()`方法使用CAS操作的`compareAndSet()`方法尝试将`current`更新为`next`。CAS操作会比较当前的`value`与期望值`current`是否相等，如果相等，则将`value`设置为`next`并返回`true`，表示CAS操作成功；如果不相等，则不进行更新，直接返回`false`，表示CAS操作失败。

* 循环重试：如果CAS操作失败（返回false），`incrementAndGet()`方法会重新进行循环，再次尝试执行CAS操作，直到CAS操作成功为止。这样确保了自增操作的原子性，并处理了多线程并发修改的情况。

综上所述，`AtomicInteger`通过内部使用`volatile`关键字保证对共享变量的可见性，并结合CAS操作来保证对共享变量的原子性操作。在执行CAS操作时，使用硬件提供的原子指令来进行比较与交换，从而避免了传统锁带来的性能开销。由于CAS是一种无锁算法，`AtomicInteger`在多线程环境下具有良好的性能，可以有效地实现线程安全的整型变量操作。

以下是一个AtomicInteger的简单使用示例：

``` java
import java.util.concurrent.atomic.AtomicInteger;

public class CASDemo {
    private static int count1 = 0;
    private static AtomicInteger count2 = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                count1++;
                count2.getAndIncrement();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) {
                count1++;
                count2.getAndIncrement();
            }
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("使用int累加结果：" + count1);
        System.out.println("使用AtomicInteger累加结果：" + count2.get());
    }
}
```

### JDK的CAS原子类

`java.util.concurrent.atomic`包下提供了一系列原子类，用于提供对基本数据类型的原子操作和对引用类型的原子更新操作。这些原子类可以保证在多线程环境下对共享变量的操作是原子的，避免了竞争条件和线程安全问题。

以下是`java.util.concurrent.atomic`包下的主要原子类及其作用：

* `AtomicBoolean`：提供原子操作用于对`boolean`类型的变量进行更新和读取。

* `AtomicInteger`：提供原子操作用于对`int`类型的变量进行更新和读取。

* `AtomicLong`：提供原子操作用于对`long`类型的变量进行更新和读取。

* `AtomicReference<V>`：提供原子操作用于对引用类型（泛型V）的变量进行更新和读取。

* `AtomicIntegerArray`：提供原子操作用于对`int[]`数组中元素进行更新和读取。

* `AtomicLongArray`：提供原子操作用于对`long[]`数组中元素进行更新和读取。

* `AtomicReferenceArray<V>`：提供原子操作用于对引用类型（泛型V）数组中元素进行更新和读取。

* `AtomicIntegerFieldUpdater<T>`：通过反射方式提供原子操作用于更新某类的`int`类型字段。

* `AtomicLongFieldUpdater<T>`：通过反射方式提供原子操作用于更新某类的`long`类型字段。

* `AtomicReferenceFieldUpdater<T, V>`：通过反射方式提供原子操作用于更新某类的引用类型（泛型V）字段。

* `AtomicStampedReference<V>`：提供原子操作用于对带有时间戳的引用类型（泛型V）进行更新和读取，即处理“ABA”问题。

* `LongAdder`：提供原子操作用于对`long`类型的变量进行更新和读取，与`AtomicLong`相比，`LongAdder`在高并发环境下性能更好。

* `LongAccumulator`：提供原子操作用于对`long`类型的变量进行更新和读取，与`AtomicLong`相比，`LongAccumulator`支持自定义的累加操作。

这些原子类可以用于各种多线程环境下的场景，例如计数器、标记位、对象引用的更新等。它们的操作是原子的，避免了使用传统的锁机制带来的性能开销，提供了一种高效的线程安全方式来操作共享变量。

需要注意的是，虽然这些原子类可以确保单个操作的原子性，但多个操作的组合并不一定是原子的。如果需要对多个操作实现原子性的组合操作，仍然需要考虑同步问题。同时，使用这些原子类时，需要确保被操作的对象是线程安全的，以避免其他数据问题。

### CAS带来的ABA问题

CAS（Compare and Swap）操作是一种乐观锁机制，它可以在多线程环境下保证对共享变量的操作是原子性的。但是，CAS操作也会带来一些问题，其中最常见的问题就是“ABA”问题。

"ABA"问题是指一个线程在进行CAS操作时，可能发生以下情况：

* 线程T1读取内存位置A的值为V1。

* 然后线程T2修改内存位置A的值为V2，并执行一系列操作，将内存位置A的值再次修改回V1。

* 最后线程T1执行CAS操作，发现A的值还是V1，认为CAS操作成功，但实际上A的值已经被修改过了。

从线程T1的视角来看，CAS操作期间内存位置A的值并未发生变化，所以它认为CAS操作是成功的。但实际上，线程T2的修改导致了内存位置A的值从V1到V2再到V1的变化，即使在CAS操作的瞬间A的值还是V1，但实际上它经历了一个"ABA"的过程。

"ABA"问题可能会导致一些潜在的错误。例如，在基于CAS的无锁算法中，当执行CAS操作时，线程可能无法感知到共享变量在操作过程中发生了"ABA"的变化，从而导致操作错误。

"ABA"示例代码如下：

``` java
import java.util.concurrent.atomic.AtomicInteger;

public class ABADemo {
    private static AtomicInteger count = new AtomicInteger(100);

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            count.compareAndSet(100, 101);
            count.compareAndSet(101, 100);
        });

        Thread t2 = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            boolean result = count.compareAndSet(100, 102);
            System.out.println("线程2执行CAS操作的结果：" + result);
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("最终的count值：" + count.get());
    }
}
```

解决“ABA”问题的常见方法是引入版本号或时间戳的概念。这样，在CAS操作时，不仅会比较共享变量的值，还会比较版本号或时间戳，以确保操作的正确性。Java提供了`AtomicStampedReference`来解决ABA问题，其中的`stamp`（戳记）表示版本号或时间戳，与引用值一起作为CAS操作的一部分，确保CAS操作的正确性。

使用`AtomicStampedReference`解决"ABA"问题的示例代码如下：

``` java
import java.util.concurrent.atomic.AtomicStampedReference;

public class ABADemo {
    private static AtomicStampedReference<Integer> count = new AtomicStampedReference<>(100, 0);

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            int stamp = count.getStamp();
            count.compareAndSet(100, 101, stamp, stamp + 1);
            stamp = count.getStamp();
            count.compareAndSet(101, 100, stamp, stamp + 1);
        });

        Thread t2 = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            int stamp = count.getStamp();
            boolean result = count.compareAndSet(100, 102, stamp, stamp + 1);
            System.out.println("线程2执行CAS操作的结果：" + result);
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("最终的count值：" + count.getReference());
    }
}
```

需要注意的是，并非所有情况下都需要解决“ABA”问题。在某些场景下，“ABA”问题可能不会产生负面影响，因此需要根据具体情况来决定是否采取相应的解决方案。但在某些关键场景下，特别是对于有状态的数据结构，解决“ABA”问题是非常重要的。

### JDK CAS的底层实现-Unsafe

`sun.misc.Unsafe`是Java的一个不稳定、不推荐使用的内部类，提供了直接操作内存和执行CAS（Compare and Swap）等底层操作的功能。由于它是`sun.misc`包下的非公开 API，并且直接操作内存和硬件级别的原子操作，使用`Unsafe`需要谨慎，并不推荐在生产环境中使用（**自身应用程序也无法使用Unsafe类，非Bootstrap类加载器加载的类会抛出异常，而应用程序的类由App Loader加载**）。`Unsafe`类中的所有方法都是native方法，也就是说它们都是使用C/C++实现的，因此可以直接操作系统底层资源，而不受Java内存管理机制的限制。

`Unsafe`类的部分方法列表包括：

* `compareAndSwapInt(Object obj, long offset, int expected, int update)`：执行CAS操作，将`obj`对象在内存偏移量`offset`处的`int`类型值与`expected`比较，如果相等，则将其更新为`update`。该方法是实现CAS操作的关键方法。

* `getInt(Object obj, long offset)`：获取`obj`对象在内存偏移量`offset`处的`int`类型值。

* `putInt(Object obj, long offset, int value)`：将`int`类型值`value`设置到`obj`对象在内存偏移量`offset`处。

* `getObject(Object obj, long offset)`：获取`obj`对象在内存偏移量`offset`处的引用类型值。

* `putObject(Object obj, long offset, Object value)`：将引用类型值`value`设置到`obj`对象在内存偏移量`offset`处。

Unsafe类通过直接操作对象的内存偏移量，实现了无锁的原子操作，包括CAS操作。在实现CAS时，它通过硬件指令来实现原子性操作。这些原子指令在底层是由处理器提供的，能够确保在多线程环境下进行原子操作，避免了传统锁带来的性能开销。

在Java的并发包中，`AtomicInteger`、`AtomicLong`等原子类的实现就依赖于`Unsafe`类的CAS操作。然而，由于`Unsafe`类是非常底层的，并且直接操作内存，使用不当可能导致严重的内存错误。因此，一般情况下，开发者应该优先考虑使用`java.util.concurrent.atomic`包下的原子类，而避免直接使用`Unsafe`类。

## 设计模式-Future

在设计模式中，"Future" 是一种并发设计模式，用于处理异步计算和并发编程。它的核心思想是异步调用。当我们调用一个方法时，如果这个方法需要很长的时间才能返回，那么就会导致线程阻塞，从而影响程序的执行效率。为了避免这种情况，我们可以通过Future模式来实现异步调用，即在调用一个方法时，不需要立即等待它的执行结果，而是在另一个线程中执行这个方法，并在需要的时候获取执行结果。

### Future模式的主要角色

Future模式主要由如下角色构成：

* Main：客户端主线程，用于提交任务和获取任务执行结果，即调用Client发出请求。

* Client：客户端，用于提交任务，返回Data对象，立即返回FutureData，并开启ClientThread线程装配RealData。

* Data：返回数据的接口，FutureData和RealData是其具体实现类。

* FutureData：Future模式的核心，用于获取任务执行结果，Future数据，构造很快，但是是一个虚拟的数据，需要装配RealData，好比一个订单。

* RealData：真实的任务执行结果，其构造是比较慢的。

Future模式的主要角色关系如下：

![Future模式的角色关系图](/images/Future.jpg)

### Future模式的简单实现

为了更好的理解Future模式的主要角色是如何互相配合工作的，下面通过一个简单的示例来说明Future模式。

Data:

``` java
public interface Data {
    String getResult();
}
```

FutureData:

``` java
public class FutureData implements Data {
    // 内部需要维护RealData
    private RealData realData;
    private boolean isReady = false;

    public synchronized void setRealData(RealData realData) {
        if (isReady) {
            return;
        }
        this.realData = realData;
        isReady = true;
        // 内部需要维护RealData
        notifyAll();
    }

    @Override
    public synchronized String getResult() {
        while (!isReady) {
            try {
                // 内部需要维护RealData
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //真正需要的数据从RealData获取
        return realData.getResult();
    }
}
```

RealData:

``` java
public class RealData implements Data {
    private final String result;

    public RealData(String para) {
        //RealData的构造可能很慢，需要用户等待很久，这里使用sleep模拟
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 10; i++) {
            sb.append(para);
            try {
                //使用sleep模拟RealData的构造，可能很慢，需要用户等待很久
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        result = sb.toString();
    }

    @Override
    public String getResult() {
        return result;
    }
}
```

Client:

``` java
public class Client {
    public Data request(final String queryStr) {
        final FutureData future = new FutureData();
        new Thread(() -> {
            // RealData的构建很慢，所以在单独的线程中进行
            RealData realData = new RealData(queryStr);
            //setRealData()的时候会notify()等待在这个future上的对象
            future.setRealData(realData);
        }).start();
        // FutureData会被立即返回，不会等待RealData被构造完
        return future;
    }
}
```

Main:

``` java
public class Main {
    public static void main(String[] args) {
        Client client = new Client();
        // 这里会立即返回，因为得到的是FutureData而不是RealData
        Data data = client.request("name");
        System.out.println("请求完毕");
        try {
            // 这里可以用一个sleep代替了对其他业务逻辑的处理
            // 在处理这些业务逻辑的过程中，RealData被创建，从而充分利用了等待时间
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 使用真实的数据
        // 如果到这里数据还没有准备好，getResult()会等待数据准备完毕才返回
        System.out.println("数据 = " + data.getResult());
    }
}
```

以上便是一个Future模式的简单实现。从上面的示例中我们可以看到，当我们调用Client的request方法时，会立即返回FutureData，而不是RealData，这样就可以充分利用等待时间，提高程序的执行效率。当我们需要使用数据时，可以通过FutureData的getResult方法来获取数据，如果数据没有准备好，那么程序就会阻塞，直到RealData准备好数据并注入到FutureData中才返回。

### JDK中Future的Client-Callable

`Callable` 是 Java 中用于支持返回结果和抛出异常的多线程任务的接口。它与 `Runnable` 接口类似，但提供了一个 `call()` 方法来执行任务，允许任务执行的结果有返回值和异常处理。因此，它更适合于执行需要返回结果或抛出异常的复杂计算任务。`call` 方法定义如下：

* `V call() throws Exception`: 该方法在实现类中实现具体的任务逻辑。它可以返回一个结果类型为 `V` 的值，也可以抛出 `Exception` 或其子类的异常。如果任务执行过程中抛出异常，调用 `Future.get()` 方法获取结果时，会抛出 `ExecutionException`，并将原始异常包装在其中。

```java
import java.util.concurrent.*;

public class CallableDemo {

    public static void main(String[] args) {
        // 创建一个线程池
        ExecutorService executor = Executors.newFixedThreadPool(1);

        // 创建一个Callable任务
        Callable<Long> task = () -> {
            int number = 5;
            long factorial = 1;
            for (int i = 1; i <= number; i++) {
                factorial *= i;
            }
            return factorial;
        };

        try {
            // 提交Future任务
            Future<Long> future = executor.submit(task);

            // 从 Future 获取结果（这将阻塞直到结果可用）
            long result = future.get();
            System.out.println("Factorial of 5 is: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        // 关闭线程池
        executor.shutdown();
    }
}
```

### JDK中的Future模式

为了处理异步计算和更好的支持并发编程，JDK也实现了一整套Future模式。在JDK中，Future模式主要由如下角色构成：

* Future：Future模式的核心接口，用于获取异步计算的结果，即Future模式中的Data角色。

* FutureTask：Future的实现类，用于包装Callable或Runnable任务，即Future模式中的FutureData角色。

* Callable/Runnable：用于包装需要异步执行的任务，即Future模式中的RealData角色。

* Executor：线程池，用于执行异步任务，即Future模式中的Client角色。

#### 异步计算结果-Future

JDK中的Future接口是实现Future模式的核心接口之一，它表示一个异步计算的结果，提供了一系列方法来检查计算是否完成、等待计算完成以及获取计算结果。

`Future` 接口定义了以下方法：

* `boolean cancel(boolean mayInterruptIfRunning)`: 尝试取消正在执行的任务。如果任务已经完成或已经被取消，则此方法返回 false。如果任务正在运行，并且 `mayInterruptIfRunning` 为 true，则任务将被中断。
  * 如果任务尚未开始执行，或者已经执行完成，那么 `cancel()` 方法会尝试取消任务，并返回 true。
  * 如果任务正在运行，并且 `mayInterruptIfRunning` 为 true，那么会向任务发送中断信号，并尝试取消任务，然后返回 true。
  * 如果任务正在运行，并且 `mayInterruptIfRunning` 为 false，那么不会中断任务，仅尝试取消任务，并返回 false。
  * 如果任务已经被取消，那么 `cancel()` 方法不会做任何操作，返回 false。

* `boolean isCancelled()`: 返回任务是否已被取消。如果任务在运行过程中被取消，或者在任务完成前调用了 `cancel()` 方法将其取消，那么 `isCancelled()` 方法会返回 true。

* `boolean isDone()`: 返回任务是否已完成。如果任务已经执行完成、或者被取消、或者因为某些原因导致了异常而结束，那么 `isDone()` 方法会返回 true。

* `V get() throws InterruptedException, ExecutionException`: 获取计算结果，如果任务尚未完成，则阻塞直到任务完成。
  * 如果任务正常完成，`get()` 方法会返回任务的计算结果（类型为 `V`）。
  * 如果任务在执行过程中被中断，`get()` 方法会抛出 `InterruptedException` 异常。
  * 如果任务由于某些原因导致异常而结束，`get()` 方法会抛出 `ExecutionException` 异常，并将原始异常封装在其中。

* `V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException`: 获取计算结果，如果任务尚未完成，则最多阻塞指定的时间。
  * 如果任务在指定的时间内正常完成，`get()` 方法会返回任务的计算结果（类型为 `V`）。
  * 如果任务在执行过程中被中断，`get()` 方法会抛出 `InterruptedException` 异常。
  * 如果任务由于某些原因导致异常而结束，`get()` 方法会抛出 `ExecutionException` 异常，并将原始异常封装在其中。
  * 如果任务在指定的时间内未完成，`get()` 方法会抛出 `TimeoutException` 异常。

`FutureTask` 类实现了 `Future` 接口，它是 `Future` 模式的核心实现类之一，用于将 Callable 或 Runnable 任务包装成异步计算任务。

#### JDK的Future模式实现

``` java
import java.util.concurrent.*;

public class FutureDemo {

    public static void main(String[] args) {
        // 创建线程池
        ExecutorService executor = Executors.newFixedThreadPool(1);

        // 创建一个Callable任务
        Callable<Integer> task = () -> {
            int sum = 0;
            for (int i = 1; i <= 1000; i++) {
                sum += i;
                Thread.sleep(1); // Simulate a time-consuming task
            }
            return sum;
        };

        // 包装Callable任务为FutureTask
        FutureTask<Integer> futureTask = new FutureTask<>(task);

        // 提交任务
        executor.submit(futureTask);

        // 模拟做其它任务
        System.out.println("Doing something else...");

        try {
            // 从 Future 获取结果（这将阻塞直到结果可用）
            int result = futureTask.get();
            System.out.println("The result is: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        // 关闭线程池
        executor.shutdown();
    }
}

```

### JDK中增强的Future-ComplatedFuture

传统给的Future+线程池异步配合，提高了程序的执行效率。但是Future对于结果的获取，不是很友好，只能通过阻塞(`get()`)或者轮询的方式(`isDone()`)得到任务的结果。**阻塞的方式和异步编程的设计理念相违背，而轮询的方式会耗费无谓的CPU资源。**因此，JDK8设计出CompletableFuture。CompletableFuture提供了一种观察者模式类似的机制，可以让任务执行完成后通知监听的一方。

CompletableFuture是Java 8中新增的增强型 Future，它实现了Future和CompletionStage接口，提供了更强大和灵活的异步编程功能。它是 `java.util.concurrent.Future` 接口的一个实现，同时也实现了 `CompletionStage` 接口，支持更多的操作和组合，使异步编程更加简洁和高效。

使用`CompletableFuture`替代`Future`改进上述demo：

``` java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class CompletableFutureDemo {

    public static void main(String[] args) {
        // 创建一个CompletableFuture对象
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
            int sum = 0;
            for (int i = 1; i <= 1000; i++) {
                sum += i;
                try {
                    Thread.sleep(1); // 模拟一个耗时的任务
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            return sum;
        });

        // 模拟做其他任务
        System.out.println("Doing something else...");

        // 获取计算结果
        try {
            // 依然会阻塞主线程，后面介绍其他非阻塞方式
            int result = future.get();
            System.out.println("The result is: " + result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

可以从示例代码中看到，使用`CompletableFuture`之后，不再需要创建线程池了，因为实际上，使用的是默认线程池ForkJoinPool.commonPool。

#### CompletableFuture方法总结

`CompletableFuture` 类提供了大量的方法来支持异步编程，这些方法可以分为以下几类：

* 创建异步计算任务

  * `runAsync(Runnable runnable)`: 创建一个异步任务，不返回线程执行结果，返回 `CompletableFuture<Void>` 对象。

  * `supplyAsync(Supplier<U> supplier)`: 创建一个异步任务，并返回线程执行结果，返回 `CompletableFuture<U>` 对象。

* 获取任务结果

  * `get()`: 获取计算结果，如果任务尚未完成，则阻塞直到任务完成。

  * `get(long timeout, TimeUnit unit)`: 获取计算结果，如果任务尚未完成，则最多阻塞指定的时间。

  * `join()`: 获取计算结果，如果任务尚未完成，会等待任务完成。 完成时返回结果值，否则抛出unchecked异常。

  * `getNow(U valueIfAbsent)`: 获取计算结果，如果任务已经完成，返回计算结果，否则返回指定的值。

  * `complete(T value)`: 如果任务还未完成，则将任务的计算结果设置为指定的值。

  * `completeExceptionally(Throwable ex)`: 如果任务还未完成，则将任务的计算结果设置为指定的异常并抛出。

* 对异步计算任务的结果进行处理

  * `thenApply(Function<? super T,? extends U> fn)`: 当异步计算完成时，对异步计算的结果进行处理，并返回处理后的结果。**同步方法，在当前线程中执行转换操作，会阻塞当前线程直到转换完成。下文不再介绍，同理。**

  * `thenApplyAsync(Function<? super T,? extends U> fn)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，并返回 `CompletableFuture<U>` 对象。**异步方法，不会阻塞当前线程，可以并发执行转换任务。下文不再介绍，同理。**

  * `thenAccept(Consumer<? super T> action)`: 当异步计算完成时，对异步计算的结果进行处理，无返回值。

  * `thenAcceptAsync(Consumer<? super T> action)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，无返回值。

  * `thenRun(Runnable action)`: 当异步计算完成时，对异步计算的结果进行处理，无返回值。

  * `thenRunAsync(Runnable action)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，无返回值。

  * `whenComplete(BiConsumer<? super T,? super Throwable> action)`: 当异步计算完成时，对异步计算的结果进行处理，无返回值。

  * `whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，无返回值。

* 任务组合处理

  * `thenCompose(Function<? super T,? extends CompletionStage<U>> fn)`: 当异步计算完成时，对异步计算的结果进行处理，返回 `CompletionStage<U>` 对象。

  * `thenComposeAsync(Function<? super T,? extends CompletionStage<U>> fn)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，并返回 `CompletionStage<U>` 对象。

  * `thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)`: 当异步计算完成时，对异步计算的结果进行处理，返回 `CompletionStage<V>` 对象。

  * `thenCombineAsync(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，并返回 `CompletionStage<V>` 对象。

  * `thenAcceptBoth(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)`: 当异步计算完成时，对异步计算的结果进行处理，无返回值。

  * `thenAcceptBothAsync(CompletionStage<? extends U> other, BiConsumer<? super T,? super U> action)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，无返回值。

  * `runAfterBoth(CompletionStage<?> other, Runnable action)`: 当异步计算完成时，对异步计算的结果进行处理，无返回值。

  * `runAfterBothAsync(CompletionStage<?> other, Runnable action)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，无返回值。

  * `applyToEither(CompletionStage<? extends T> other, Function<? super T,U> fn)`: 当异步计算完成时，对异步计算的结果进行处理，返回 `CompletionStage<U>` 对象。

  * `applyToEitherAsync(CompletionStage<? extends T> other, Function<? super T,U> fn)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，并返回 `CompletionStage<U>` 对象。

  * `acceptEither(CompletionStage<? extends T> other, Consumer<? super T> action)`: 当异步计算完成时，对异步计算的结果进行处理，无返回值。

  * `acceptEitherAsync(CompletionStage<? extends T> other, Consumer<? super T> action)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，无返回值。

  * `runAfterEither(CompletionStage<?> other, Runnable action)`: 当异步计算完成时，对异步计算的结果进行处理，无返回值。

  * `runAfterEitherAsync(CompletionStage<?> other, Runnable action)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，无返回值。

* 异常处理

  * `handle(BiFunction<? super T,Throwable,? extends U> fn)`: 当异步计算完成时，对异步计算的结果进行处理，可以根据计算的结果和异常进行不同的处理，并返回处理后的结果。

  * `handleAsync(BiFunction<? super T,Throwable,? extends U> fn)`: 当异步计算完成时，对异步计算的结果进行处理，使用 `ForkJoinPool.commonPool()` 中的线程执行处理，并返回 `CompletableFuture<U>` 对象。

  * `CompletableFuture<T> exceptionally(Function<Throwable,? extends T> fn)`: 当异步计算出现异常时，对异常进行处理并返回处理后的结果。

以上是`CompletableFuture`类中一些常用的方法。`CompletableFuture`通过提供链式操作和丰富的异步编程功能，使得并发编程更加简洁、高效和灵活。使用`CompletableFuture` 可以方便地处理异步任务的结果、异常和组合，充分发挥多核处理器的性能，提高程序的并发性能。
