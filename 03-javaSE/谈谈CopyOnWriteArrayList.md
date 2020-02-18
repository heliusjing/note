# 谈谈CopyOnWriteArrayList

## 前言

如果面试官问：

`ArrayList`是线程安全的么？如果ArrayList线程不安全的话，那你知道有哪些线程安全的集合么？

如果你答可以使用`Vector`、`Collections`下的方法对集合进行一层包装。

仅仅答到这的话，那太遗憾了，你要回去等通知了。

---

## :sob: ArrayList线程不安全

我们知道`ArrayList`是线程不安全的，那我们首先来看一个`ArrayList` 线程不安全的示例

```java
     public static void main(String[] args) {
         List<String> list = new CopyOnWriteArrayList<>();
         for (int i = 1; i <= 30; i++) {
             new Thread(() -> {
                 list.add(UUID.randomUUID().toString().substring(1, 8));
                 System.out.println(list);
             }, String.valueOf(i)).start();
         }
         /**
          * 1.故障现象
          *  java.util.ConcurrentModificationException
          * 2.导致原因
          *    并发争抢修改导致
          * 3.解决方案
          *  3.1 new Vector<>()
          *  3.2 Collections.synchronizedList(new ArrayList<>());
          *  3.3 new CopyOnWriteArrayList<>();
          *
          *
          * 4.优化建议
          */
     }
```

这段代码是在多线程环境下执行，多执行几遍你就会发现问题，这是由于**并发修改**所致。

## CopyOnWriteArrayList

底层实现采用了  **写入时拷贝** 的思想，增删改操作会将底层数组拷贝一份，更
改操作在新数组上执行，这时不影响其它线程的**并发读，读写分离**，适合读多写少的场景，当然你也找不到更好的了。

我们直接看其添加方法add()

```java
  public boolean add(E e) {
		
		// 加锁
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
			
			// 得到原数组的长度和元素
            Object[] elements = getArray();
            int len = elements.length;
			
			// 复制出一个新数组
            Object[] newElements = Arrays.copyOf(elements, len + 1);
			
			// 添加时，将新元素添加到新数组中
            newElements[len] = e;
			
			// 将volatile Object[] array 的指向替换成新数组
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
```

哈哈 ，是不是挺简单的

通过代码我们可以知道：在添加的时候就上锁，并**复制一个新数组，增加操作在新数组上完成，将array指向到新数组中**，最后解锁。

:arrow_heading_down:

而对于读操作，是未加锁的

```java
public void forEach(Consumer<? super E> action) {
	Objects.requireNonNull(action);
	for (Object x : getArray()) {
	@SuppressWarnings("unchecked") E e = (E) x;
		action.accept(e);
	}
}
```

即只有并发写的时候，通过互斥保证线程安全。

## 弱一致性

思考：如果两个现程，一个在修改，一个在遍历， 修改是上锁的，且都是在复制一份在新数组上完成，那此时，遍历拿到的又是旧数组，这就造成了遍历的是旧数据。

举个例子：

```java
CopyOnWriteArrayListpublic class CopyOnWriteArrayListTest {
    public static void main(String[] args) throws InterruptedException {

        CopyOnWriteArrayList<Integer> list = new CopyOnWriteArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        Iterator<Integer> iter = list.iterator();
        new Thread(() -> {
            list.remove(0);
            System.out.println(list);
        }).start();
        TimeUnit.SECONDS.sleep(1);
        while (iter.hasNext()) {
            System.out.println(iter.next());
        }
    }
}
```

这不算是什么缺点，一致性和并发一这是冲突的，比如数据库。