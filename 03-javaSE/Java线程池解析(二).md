# Java线程池解析(二)



> :notebook_with_decorative_cover:上一篇文章介绍了线程池的基础知识，这节将更加深入;对于上一篇重复过的知识，这里不再赘述！

## 

## :athletic_shoe: ThreadPoolExecutor

再看继承结构：

<img src="Java线程池解析(二).assets/image-20200209131328498.png" alt="image-20200209131328498" style="zoom:67%;" />

---

## 线程池状态

上一节中已经阐述了线程池的**五种状态**

这对应于源码中的：

```java
    
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3;      // 29
private static final int CAPACITY   = (1 << COUNT_BITS) - 1; // 00011111 ... ... 11111111

// 状态在高位存储：RUNNING算起来较复杂，注意负数的补码=反码+1
private static final int RUNNING    = -1 << COUNT_BITS;      // 11100000 ... ... 00000000
private static final int SHUTDOWN   =  0 << COUNT_BITS;      // 00000000 ... ... 00000000
private static final int STOP       =  1 << COUNT_BITS;      // 00100000 ... ... 00000000
private static final int TIDYING    =  2 << COUNT_BITS;      // 01000000 ... ... 00000000
private static final int TERMINATED =  3 << COUNT_BITS;      // 01100000 ... ... 00000000
    // Packing and unpacking ctl
    private static int runStateOf(int c)     { return c & ~CAPACITY; }
    private static int workerCountOf(int c)  { return c & CAPACITY; }
	// rs 为高 3 位代表线程池状态， wc 为低 29 位代表线程个数，ctl 是合并它们
    private static int ctlOf(int rs, int wc) { return rs | wc; }

    /*
     * Bit field accessors that don't require unpacking ctl.
     * These depend on the bit layout and on workerCount being never negative.
     */

    private static boolean runStateLessThan(int c, int s) {
        return c < s;
    }

    private static boolean runStateAtLeast(int c, int s) {
        return c >= s;
    }

    private static boolean isRunning(int c) {
        return c < SHUTDOWN;
    }
```

这是`ThreadPoolExecutor`类开头的一段代码，看起来比较吃力。

其中的位运算，这里就不做具体验证了。



|   状态名   | 高 3位 | 接收新任务 | 处理阻塞队列任务 |                   说明                    |
| :--------: | :----: | :--------: | :--------------: | :---------------------------------------: |
|  RUNNING   |  111   |     Y      |        Y         |                                           |
|  SHUTDOWN  |  000   |     N      |        Y         | 不会接收新任务，但会处理阻塞队列剩余任务  |
|    STOP    |  001   |     N      |        N         | 会中断正在执行的任务，并抛弃阻塞队列任务  |
|  TIDYING   |  010   |     -      |        -         | 任务全执行完毕，活动线程为 0 即将进入终结 |
| TERMINATED |  011   |     -      |        -         |                 终结状态                  |

从数字上比较**，TERMINATED > TIDYING > STOP > SHUTDOWN > RUNNING**

这些信息存储在一个原子变量 `ctl` 中，目的是将**线程池状态与线程个数**合二为一，这样就可以用一次 cas 原子操作进行赋值，更容易保证**在多线程环境下保证运行状态和线程数量的统一**。

都是大师的智慧啊

---

## :fire_engine:拒绝策略

`JDK`提供了4种拒绝策略的实现，其他框架也提供了实现。

- **AbortPolicy(抛出一个异常，默认的)**
- **DiscardPolicy(直接丢弃任务)**
- **DiscardOldestPolicy（丢弃队列里最老的任务，将当前这个任务继续提交给线程池）**
- **CallerRunsPolicy（交给线程池调用所在的线程进行处理)**
- Dubbo 的实现，在抛出 RejectedExecutionException 异常之前会记录日志，并 dump 线程栈信息，方
  便定位问题
- Netty 的实现，是创建一个新线程来执行任务
- ActiveMQ 的实现，带超时等待（60s）尝试放入队列，类似我们之前自定义的拒绝策略
- PinPoint 的实现，它使用了一个拒绝策略链，会逐一尝试策略链中每种拒绝策略

**例如：**`AbortPolicy`

```java
    public static class AbortPolicy implements RejectedExecutionHandler {
        /**
         * Creates an {@code AbortPolicy}.
         */
        public AbortPolicy() { }

        /**
         * Always throws RejectedExecutionException.
         *
         * @param r the runnable task requested to be executed
         * @param e the executor attempting to execute this task
         * @throws RejectedExecutionException always
         */
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("Task " + r.toString() +
                                                 " rejected from " +
                                                 e.toString());
        }
    }
```

其他可自行参阅`ThreadPoolExecutor`类

---

