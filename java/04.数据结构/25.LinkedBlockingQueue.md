## 前言

在前面的文章ArrayBlockingQueue中，已经对JDK中的BlockingQueue中的做了一个回顾，同时对ArrayBlockingQueue中的核心方法作了说明，而LinkedBlockingQueue作为JDK中BlockingQueue家族系列中一员，由于其作为固定大小线程池(Executors.newFixedThreadPool())底层所使用的阻塞队列，分析它的目的主要在于2点：

1. 与ArrayBlockingQueue进行类比学习，加深各种数据结构的理解
2. 了解底层实现，能够更好地理解每一种阻塞队列对线程池性能的影响，做到真正的知其然，且知其所以然

- 源码分析LinkedBlockingQueue的实现
- 与ArrayBlockingQueue进行比较
- 说明为什么选择LinkedBlockingQueue作为固定大小的线程池的阻塞队列

## LinkedBlockingQueue深入分析

LinkedBlockingQueue，见名之意，它是由一个基于链表的阻塞队列，首先看一下的核心组成：

```java
    // 所有的元素都通过Node这个静态内部类来进行存储，这与LinkedList的处理方式完全一样
    static class Node<E> {
        //使用item来保存元素本身
        E item;
        //保存当前节点的后继节点
        Node<E> next;
        Node(E x) { item = x; }
    }
    /**
        阻塞队列所能存储的最大容量
        用户可以在创建时手动指定最大容量,如果用户没有指定最大容量
        那么最默认的最大容量为Integer.MAX_VALUE.
    */
    private final int capacity;

    /** 
        当前阻塞队列中的元素数量
        PS:如果你看过ArrayBlockingQueue的源码,你会发现
        ArrayBlockingQueue底层保存元素数量使用的是一个
        普通的int类型变量。其原因是在ArrayBlockingQueue底层
        对于元素的入队列和出队列使用的是同一个lock对象。而数
        量的修改都是在处于线程获取锁的情况下进行操作，因此不
        会有线程安全问题。
        而LinkedBlockingQueue却不是，它的入队列和出队列使用的是两个    
        不同的lock对象,因此无论是在入队列还是出队列，都会涉及对元素数
        量的并发修改，(之后通过源码可以更加清楚地看到)因此这里使用了一个原子操作类
        来解决对同一个变量进行并发修改的线程安全问题。
    */
    private final AtomicInteger count = new AtomicInteger(0);

    /**
     * 链表的头部
     * LinkedBlockingQueue的头部具有一个不变性:
     * 头部的元素总是为null，head.item==null   
     */
    private transient Node<E> head;

    /**
     * 链表的尾部
     * LinkedBlockingQueue的尾部也具有一个不变性:
     * 即last.next==null
     */
    private transient Node<E> last;

    /**
     元素出队列时线程所获取的锁
     当执行take、poll等操作时线程需要获取的锁
    */
    private final ReentrantLock takeLock = new ReentrantLock();

    /**
    当队列为空时，通过该Condition让从队列中获取元素的线程处于等待状态
    */
    private final Condition notEmpty = takeLock.newCondition();

    /** 
      元素入队列时线程所获取的锁
      当执行add、put、offer等操作时线程需要获取锁
    */
    private final ReentrantLock putLock = new ReentrantLock();

    /** 
     当队列的元素已经达到capactiy，通过该Condition让元素入队列的线程处于等待状态
    */
    private final Condition notFull = putLock.newCondition();
```

通过上面的分析，我们可以发现LinkedBlockingQueue在入队列和出队列时使用的不是同一个Lock，这也意味着它们之间的操作不会存在互斥操作。在多个CPU的情况下，它们可以做到真正的在同一时刻既消费、又生产，能够做到并行处理。

下面让我们看下LinkedBlockingQueue的构造方法：

```java
    /**
     * 如果用户没有显示指定capacity的值，默认使用int的最大值
     */
    public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }
    /**
     可以看到,当队列中没有任何元素的时候,此时队列的头部就等于队列的尾部,
     指向的是同一个节点,并且元素的内容为null
    */
    public LinkedBlockingQueue(int capacity) {
        if (capacity <= 0) throw new IllegalArgumentException();
        this.capacity = capacity;
        last = head = new Node<E>(null);
    }

    /*
    在初始化LinkedBlockingQueue的时候，还可以直接将一个集合
    中的元素全部入队列，此时队列最大容量依然是int的最大值。
    */
    public LinkedBlockingQueue(Collection<? extends E> c) {
        this(Integer.MAX_VALUE);
        final ReentrantLock putLock = this.putLock;
        //获取锁
        putLock.lock(); // Never contended, but necessary for visibility
        try {
            //迭代集合中的每一个元素,让其入队列,并且更新一下当前队列中的元素数量
            int n = 0;
            for (E e : c) {
                if (e == null)
                    throw new NullPointerException();
                if (n == capacity)
                    throw new IllegalStateException("Queue full");
                //参考下面的enqueue分析        
                enqueue(new Node<E>(e));
                ++n;
            }
            count.set(n);
        } finally {
            //释放锁
            putLock.unlock();
        }
    }

    /**
     * 我去，这代码其实可读性不怎么样啊。
     * 其实下面的代码等价于如下内容:
     * last.next=node;
     * last = node;
     * 其实也没有什么花样:
       就是让新入队列的元素成为原来的last的next，让进入的元素称为last
     *
     */
    private void enqueue(Node<E> node) {
        // assert putLock.isHeldByCurrentThread();
        // assert last.next == null;
        last = last.next = node;
    }
```

在分析完LinkedBlockingQueue的核心组成之后,下面让我们再看下核心的几个操作方法,首先分析一下元素入队列的过程:

```java
    public void put(E e) throws InterruptedException {
        if (e == null) throw new NullPointerException();
        // Note: convention in all put/take/etc is to preset local var

        /*注意上面这句话,约定所有的put/take操作都会预先设置本地变量,
        可以看到下面有一个将putLock赋值给了一个局部变量的操作
        */
        int c = -1;
        Node<E> node = new Node(e);
        /* 
         在这里首先获取到putLock,以及当前队列的元素数量
         即上面所描述的预设置本地变量操作
        */
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        /*
            执行可中断的锁获取操作,即意味着如果线程由于获取
            锁而处于Blocked状态时，线程是可以被中断而不再继
            续等待，这也是一种避免死锁的一种方式，不会因为
            发现到死锁之后而由于无法中断线程最终只能重启应用。
        */
        putLock.lockInterruptibly();
        try {
            /*
            当队列的容量到底最大容量时,此时线程将处于等待状
            态，直到队列有空闲的位置才继续执行。使用while判
            断依旧是为了放置线程被"伪唤醒”而出现的情况,即当
            线程被唤醒时而队列的大小依旧等于capacity时，线
            程应该继续等待。
            */
            while (count.get() == capacity) {
                notFull.await();
            }
            //让元素进行队列的末尾,enqueue代码在上面分析过了
            enqueue(node);
            //首先获取原先队列中的元素个数,然后再对队列中的元素个数+1.
            c = count.getAndIncrement();
            /*注:c+1得到的结果是新元素入队列之后队列元素的总和。
            当前队列中的总元素个数小于最大容量时,此时唤醒其他执行入队列的线程
            让它们可以放入元素,如果新加入元素之后,队列的大小等于capacity，
            那么就意味着此时队列已经满了,也就没有必须要唤醒其他正在等待入队列的线程,因为唤醒它们之后，它们也还是继续等待。
            */
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            //完成对锁的释放
            putLock.unlock();
        }
        /*当c=0时，即意味着之前的队列是空队列,出队列的线程都处于等待状态，
        现在新添加了一个新的元素,即队列不再为空,因此它会唤醒正在等待获取元素的线程。
        */
        if (c == 0)
            signalNotEmpty();
    }

    /*
    唤醒正在等待获取元素的线程,告诉它们现在队列中有元素了
    */
    private void signalNotEmpty() {
        final ReentrantLock takeLock = this.takeLock;
        takeLock.lock();
        try {
            //通过notEmpty唤醒获取元素的线程
            notEmpty.signal();
        } finally {
            takeLock.unlock();
        }
    }
```

看完put方法，下面再看看下offer是如何处理的方法：

```java
    /**
    在BlockingQueue接口中除了定义put方法外(当队列元素满了之后就会阻塞，
    直到队列有新的空间可以方法线程才会继续执行)，还定义一个offer方法，
    该方法会返回一个boolean值，当入队列成功返回true,入队列失败返回false。
    该方法与put方法基本操作基本一致，只是有细微的差异。
    */
     public boolean offer(E e) {
        if (e == null) throw new NullPointerException();
        final AtomicInteger count = this.count;
        /*
            当队列已经满了，它不会继续等待,而是直接返回。
            因此该方法是非阻塞的。
        */
        if (count.get() == capacity)
            return false;
        int c = -1;
        Node<E> node = new Node(e);
        final ReentrantLock putLock = this.putLock;
        putLock.lock();
        try {
            /*
            当获取到锁时，需要进行二次的检查,因为可能当队列的大小为capacity-1时，
            两个线程同时去抢占锁，而只有一个线程抢占成功，那么此时
            当线程将元素入队列后，释放锁，后面的线程抢占锁之后，此时队列
            大小已经达到capacity，所以将它无法让元素入队列。
            下面的其余操作和put都一样，此处不再详述
            */
            if (count.get() < capacity) {
                enqueue(node);
                c = count.getAndIncrement();
                if (c + 1 < capacity)
                    notFull.signal();
            }
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();
        return c >= 0;
    }
```

BlockingQueue还定义了一个限时等待插入操作，即在等待一定的时间内，如果队列有空间可以插入，那么就将元素入队列，然后返回true,如果在过完指定的时间后依旧没有空间可以插入，那么就返回false，下面是限时等待操作的分析:

```java
        /**
         通过timeout和TimeUnit来指定等待的时长
         timeout为时间的长度,TimeUnit为时间的单位
        */
        public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        if (e == null) throw new NullPointerException();
        //将指定的时间长度转换为毫秒来进行处理
        long nanos = unit.toNanos(timeout);
        int c = -1;
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
            while (count.get() == capacity) {
                //如果等待的剩余时间小于等于0，那么直接返回
                if (nanos <= 0)
                    return false;
            /*
              通过condition来完成等待，此时当前线程会完成锁的，并且处于等待状态
              直到被其他线程唤醒该线程、或者当前线程被中断、
              等待的时间截至才会返回，该返回值为从方法调用到返回所经历的时长。
              注意：上面的代码是condition的awitNanos()方法的通用写法，
              可以参看Condition.awaitNaos的API文档。
              下面的其余操作和put都一样，此处不再详述
            */
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(new Node<E>(e));
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();
        return true;
    }
```

通过上面的分析，我们应该比较清楚地知道了LinkedBlockingQueue的入队列的操作，其主要是通过获取到putLock锁来完成，当队列的数量达到最大值，此时会导致线程处于阻塞状态或者返回false(根据具体的方法来看)；如果队列还有剩余的空间，那么此时会新创建出一个Node对象，将其设置到队列的尾部，作为LinkedBlockingQueue的last元素。

在分析完入队列的过程之后，我们接下来看看LinkedBlockingQueue出队列的过程；由于BlockingQueue的方法都具有对称性，此处就只分析take方法的实现，其余方法的实现都如出一辙：

```java
 public E take() throws InterruptedException {
        E x;
        int c = -1;
        final AtomicInteger count = this.count;
        final ReentrantLock takeLock = this.takeLock;
        //通过takeLock获取锁，并且支持线程中断
        takeLock.lockInterruptibly();
        try {
            //当队列为空时，则让当前线程处于等待
            while (count.get() == 0) {
                notEmpty.await();
            }
            //完成元素的出队列
            x = dequeue();
            /*            
               队列元素个数完成原子化操作-1,可以看到count元素会
               在插入元素的线程和获取元素的线程进行并发修改操作。
            */
            c = count.getAndDecrement();
            /*
              当一个元素出队列之后，队列的大小依旧大于1时
              当前线程会唤醒其他执行元素出队列的线程,让它们也
              可以执行元素的获取
            */
            if (c > 1)
                notEmpty.signal();
        } finally {
            //完成锁的释放
            takeLock.unlock();
        }
        /*
            当c==capaitcy时，即在获取当前元素之前，
            队列已经满了，而此时获取元素之后，队列就会
            空出一个位置，故当前线程会唤醒执行插入操作的线
            程通知其他中的一个可以进行插入操作。
        */
        if (c == capacity)
            signalNotFull();
        return x;
    }


    /**
     * 让头部元素出队列的过程
     * 其最终的目的是让原来的head被GC回收，让其的next成为head
     * 并且新的head的item为null.
     * 因为LinkedBlockingQueue的头部具有一致性:即元素为null。
     */
    private E dequeue() {
        Node<E> h = head;
        Node<E> first = h.next;
        h.next = h; // help GC
        head = first;
        E x = first.item;
        first.item = null;
        return x;
    }
```

![](http://upload-images.jianshu.io/upload_images/1684370-9bea30e290a8822c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于LinkedBlockingQueue的源码分析就到这里，下面让我们将LinkedBlockingQueue与ArrayBlockingQueue进行一个比较。

## LinkedBlockingQueue与ArrayBlockingQueue的比较

ArrayBlockingQueue由于其底层基于数组，并且在创建时指定存储的大小，在完成后就会立即在内存分配固定大小容量的数组元素，因此其存储通常有限，故其是一个**“有界“**的阻塞队列；

而LinkedBlockingQueue可以由用户指定最大存储容量，也可以无需指定，如果不指定则最大存储容量将是Integer.MAX_VALUE，即可以看作是一个**“无界”**的阻塞队列，由于其节点的创建都是动态创建，并且在节点出队列后可以被GC所回收，因此其具有灵活的伸缩性。但是由于ArrayBlockingQueue的有界性，因此其能够更好的对于性能进行预测，而LinkedBlockingQueue由于没有限制大小，当任务非常多的时候，不停地向队列中存储，就有可能导致内存溢出的情况发生。

其次，ArrayBlockingQueue中在入队列和出队列操作过程中，使用的是同一个lock，所以即使在多核CPU的情况下，其读取和操作的都无法做到并行，而LinkedBlockingQueue的读取和插入操作所使用的锁是两个不同的lock，它们之间的操作互相不受干扰，因此两种操作可以并行完成，故LinkedBlockingQueue的吞吐量要高于ArrayBlockingQueue。

## 选择LinkedBlockingQueue的理由

```java
    /**
        下面的代码是Executors创建固定大小线程池的代码，其使用了
        LinkedBlockingQueue来作为任务队列。
    */
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```

JDK中选用LinkedBlockingQueue作为阻塞队列的原因就在于其**无界性**。因为线程大小固定的线程池，其线程的数量是不具备伸缩性的，当任务非常繁忙的时候，就势必会导致所有的线程都处于工作状态，如果使用一个有界的阻塞队列来进行处理，那么就非常有可能很快导致队列满的情况发生，从而导致任务无法提交而抛出RejectedExecutionException，而使用无界队列由于其良好的存储容量的伸缩性，可以很好的去缓冲任务繁忙情况下场景，即使任务非常多，也可以进行动态扩容，当任务被处理完成之后，队列中的节点也会被随之被GC回收，非常灵活。