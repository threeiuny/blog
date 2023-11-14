---
title: Java线程的基本操作
date: 2023-06-24 15:33:57
tags: [Java, 线程]
---

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

<!-- more -->

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
