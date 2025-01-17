### 前言

  本文的主要详细分析ArrayBlockingQueue的实现原理，由于该并发集合其底层是使用了java.util.ReentrantLock和java.util.Condition来完成并发控制的，我们可以通过JDK的源代码更好的学习这些并发控制类的使用，同时该类也是所有并发集合中最简单的一个，分析该类的源码也是为之后分析其他并发集合做好基础。



## 一、Queue接口和BlockingQueue接口回顾

### 1.1 Queue接口回顾

> 在Queue接口中，除了继承Collection接口中定义的方法外，它还分别额外地定义**插入、删除、查询**这3个操作，其中每一个操作都以两种不同的形式存在，每一种形式都对应着一个方法。

方法说明：

|             | Throws exception | Returns special value |
| ----------- | ---------------- | --------------------- |
| **Insert**  | add(e)           | offer(e)              |
| **Remove**  | remove()         | poll()                |
| **Examine** | element()        | peek()                |

1. add方法在将一个元素插入到队列的尾部时，如果出现队列已经满了，那么就会抛出IllegalStateException,而使用offer方法时，如果队列满了，则添加失败,返回false,但并不会引发异常。
2. remove方法是获取队列的头部元素并且删除，如果当队列为空时，那么就会抛出NoSuchElementException。而poll在队列为空时，则返回一个null。
3. element方法是从队列中获取到队列的第一个元素，但不会删除，但是如果队列为空时，那么它就会抛出NoSuchElementException。peek方法与之类似，只是不会抛出异常，而是返回false。

后面我们在分析ArrayBlockingQueue的方法时，主要也是围绕着这几个方法来进行分析。

### 1.2 BlockingQueue接口回顾

> BlockingQueue是JDK1.5出现的接口，它在原来的Queue接口基础上提供了更多的额外功能：当获取队列中的头部元素时，如果队列为空，那么它将会使执行线程处于等待状态；当添加一个元素到队列的尾部时，如果队列已经满了，那么它同样会使执行的线程处于等待状态。

前面我们在说Queue接口时提到过，它针对于相同的操作提供了2种不同的形式，而BlockingQueue更夸张，针对于相同的操作提供了4种不同的形式。

> 该四种形式分别为：
>
> 1. 抛出异常  
> 2. 返回一个特殊值(可能是null或者是false，取决于具体的操作)
> 3. 阻塞当前执行直到其可以继续
> 4. 当线程被挂起后，等待最大的时间，如果一旦超时，即使该操作依旧无法继续执行，线程也不会再继续等待下去。

对应的方法说明:

|             | Throws exception | Returns special value | Blocks | Times out            |
| ----------- | ---------------- | --------------------- | ------ | -------------------- |
| **Insert**  | add(e)           | offer(e)              | put(e) | offer(e, time, unit) |
| **Remove**  | remove()         | poll()                | take() | poll(time, unit)     |
| **Examine** | element()        | peek()                | 无     | 无                   |

BlockingQueue虽然比起Queue在操作上提供了更多的支持，但是它在使用有如下的几点:

> 1. BlockingQueue中是不允许添加null的，该接受在声明的时候就要求所有的实现类在接收到一个null的时候，都应该抛出NullPointerException。
> 2. BlockingQueue是线程安全的，因此它的所有和队列相关的方法都具有原子性。但是对于那么从Collection接口中继承而来的批量操作方法，比如addAll(Collection e)等方法，BlockingQueue的实现通常没有保证其具有原子性，因此我们在使用的BlockingQueue，应该尽可能地不去使用这些方法。
> 3. BlockingQueue主要应用于生产者与消费者的模型中，其元素的添加和获取都是极具规律性的。但是对于remove(Object o)这样的方法，虽然BlockingQueue可以保证元素正确的删除，但是这样的操作会非常响应性能，因此我们在没有特殊的情况下，也应该避免使用这类方法。

## 二、ArrayBlockingQueue深入分析

有了上面的铺垫，下面我们就可以真正开始分析ArrayBlockingQueue了。在分析之前，首先让我们看看API对其的描述。
**注意:这里使用的JDK版本为1.7，不同的JDK版本在实现上存在不同**

![](https://upload-images.jianshu.io/upload_images/1684370-ca508b8735c40257.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先让我们看下ArrayBlockingQueue的核心组成：

```java
 /** 底层维护队列元素的数组 */
    final Object[] items;

    /**  当读取元素时数组的下标(这里称为读下标) */
    int takeIndex;

    /** 添加元素时数组的下标 (这里称为写小标)*/
    int putIndex;

    /** 队列中的元素个数 */
    int count;

    /**用于并发控制的工具类**/
    final ReentrantLock lock;

    /** 控制take操作时是否让线程等待 */
    private final Condition notEmpty;

    /** 控制put操作时是否让线程等待 */
    private final Condition notFull;
```

> take方法分析(369-379行):

```java
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        /*
            尝试获取锁，如果此时锁被其他线程锁占用，那么当前线程就处于Waiting的状态。           
            注意:当方法是支持线程中断响应的如果其他线程此时中断当前线程，
            那么当前线程就会抛出InterruptedException 
         */
        lock.lockInterruptibly();
        try {
            /*
          如果此时队列中的元素个数为0,那么就让当前线程wait,并且释放锁。
          注意:这里使用了while进行重复检查，是为了防止当前线程可能由于其他未知的原因被唤醒。
          (通常这种情况被称为"spurious wakeup")
            */    
        while (count == 0)
                notEmpty.await();
            //如果队列不为空，则从队列的头部取元素
            return extract();
        } finally {
             //完成锁的释放
            lock.unlock();
        }
    }
```

> extract方法分析(163-171):

```java
    /*
      根据takeIndex来获取当前的元素,然后通知其他等待的线程。
      Call only when holding lock.(只有当前线程已经持有了锁之后，它才能调用该方法)
     */
    private E extract() {
        final Object[] items = this.items;

        //根据takeIndex获取元素,因为元素是一个Object类型的数组,因此它通过cast方法将其转换成泛型。
        E x = this.<E>cast(items[takeIndex]);

        //将当前位置的元素设置为null
        items[takeIndex] = null;

        //并且将takeIndex++,注意：这里因为已经使用了锁，因此inc方法中没有使用到原子操作
        takeIndex = inc(takeIndex);

        //将队列中的总的元素减1
        --count;
        //唤醒其他等待的线程
        notFull.signal();
        return x;
    }
```

> put方法分析(318-239)

```java
    public void put(E e) throws InterruptedException {
        //首先检查元素是否为空，否则抛出NullPointerException
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        //进行锁的抢占
        lock.lockInterruptibly();
        try {
            /*当队列的长度等于数组的长度,此时说明队列已经满了,这里同样
              使用了while来方式当前线程被"伪唤醒"。*/
            while (count == items.length)
                //则让当前线程处于等待状态
                notFull.await();
            //一旦获取到锁并且队列还未满时，则执行insert操作。
            insert(e);
        } finally {
            //完成锁的释放
            lock.unlock();
        }
    }

      //检查元素是否为空
     private static void checkNotNull(Object v) {
        if (v == null)
            throw new NullPointerException();
      }

     //该方法的逻辑非常简单
    private void insert(E x) {
        //将当前元素设置到putIndex位置   
        items[putIndex] = x;
        //让putIndex++
        putIndex = inc(putIndex);
        //将队列的大小加1
        ++count;
        //唤醒其他正在处于等待状态的线程
        notEmpty.signal();
    }
```

**注:ArrayBlockingQueue其实是一个循环队列**
我们使用一个图来简单说明一下:

> `黄色表示数组中有元素`

![](https://upload-images.jianshu.io/upload_images/1684370-74dd3cf6fd2be432.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当再一次执行put的时候,其结果为：

![](https://upload-images.jianshu.io/upload_images/1684370-bdbe3fe4f43d5130.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时放入的元素会从头开始置，我们通过其incr方法更加清晰的看出其底层的操作：

```java
    /**
     * Circularly increment i.
     */
    final int inc(int i) {
        //当takeIndex的值等于数组的长度时,就会重新置为0，这个一个循环递增的过程
        return (++i == items.length) ? 0 : i;
    }
```

至此，ArrayBlockingQueue的核心部分就分析完了，其余的队列操作基本上都是换汤不换药的，此处不再一一列举。