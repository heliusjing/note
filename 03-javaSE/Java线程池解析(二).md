# Java线程池解析(二)



> :notebook_with_decorative_cover:上一篇文章介绍了线程池的基础知识，这节将更加深入;对于上一篇重复过的知识，这里不再赘述！

## :fire: 线程池的执行

我们在调用线程池执行任务的时候，一般直接调用

> **ThreadPoolExecutor类的execute方法**

```java
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();

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
```

具体的执行流程文字描述版，我们已经在上一篇文章中详细解释了。

