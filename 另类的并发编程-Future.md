---
title: 另类的并发编程-Future
date: 2023-07-30 17:12:43
tags: [并发编程, Future]
---

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

<!-- more -->

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
