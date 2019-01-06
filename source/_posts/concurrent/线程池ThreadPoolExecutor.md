---
title: 线程池ThreadPoolExecutor
categories: 
- concurrent
tags:
---


# 1、线程池的好处
线程是稀缺资源，如果被无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，
合理的使用线程池对线程进行统一分配、调优和监控，有以下好处：
1、降低资源消耗；
2、提高响应速度；
3、提高线程的可管理性。


# 2、线程池常见的创建方式
Java1.5中引入的Executor框架把任务的提交和执行进行解耦，只需要定义好任务，
然后提交给线程池，而不用关心该任务是如何执行、被哪个线程执行，以及什么时候执行。

常见的创建线程池方式有以下几种：
Executors.newCachedThreadPool()：无限线程池。
Executors.newFixedThreadPool(nThreads)：创建固定大小的线程池。
Executors.newSingleThreadExecutor()：创建单个线程的线程池。

其实看这三种方式创建的源码就会发现，实际上还是利用 ThreadPoolExecutor 类实现的。

ThreadPoolExecutor(int corePoolSize, 
                    int maximumPoolSize, 
                    long keepAliveTime,
                     TimeUnit unit, 
                     BlockingQueue<Runnable> workQueue,
                      RejectedExecutionHandler handler)
这几个核心参数的作用：
corePoolSize 为线程池的基本大小。
maximumPoolSize 为线程池最大线程大小。
keepAliveTime 和 unit 则是线程空闲后的存活时间。
workQueue 用于存放任务的阻塞队列。
handler 当队列和最大线程池都满了之后的饱和策略。

# 3、线程池的执行过程
通常我们都是使用:
threadPool.execute(new Job());
这样的方式来提交一个任务到线程池中，所以核心的逻辑就是 execute() 函数了。
在具体分析之前先了解下线程池中所定义的状态，这些状态都和线程的执行密切相关：

 // runState is stored in the high-order bits
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;
    
- RUNNING 自然是运行状态，指可以接受任务执行队列里的任务
- SHUTDOWN 指调用了 shutdown() 方法，不再接受新任务了，但是队列里的任务得执行完毕。
- STOP 指调用了 shutdownNow() 方法，不再接受新任务，同时抛弃阻塞队列里的所有任务并中断所有正在执行任务。
- TIDYING 所有任务都执行完毕，在调用 shutdown()/shutdownNow() 中都会尝试更新为这个状态。
- TERMINATED 终止状态，当执行 terminated() 后会更新为这个状态。

各个状态之间的转化关系：
![线程池状态转化](/images/线程池状态转化.png)

然后看看 execute() 方法是如何处理的：

    public void execute(Runnable command) {
            if (command == null)
                throw new NullPointerException();
            /*
             * Proceed in 3 steps:
             *
             * 1. If fewer than corePoolSize threads are running, try to
             * start a new thread with the given command as its first
             * task.  The call to addWorker atomically checks runState and
             * workerCount, and so prevents false alarms that would add
             * threads when it shouldn't, by returning false.
             *
             * 2. If a task can be successfully queued, then we still need
             * to double-check whether we should have added a thread
             * (because existing ones died since last checking) or that
             * the pool shut down since entry into this method. So we
             * recheck state and if necessary roll back the enqueuing if
             * stopped, or start a new thread if there are none.
             *
             * 3. If we cannot queue task, then we try to add a new
             * thread.  If it fails, we know we are shut down or saturated
             * and so reject the task.
             */
            int c = ctl.get();
            if (workerCountOf(c) < corePoolSize) {
                if (addWorker(command, true))
                    return;
                c = ctl.get();
            }
            if (isRunning(c) && workQueue.offer(command)) {
                int recheck = ctl.get();
                if (! isRunning(recheck) && remove(command))
                    reject(command);
                else if (workerCountOf(recheck) == 0)
                    addWorker(null, false);
            }
            else if (!addWorker(command, false))
                reject(command);
        }

![线程池execute方法执行过程](/images/线程池execute方法执行过程.png)

1、获取当前线程池的状态。
2、当前线程数量小于 coreSize 时创建一个新的线程运行。
3、如果当前线程处于运行状态，并且写入阻塞队列成功。
4、双重检查，再次获取线程状态；如果线程状态变了（非运行状态）就需要从阻塞队列移除任务，并尝试判断线程是否全部执行完毕。同时执行拒绝策略。
5、如果当前线程池为空就新创建一个线程并执行。
6、如果在第三步的判断为非运行状态，尝试新建线程，如果失败则执行拒绝策略。

![线程池execute方法执行过程1](/images/线程池execute方法执行过程1.png)
![线程池execute方法执行过程2](/images/线程池execute方法执行过程2.png)


# 4、如何配置线程池的大小
流程聊完了再来看看上文提到了几个核心参数应该如何配置呢？
有一点是肯定的，线程池肯定是不是越大越好。
通常我们是需要根据这批任务执行的性质来确定的。
IO 密集型任务：由于线程并不是一直在运行，所以可以尽可能的多配置线程，比如 CPU 个数 * 2
CPU 密集型任务（大量复杂的运算）应当分配较少的线程，比如 CPU 个数相当的大小。
当然这些都是经验值，最好的方式还是根据实际情况测试得出最佳配置。

# 5、优雅的关闭线程池
有运行任务自然也有关闭任务，从上文提到的 5 个状态就能看出如何来关闭线程池。
其实无非就是两个方法 shutdown()/shutdownNow()。
但他们有着重要的区别：
shutdown() 执行后停止接受新任务，会把队列的任务执行完毕。
shutdownNow() 也是停止接受新任务，但会中断所有的任务，将线程池状态变为 stop。
两个方法都会中断线程，用户可自行判断是否需要响应中断。
shutdownNow() 要更简单粗暴，可以根据实际场景选择不同的方法。

    long start = System.currentTimeMillis();
    for (int i = 0; i <= 5; i++) {
       pool.execute(new Job());
    }
    
    pool.shutdown();
    
    while (!pool.awaitTermination(1, TimeUnit.SECONDS)) {
       LOGGER.info("线程还在执行。。。");
    }
    long end = System.currentTimeMillis();
    LOGGER.info("一共处理了【{}】", (end - start));
    
pool.awaitTermination(1, TimeUnit.SECONDS) 会每隔一秒钟检查一次是否执行完毕（状态为 TERMINATED），
当从 while 循环退出时就表明线程池已经完全终止了。


