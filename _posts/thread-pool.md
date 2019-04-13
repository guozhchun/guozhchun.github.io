---
title: 谈谈线程池
date: 2019-03-24 18:26:41
tags: java
categories: java
---

# 前言

多线程是 java 并发编程中的一个重要特性，最简单的用法是在需要使用多线程的地方 new 一个新的线程让其异步运行。但是，线程的创建是一个比较大的开销，如果有多个任务需要频繁间歇地异步执行，为每个任务单独创建一个线程就显得有点浪费。因此，可以考虑使用线程池的做法，在线程池中保持一定数量的线程，当有任务需要异步执行时，直接让线程池中的空闲线程获取该任务进行执行。这样就能避免每次创建线程的开销。

<!-- more -->

# 线程池的基本概念

当创建一个线程池的时候，需要传入以下参数

```java
int corePoolSize                  // 核心线程数量，即线程池中所有线程都空闲时，允许存活的线程数量（allowCoreThreadTimeOut 为 true 时，存活线程数量允许为 0）
int maximumPoolSize               // 线程池中线程的最大数量，超过这个数量时，线程池不会再接受新的任务
long keepAliveTime                // 线程空闲时，允许存活的时间
TimeUnit unit                     // 线程空闲时，允许存活的时间的时间单位
BlockingQueue<Runnable> workQueue // 工作队列（也称阻塞队列），当提交的任务数量多于核心线程数量时，会将新提交的任务放入此队列中
ThreadFactory threadFactory       // 创建线程的工厂
RejectedExecutionHandler handler  // 线程池拒绝接受新任务的处理器
```

# 线程池的创建

一般情况下，我们使用 `Executors` 类中的工厂类创建所需要的线程池，常用的两种线程池是 `Executors.newCachedThreadPool()` 、`Executors.newFixedThreadPool()`，其构造函数如下

```java
// newCachedThreadPool 使用的是 SynchronousQueue 工作队列
// 当提交一个任务时，如果线程池中刚好有一个线程处理完其他的任务空闲下来，就将任务交给这个线程进行处理
// 否则，创建一个新的线程处理该任务
// 由于 corePoolSize 为 0，所以当线程空闲一段时间后，会被销毁
// 由于 maximumPoolSize 为 Integer.MAX_VALUE，因此当任务很多时，也能创建相应的线程及时地处理任务
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

// newFixedThreadPool 会设置 corePoolSize 和 maximumPoolSize 相等，从而保证固定的线程数
// 同时传入的是一个无边界的阻塞队列，最大限度下保证所有任务都能被处理（任务过多时，内存耗尽溢出）
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

# 线程池的状态

线程池中定义了 5 个状态：`RUNNING`、`SHUTDOWN`、`STOP`、`TIDYING`、`TERMINATED`。

`RUNNING`：表示线程池处于运行状态

`SHUTDOWN`：表示线程池处于关闭状态，可调用 `shutdown` 函数将线程池的状态由 `RUNNING` 变成此状态。当线程池处于此状态下时，线程池不再接受新的任务，但是对于已经提交的任务和正在运行的任务，会继续执行。

`STOP`：表示线程池处于停止状态，可调用 `shutdownNow` 函数将线程池的状态由 `RUNNING` 或 `SHUTDOWN` 变成此状态。当线程处于此状态时，线程池不在接受新的任务，同时会中断正在执行的任务，而对于工作队列中等待执行的任务，将不再调用执行。

`TIDYING`：表示线程池处于「纯净待结束」状态。此状态可由 `SHUTDOWN` 状态或 `STOP` 状态转换而来。对于状态 `SHUTDOWN`，如果其工作队列中没有等待的线程，同时没有线程在运行时，将其状态变为 `TIDYING`。对于状态 `STOP`，当线程池中没有线程在运行时，将其状态转变为 `TIDYING`。当线程池转换此状态时，会调用可扩展的`terminated` 函数。

`TERMINATED`：表示线程池处于结束状态。此状态由 `TIDYING` 转换而来。当执行完 `terminated` 后线程池变为此状态。

其各个状态之间的转换关系大致如下图所示

![线程池状态转换图](/images/thread-pool-state.png)

# 线程池的运行过程

当我们执行 `execute.submit(task)`向线程池中提交任务时，其会先判断当前运行线程是否小于核心线程数量，如果是，则新建一个线程运行，否则判断任务是否能放入工作队列（阻塞队列）中，如果可以，则放入阻塞队列中等待调度执行，否则继续判断当前运行线程数量是否大于最大线程数量，如果小于则新建线程运行任务，否则执行拒绝策略。

## 流程图

提交任务后线程池的处理过程，大致流程图如下所示

```flow
start=>start: start
submit=>operation: 提交一个任务
lessThanCoreSize=>condition: 运行的线程数量
							 小于 corePoolSize
newThread=>operation: 新建一个线程运行任务
isQueueFull=>condition: 阻塞队列是否已满
waitRun=>operation: 等待调度执行
lessThanMaxSize=>condition: 运行的线程数量小于 
							maximumPoolSize
executeRejection=>operation: 执行拒绝策略
end=>end: end
start->submit->lessThanCoreSize(no)->isQueueFull(yes)->lessThanMaxSize(no)->executeRejection->end
lessThanCoreSize(yes, right)->newThread->end
isQueueFull(no, right)->waitRun->end
lessThanMaxSize(yes)->newThread->end

```

## 部分源码解析

当执行 `submit` 函数时，会调用 `execute` 函数

```java
public Future<?> submit(Runnable task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<Void> ftask = newTaskFor(task, null);
    execute(ftask);
    return ftask;
}

public <T> Future<T> submit(Runnable task, T result) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task, result);
    execute(ftask);
    return ftask;
}

public <T> Future<T> submit(Callable<T> task) {
    if (task == null) throw new NullPointerException();
    RunnableFuture<T> ftask = newTaskFor(task);
    execute(ftask);
    return ftask;
}
```

在 `execute` 函数中，会依次对线程数量是否小于核心线程数量、是否能加入阻塞队列、线程数量是否小于最大线程数量进行判断。当线程数量小于核心线程数量时，会新建一个线程运行当前任务，否则加入阻塞队列，当加入阻塞队列失败时，会判断线程数量是否小于最大线程数量，如果小于则新建线程运行当前任务，否则执行拒绝策略。

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    
    // ctl 变量是一个 AtomicInteger 类型的整数，存储着当着线程池的状态，以及线程池中线程的数量
    // 线程池的状态用整数的最高 3 位表示
    // 整数的低 29 位表示线程数量，即线程池中线程数量最多只能是 2^29 - 1
    int c = ctl.get();
    // 如果线程池中的数量小于核心线程数量，则新建一个线程执行该任务
    // 线程池内部维护一个 worker 变量，表示正在运行的线程
    // addWorker 函数内部会对线程池的状态及线程数量进行判断，满足条件后会创建一个新的线程运行任务
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    // 程序运行到此处，意味这线程池中线程数量大于核心线程数量
    // 此处判断线程池状态是否是运行状态，如果是运行状态，则尝试将任务放入工作队列（阻塞队列）中
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 此处再次对线程池的状态进行判断，主要是考虑并发的情况
        // 可能存在一个线程提交任务放入阻塞队列成功后，另一个线程提交了 shutdonw 请求关闭线程池
        // 因此此处再次判断状态，如果不是运行状态，则将任务从阻塞队列中移除
        // 因为执行了 shutdonw 或shutdownNow 后不允许提交新的任务
        if (! isRunning(recheck) && remove(command))
            reject(command);
        // 此处判断线程数量是否为 0，如果为 0，则新建一个线程，并且让这个线程从阻塞队列中拉取任务运行
        // 这种考虑是因为线程池中 allowCoreThreadTimeOut 可能被设置为 true
        // 那么，可能存在一种情况：在将任务放入阻塞队列后，核心线程因为空闲的时间达到阈值而被回收
        // 此时，如果不新建一个线程，那么阻塞队列中的任务将不能及时得到处理
        // 由于已经将任务放入阻塞队列中，因此此处不能直接处理当前提交的任务，而应该按照规则从队列中拉取
        // 因此，此处 addWorker 函数传入的是 null 而不是当前的任务
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 程序运行到此处，说明任务放入阻塞队列失败（阻塞队列满了）
    // 直接执行 addWorker 函数，在函数中判断线程数量是否达到 maximumPoolSize
    // 如果小于，则新建一个线程，运行当前的任务
    else if (!addWorker(command, false))
        // 程序运行到这里，说明线程池中线程数量已经等于 maximumPoolSize，执行拒绝策略
        reject(command);
}
```

接下来看看 `addWorker` 函数

```java
// 第一个参数 firstTask：表示新建线程后要运行的任务，如果为空，则从阻塞队列中拉取一个任务运行
// 第二个参数 core：表示是否用核心线程数与线程池中的线程数量进行比较
//                true：表示用 corePoolSize (核心线程数) 与线程池中线程数进行比较
//                false：表示用 maximumPoolSize （最大线程数）与线程池中线程数进行比较
private boolean addWorker(Runnable firstTask, boolean core) {
    retry:
    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 判断线程池的状态
        // 如果线程池状态不是运行状态，即 SHUTDOWN、STOP、TIDYING、TERMINATED，则不再新建线程
        // 只有一种例外，即在SHUTDOWN状态并且工作队列（阻塞队列）中有未处理的任务，并且firstTask为空
        // 因为在 SHUTDOWN 状态，不允许提交新的任务，当时对于已经存在工作队列中的任务，必须处理完毕
        // 从 worker 类中的 run 函数中可知，当 firstTask 为空时，getTask 函数会从工作队列中获取任务
        if (rs >= SHUTDOWN &&
            ! (rs == SHUTDOWN &&
               firstTask == null &&
               ! workQueue.isEmpty()))
            return false;

        for (;;) {
            int wc = workerCountOf(c);
            // 此处对线程池中的线程数量进行判断
            if (wc >= CAPACITY ||
                wc >= (core ? corePoolSize : maximumPoolSize))
                return false;
            // 此处用 CAS 判断是否增加数量成功
            // 增加成功，则跳出循环，进行后续的新建线程操作
            if (compareAndIncrementWorkerCount(c))
                break retry;
            // 一般情况下，上述 CAS 增加不成功时，只需要在内部继续循环即可
            // 当是考虑到可能线程池状态发现改变，因此此处对线程池状态进行判断
            // 如果线程池的状态发生了改变，则跳出内部循环，开始下一轮的外层循环
            c = ctl.get();  // Re-read ctl
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }

    // 以上是对线程池状态以及线程数量的判断
    // 以下是新建线程并开始执行任务
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        // 用 worker 封装任务，worker 中的 thread 就是传入的 firstTask 参数
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                int rs = runStateOf(ctl.get());

                if (rs < SHUTDOWN ||
                    (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable
                        throw new IllegalThreadStateException();
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            if (workerAdded) {
                // 新建的线程开始执行，传入的任务也在此处开始执行
                // 此方法会调用 worker 中的 run 方法
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        // 如果线程没有启动成功，需要进行一些清理操作
        // 例如将前面放入 workers 的元素移除
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}
```

线程池中新建线程会执行 `worker` 的 `run` 方法，`run` 方法中只是简单的调用 `runWorker` 方法，因此，接下来看下 `runWorker` 的实现。

```java
public void run() {
    runWorker(this);
}

final void runWorker(Worker w) {
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;
    w.unlock(); // allow interrupts
    boolean completedAbruptly = true;
    try {
        // 这里是循环获取任务并执行任务
        // task 可以是 worker 中的 firstTask，即新建线程时传入的要执行的任务
        // 如果 task 为空，则执行 getTask 函数从工作队列中获取
        while (task != null || (task = getTask()) != null) {
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            if ((runStateAtLeast(ctl.get(), STOP) ||
                 (Thread.interrupted() &&
                  runStateAtLeast(ctl.get(), STOP))) &&
                !wt.isInterrupted())
                wt.interrupt();
            try {
                // 可扩展的函数，在执行任务前运行
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    // 此处是任务真正执行的地方
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    // 可扩展的函数，在执行任务后运行
                    afterExecute(task, thrown);
                }
            } finally {
                // 任务执行完毕后，重置 task 为空，使其可以继续从工作队列中获取新的任务
                task = null;
                w.completedTasks++;
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        processWorkerExit(w, completedAbruptly);
    }
}
```

接下来看下 `getTask` 的实现

```java
// 此函数从工作队列中获取任务，首先会执行状态检查
// 如果是 STOP、TIDYING、TERMINATED 状态，则返回 null。对于 TIDYING、TERMINATED 状态，工作队列肯定是空的，对于 STOP 状态，其会放弃工作队列中的任务，因此也不用从工作队列中获取任务
// 如果是 SHUTDOWN 状态，则判断工作队列是否为空，如果为空，则返回 null
// 在通过状态检查后，会进行线程数量判断，如果线程数量大于 maximumPoolSize（可以通过 setMaximumPoolSize 函数动态设置），则返回 null
// 在通过线程数量检查后，会根据是否要超时等待从工作队列中获取任务，如果设置了超时，则超时后返回 null，否则从工作队列中获取任务返回
private Runnable getTask() {
    boolean timedOut = false; // Did the last poll() time out?

    for (;;) {
        int c = ctl.get();
        int rs = runStateOf(c);

        // 线程池状态检查
        // 如果是 SHUTDOWN 状态，则判断工作队列是否为空，如果为空，则返回 null
        // 如果是 STOP、TIDYING、TERMINATED 状态，则返回 null
        // 对于 TIDYING、TERMINATED 状态，工作队列肯定是空的，所以直接返回 null
        // 对于 STOP 状态，其会放弃工作队列中的任务，因此也不用从工作队列中获取任务
        if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
            decrementWorkerCount();
            return null;
        }

        int wc = workerCountOf(c);

        // Are workers subject to culling?
        boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

        if ((wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())) {
            if (compareAndDecrementWorkerCount(c))
                return null;
            continue;
        }

        try {
            // 到工作队列中获取任务
            // 如果设置了超时机制，则等待一定时间后返回，否则阻塞等待直到取到任务为止
            Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
            workQueue.take();
            if (r != null)
                return r;
            timedOut = true;
        } catch (InterruptedException retry) {
            timedOut = false;
        }
    }
}
```

至此，`execute` 的函数大致解析完毕。

# 线程池的拒绝策略

当线程池中的线程数量大于 `maximumPoolSize` 时，再次提交任务，会执行拒绝策略。线程池中提供了四种拒绝策略可以供我们直接使用。当然，也可以自定义拒绝策略，在创建线程池中传入即可。线程池中定义的四种拒绝策略如下

| 拒绝策略            | 处理方式                                                     |
| ------------------- | ------------------------------------------------------------ |
| CallerRunsPolicy    | 如果线程池没有关闭，由提交任务的线程自己执行这个任务         |
| AbortPolicy         | 抛出 RejectedExecutionException 异常，这是默认的拒绝策略，如果在创建线程池的时候不传入参数指定拒绝策略，会默认使用这个拒绝策略 |
| DiscardPolicy       | 不做任何处理，直接丢弃这个任务                               |
| DiscardOldestPolicy | 如果线程池没有关闭，从工作队列（阻塞队列）中取出队头的任务丢弃，然后重新尝试提交任务。此策略不建议与使用优先级队列（PriorityQueue）的工作队列结合使用 |

# 自定义线程池示例

本例自定义一个线程池，其中 corePoolSize 为 3，maximumPoolSize 为 5，工作队列（阻塞队列）是长度为 2 的有界阻塞队列，拒绝策略采用默认的抛 RejectedExecutionException 异常的方式 。在创建线程池后，向线程提交 10 个任务进行运行。刚开始，由于线程池中线程数量小于 3，所以会创建 3 个线程用于运行提交的任务，当任务继续提交时，会将任务放入阻塞队列中等待运行，当提交任务达到 5 个时，阻塞队列已满，此时再提交任务，会继续创建新的线程运行刚刚提交的任务。当提交任务达到 7 个时（3 个在核心线程中运行，2 个在阻塞队列中，另外 2 个是由于线程数小于 maximumPoolSize 而新创建的线程执行的），再提交任务，会执行拒绝策略，抛出 RejectedExecutionException 异常。

```java
public class TestThreadPool
{
    public static void main(String[] args)
    {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(3, 5, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<>(2));
        for (int i = 0; i < 10; i++)
        {
            System.out.println("Push task count: " + (i + 1));
            final String threadName = "TTT" + (i + 1);
            try
            {
                threadPoolExecutor.execute(new Runnable() {
                    @Override
                    public void run()
                    {
                        System.out.println("Run thread " + threadName);
                        try
                        {
                            Thread.sleep(500);
                        }
                        catch (InterruptedException e)
                        {
                            e.printStackTrace();
                        }
                    }
                });
            }
            catch (Exception e)
            {
                System.out.println("Submit thread " + threadName + " error");
                e.printStackTrace();
            }
            
            System.out.println("Pool size: " + threadPoolExecutor.getPoolSize());
            System.out.println("Queue count: " + threadPoolExecutor.getQueue().size());
            System.out.println("-----------------");
        }
        threadPoolExecutor.shutdown();
    }
}
```

输出结果如下

```
Push task count: 1
Pool size: 1
Queue count: 0
-----------------
Push task count: 2
Run thread TTT1
Pool size: 2
Queue count: 0
Run thread TTT2
-----------------
Push task count: 3
Pool size: 3
Queue count: 0
-----------------
Push task count: 4
Pool size: 3
Run thread TTT3
Queue count: 1
-----------------
Push task count: 5
Pool size: 3
Queue count: 2
-----------------
Push task count: 6
Pool size: 4
Queue count: 2
-----------------
Push task count: 7
Pool size: 5
Run thread TTT6
Queue count: 2
-----------------
Push task count: 8
Run thread TTT7
Submit thread TTT8 error
java.util.concurrent.RejectedExecutionException: Task thread.TestThreadPool$1@55f96302 rejected from java.util.concurrent.ThreadPoolExecutor@3d4eac69[Running, pool size = 5, active threads = 5, queued tasks = 2, completed tasks = 0]
Pool size: 5
Queue count: 2
	at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.reject(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.execute(Unknown Source)
	at thread.TestThreadPool.main(TestThreadPool.java:18)
-----------------
Push task count: 9
Submit thread TTT9 error
java.util.concurrent.RejectedExecutionException: Task thread.TestThreadPool$1@75b84c92 rejected from java.util.concurrent.ThreadPoolExecutor@3d4eac69[Running, pool size = 5, active threads = 5, queued tasks = 2, completed tasks = 0]
	at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.reject(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.execute(Unknown Source)
	at thread.TestThreadPool.main(TestThreadPool.java:18)
Pool size: 5
Queue count: 2
-----------------
Push task count: 10
Submit thread TTT10 error
java.util.concurrent.RejectedExecutionException: Task thread.TestThreadPool$1@232204a1 rejected from java.util.concurrent.ThreadPoolExecutor@3d4eac69[Running, pool size = 5, active threads = 5, queued tasks = 2, completed tasks = 0]
	at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.reject(Unknown Source)
	at java.util.concurrent.ThreadPoolExecutor.execute(Unknown Source)
	at thread.TestThreadPool.main(TestThreadPool.java:18)
Pool size: 5
Queue count: 2
-----------------
Run thread TTT4
Run thread TTT5

```

从输出结果中可以看出

* 当提交任务数不超过 3 时，线程池立刻创建新线程执行任务。这从`Run Thread TTT1` 到 `Run Thread TTT3` 这一段的输出结果可以看出
* 当提交任务数是 `(3, 5]` 个时，提交的任务放如阻塞队列中等待执行。这从最后最后的两行输出 `Run thread TTT4`、`Run thread TTT5` 以及这两行在 `Run thread TTT7` 后面可以看出。
* 当提交任务数是`(5, 7]`个时，线程池立刻创建新的线程执行任务。这从输出 ` Run Thread TTT3 ` 后直接输出  `Run Thread TTT6`  和  `Run Thread TTT7` 而后才输出  `Run thread TTT4`、`Run thread TTT5`  可以看出
* 当提交任务数是 `(7, 10]` 个时，线程池执行拒绝策略抛出异常。这从 `Submit thread TTT8 error`、`Submit thread TTT9 error`、`Submit thread TTT10 error` 可以看出

# 扩展阅读

[<https://javadoop.com/post/java-thread-pool>](<https://javadoop.com/post/java-thread-pool>)