---
title: JDK并发包
date: 2023-07-08 10:34:21
tags: [并发编程, Java并发包, JDK并发包, 多线程]
---

## JDK并发包

为了更好的支持并发程序，JDK内部提供了大量实用的API和框架。

### synchronized

synchronized关键字用于实现线程的同步，它可以保证在同一时刻只有一个线程可以执行某个方法或代码块，从而避免多个线程同时访问共享资源时可能出现的数据不一致问题。

synchronized关键字是一种基于Monitor对象实现的线程同步机制。Monitor对象是Java中的一种同步原语，每个Java对象都可以作为Monitor对象来使用。

当一个线程进入synchronized代码块时，它会尝试获取Monitor对象的锁。如果该锁没有被其他线程占用，那么该线程就会获取到锁，并进入临界区执行代码。如果该锁已经被其他线程占用，那么该线程就会进入阻塞状态，直到该锁被其他线程释放为止。

当一个线程退出synchronized代码块时，它会释放Monitor对象的锁。如果此时有其他线程在等待该锁，那么它们中的一个线程会获取到锁，并进入临界区执行代码。如果没有其他线程在等待该锁，那么该锁就会变为可用状态，其他线程可以继续竞争该锁。

基于进入和退出Monitor对象实现的线程同步机制，可以保证同一时刻只有一个线程进入临界区执行代码，从而避免了线程间的竞争和冲突。同时，该机制还可以保证线程间的可见性和有序性，从而保证了程序的正确性和可靠性。

![synchronized Monitor同步机制](/images/synchronized.jpg)

<!-- more -->

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
