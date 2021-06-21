# 多线程效率问题

- 单核CPU下，多线程并不能实际的提高程序运行的效率，但是能够使不同的线程轮询使用cpu
- 多核下，可以并行的执行，可以提高效率

# 线程的创建

1. 直接继承Thread的方式

```java
public static void main(String[] args) {
    new MyThread().start();
    log.debug("主线程运行结束...");
}

static class MyThread extends Thread {
    @Override
    public void run() {
        log.debug("线程运行中....");
    }
}
```

2. 使用Runnable接口的方式创建（建议使用）

```java
public static void main(String[] args) {
    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            log.debug("线程运行中....");
        }
    };
    new Thread(runnable).start();
    log.debug("主线程运行中.....");
}
```

- Thread 方式和Runnable的方式原理

Runable方式中，将Runable的实现类赋值了target

```java
this.target = target;
```

最终在run方法中，可以看到，执行的是Runnable的实现的方法

```java
@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```

Thread 方式中，执行的是子类的run方法，也就是我们重写的方法，所以两种方式原理是一样的

3. future模式,future模式能够阻塞的等待异步线程的结果，他要配合Callable的方式

```java
FutureTask<Integer> futureTask = new FutureTask<>(new Callable<Integer>() {
    @Override
    public Integer call() throws Exception {
        log.debug("future执行中.....");
        Thread.sleep(1000);
        return 1000;
    }
});
new Thread(futureTask).start();
log.debug("main 线程执行中.....");
log.debug("获取到future的结果：{}", futureTask.get());
```

# 线程切换

线程上下文切换原因：

1. 线程的cpu时间片用完
2. 垃圾回收
3. 有更高优先级的线程需要运行
4. 线程自己调用了sleep yield等方法

- 当线程切换时候，需要**程序计数器**保存当前线程的状态（记录执行指令的地址以及栈帧信息）
- 频繁切换上下文会影响性能

# 线程常见方法

## sleep

- 将线程有Runnable转为Time Waiting状态
- 其他线程可以使用interrupt方法打断正在sleep的线程，这时，线程抛出java.lang.InterruptedException异常

```java
Thread.sleep(1000);
```

- 建议使用TimeUnit来代替Thread，如：

```java
TimeUnit.SECONDS.sleep(3);
```

## yield

- 可以使线程从Running进入Runnable状态 

## 防止Cpu100%的方案

- 死循环中，不让空转一直耗费cpu，加一个sleep可以让出cpu使用权给其他程序

```java
while (true) {
    TimeUnit.SECONDS.sleep(1);
}
```

## join

- 阻塞等待线程结束

```java
//主线程同步等待t1线程
Thread t1 = new Thread(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    log.debug("t1 执行结束");
});
t1.start();
//等待t1线程结束后，在执行下面的代码
t1.join();
log.debug("main 执行结束");
```

- 有时效的等待
  - java.lang.Thread#join(long)
  - 最多等待long毫秒

## interrupt

- 打断sleep，wait, join 的线程
- 当睡眠中打断后，打断标识：false，正常过程，设置打断，标识为true

```java
Thread t1 = new Thread(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    log.debug("t1 执行结束");
});
t1.start();
t1.interrupt();
log.debug("打断标识： {}", t1.isInterrupted());
```

输出结果

```tex
打断标识： true
t1 执行结束
```

-  isInterrupted:判断是否打断，执行后不清除标记
-  Thread.interrupted()：返回打断标记，但是返回打断标记以后会清除打断标记

### interrupt对park的影响

- park作用是让线程阻塞

```java
Thread t1 = new Thread(() -> {
    //线程阻塞，等待打断
    LockSupport.park();
    log.debug("interrupt: {}", Thread.currentThread().isInterrupted());
    //线程不阻塞，因为线程打断标识=true
    LockSupport.park();
    //清除打断标识
    log.debug("interrupt: {}", Thread.interrupted());
    //线程阻塞
    LockSupport.park();
    log.debug("interrupt: {}", Thread.currentThread().isInterrupted());
});
t1.start();
Thread.sleep(1000);
t1.interrupt();
Thread.sleep(1000);
```

## 两阶段终止模式

- 在线程t1中，优雅的结束线程t2(优雅指的是：让t2做完自己该做的事情后结束线程)

### 错误思路

- 使用线程对象的stop方法
  - 会真正的杀死线程，当线程锁住某个资源时被杀死，那么此线程会没法释放锁
- 使用System.exit方法
  - 会停止整个程序

### 常见解决方案

- 如：需要做一个监控，当需要停止这个监控时，只需要对这个线程设置打断
- 睡眠2秒为了避免循环耗尽资源
- 出现异常设置打断是因为：睡眠阶段打断标识会抛出异常，并且打断标识为false

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/java/concurrent/1624007279803.png)

```java
private static Thread monitor;
public static void main(String[] args) throws InterruptedException {
    start();
    TimeUnit.SECONDS.sleep(10);
    stop();
}
public static void start() {
    monitor = new Thread(() -> {
        while (true) {
            Thread thread = Thread.currentThread();
            if(thread.isInterrupted()) {
                log.debug("==>处理一些事情后退出线程...");
                break;
            }
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                thread.interrupt();
            }
            log.debug("==>正在执行监控记录...");
        }
    });
    monitor.start();
}
public static void stop() {
    monitor.interrupt();
}
```

# 主线程和守护线程

- 正常情况下，Java会等待所有线程结束，程序才会结束
- 特殊情况：守护线程不管有没有结束，主线程结束，都会强制结束
- 垃圾回收线程就是常见的守护线程

# 线程状态

## 从API层面

```java
public enum State {
    /**
     * 初始状态
     */
    NEW,

    /**
     * 保护运行/可运行/阻塞（操作系统的阻塞，如：io的阻塞accept）状态
     */
    RUNNABLE,

    /**
     * 阻塞状态：被锁住了
     */
    BLOCKED,

    /**
     * 阻塞：调用了wating/join
     */
    WAITING,

    /**
     * 阻塞：调用sleep
     */
    TIMED_WAITING,

    /**
     * Thread state for a terminated thread.
     * The thread has completed execution.
     */
    TERMINATED;
}
```

## 从操作系统角度

## 线程状态的转换

### NEW --> RUNNABLE  

1. NEW --> RUNNABLE  

### RUNNABLE <--> WAITING  

1. 调用 obj.wait() 方法时  
2. 调用 obj.notify() ， obj.notifyAll() ， t.interrupt()  
3. 当前线程调用 LockSupport.park() 方法会让当前线程从 RUNNABLE --> WAITING  

### RUNNABLE <--> WAITING  

1. 当前线程调用 t.join() 方法时，当前线程从 RUNNABLE --> WAITING  

### RUNNABLE <--> TIMED_WAITING  

1. 当前线程调用 Thread.sleep(long n) ，当前线程从 RUNNABLE --> TIMED_WAITING  

# 共享模型-管理

## 对象头

- 一般我们造的对象，都由对象头和对象的成员属性组成

## Monitor

- monitor是操作系统提供的对象
- 当对某个对象加synchronized（obj）的时候，obj的对象头会尝试加上一个monitor的指针
- monitor里面的owner属性指向抢到锁的线程
- 此时另外一个线程来抢这个锁，则monitor的的EntryList指向抢锁的线程

- 当线程执行完，将EntryList中的线程全部唤醒，继续抢锁

![](https://gitee.com/xiaojihao/xiaoxiao/raw/master/image/java/jvm/20210619104359.png)

## 轻量级锁

轻量级锁的使用场景:如果一个对象虽然有多线程访问，但多线程访问的时间是错开的（也就是没有竞争)，那么可以使用轻量级锁来优化。

- 轻量级锁是没有阻塞的概念的

1. 线程0获取了obj的锁，在thread0对象里存储一个锁记录（obj地址），thread0的对象头的lock record与obj对象头交换，此时obj对象头锁状态变成00（表示轻量锁）

![image-20210620133724895](https://gitee.com/xiaojihao/pubImage/raw/master/image/java/concurrent/image-20210620133724895.png)

2. 如果有其他线程来已经持有了轻量级的锁，那么表明有竞争，进入锁膨胀过程
3. 当退出synchronized代码块（解锁时）锁记录的值不为null，这时使用cas将 Mark Word的值恢复给对象头
   1. 成功，解锁成功
   2. 失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程

## 锁膨胀

如果在尝试加轻量级锁的过程中，CAS操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁(有竞争)，这时需要进行锁膨胀，将轻量级锁变为重量级锁。

1. 当Thread1 去申请轻量锁的时候，发现obj锁状态是00（轻量级）
2. 此时进入锁膨胀流程
   1. obj对象申请monitor锁，让obj指向monitor重量级锁地址，owner指向已经申请好锁的thread0,entrylist阻塞队列存储一个thread1
   2. monitor地址状态变为10（重量级锁）

![image-20210620155511181](https://gitee.com/xiaojihao/pubImage/raw/master/image/java/concurrent/image-20210620155511181.png)

3. thread0解锁的时候，发现锁膨胀，则它进入重量级锁解锁流程
   1. owner变为空
   2. 唤醒entrylist的线程

## 自旋优化

重量级锁竞争的时候，还可以使用自旋来进行优化，如果当前线程自旋成功(即这时候持锁线程已经退出了同步块，释放了锁)，这时当前线程就可以避免阻塞。

- 在Java6之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会高，就多自旋几次;反之，就少自旋甚至不自旋，总之，比较智能。

## 偏向锁

- 轻量级锁在没有竞争时(就自己这个线程，每次重入仍然需要执行CAS(每一次都要看一下是不是自己线程的锁)操作。
- Java 6中引入了偏向锁来做进一步优化:只有第一次使用CAS将**线程ID设置到对象的 Mark Word头**，之后发现这个线程ID是自己的就表示没有竞争，不用重新CAS。以后只要不发生竞争，这个对象就归该线程所有

重入锁示例：

```java
static Object LOCK = new Object();
public void method1() {
    synchronized (LOCK) {
        method2();
    }
}
public void method2() {
    synchronized (LOCK) {
    }
}
```

### 偏向锁概念

![image-20210620163259171](https://gitee.com/xiaojihao/pubImage/raw/master/image/java/concurrent/image-20210620163259171.png)

- 由图可见：
  - 如果开启了偏向锁（默认开启)，那么对象创建后，markword值为0x05即最后3位为101，这时它的thread、epoch、age都为0
  - 当加锁之后，前54位就是线程id，后三位为101
- 如果调用hashcode方法后，就是撤销该对象的偏向状态

### 撤销偏向锁的场景

1. 调用hashcode方法
2. 其他线程使用对象（两个线程锁同一个对象，顺序执行场景）
3. 调用wait/notify

## 锁消除

- JIT即使编译器会将锁进行优化
- 当锁的对象没有竞争时，会对锁进行消除优化

## wait l notify

- Monitor中Owner线程发现条件不满足，调用wait方法，即可进入WaitSet变为WAITING状态
- WAITING线程会在Owner线程调用notify或notifyAll 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入EntryList重新竞争

### Api

- obj.wait()让进入object 监视器的线程到waitSet等待

```java
synchronized (LOCK) {
    //当前线程必须要获得了这个锁才能调用
    LOCK.wait();
}
```

- notify 唤醒一个线程，notifyall唤醒所有的线程

### sleep和wait区别

- sleep不会释放锁，wait会释放锁
- sleep是 Thread方法，而wait是Object的方法
- sleep不需要强制和synchronized配合使用，但wait需要和synchronized一起用

### 虚假唤醒

- 同一个锁下面，其他线程调用了notifyAll，但是当前线程并不满足唤醒条件，这时，就导致了虚假唤醒

### 使用正确姿势

```java
synchronized (LOCK) {
    while(条件不成立) {
    	LOCK.wait();
    }
    //做事情
}
//另一个线程
synchronized (LOCK) {
    LOCK.notifyall();
}
```

## park$unpark

```java
//暂停当前线程
LockSupport.park();
//回去暂停的线程
LockSupport.unpark(暂停的线程);
```

- unpark可以在park之前调用
  - 如下：park之后能里面通过

```java
Thread t1 = new Thread(() -> {
    sleep(2000);
    log.debug("==>park");
    LockSupport.park();
    log.debug("==>has unpark");
});
t1.start();
sleep(1000);
log.debug("==>unpark");
LockSupport.unpark(t1);
```

- park是以线程为单位，更加精准；notify是所有线程都通知

### 原理

- 每个thread都有一个park对象

1. 当前线程调用Unsafe.park()方法
2. 检查_counter，如果counter=0，这时，获得_mutex互斥锁
3. 调用unpark，唤醒thread，判断条件

![](https://gitee.com/xiaojihao/pubImage/raw/master/image/java/concurrent/image-20210621113043839.png)

# 保护性暂停模式

- 用在一个线程等待另一个线程的执行结果

![image-20210620230826069](https://gitee.com/xiaojihao/pubImage/raw/master/image/java/concurrent/image-20210620230826069.png)

```java
public class TestGuardedObject {

    public static void main(String[] args) {
        GuardedObject object = new GuardedObject();
        new Thread(() -> {
            log.debug("获取到结果：{}", object.get());
        }).start();

        new Thread(() -> {
            log.debug("开始运行设置结果...");
            object.set("laoxiao");
        }).start();
    }

}

class GuardedObject {
    private Object response;

    public Object get() {
        synchronized (this) {
            while (response == null) {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            return response;
        }
    }
    public void set(Object object) {
        synchronized (this) {
            try {
                TimeUnit.SECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            this.response = object;
            this.notify();
        }
    }
}
```

# 消费者生产者模式

- 与前面的保护性暂停中的GuardObject 不同，不需要产生结果和消费结果的线程一一对应

代码示例中，一个消费者，对应多个生产者

```java
@Slf4j
public class TestMessage {

    public static void main(String[] args) {
        MessageQueue<Message> queue = new MessageQueue<>(2);
        for(int i=0; i<3; i++) {
            int id = i;
            new Thread(() -> {
                queue.put(new Message(id, "消息："+id));
            }).start();
        }

        new Thread(() -> {
            while (true) {
                Message message = queue.pop();
                log.debug("获取到消息: {}", message);
            }
        }).start();
    }
}

class MessageQueue<T> {
    private LinkedList<T> list;

    private int capacity;

    public MessageQueue(int capacity) {
        this.list = new LinkedList();
        this.capacity = capacity;
    }

    public void put(T message) {
        synchronized (list) {
            while (capacity < list.size()) {
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            list.addLast(message);
            list.notifyAll();
        }

    }

    public T pop() {
        synchronized (list) {
            while (list.size() <= 0) {
                try {
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T message = list.remove();
            list.notifyAll();
            return message;
        }
    }

}

@ToString
@Getter
@Setter
@AllArgsConstructor
class Message {
    private int id;
    private String message;
}
```

# ReentrantLock

## 特点

- 可中断（如A线程持有锁，B线程可以中断他）
- 设置超时时间
- 可以设置为公平锁（可以防止线程饥饿问题）
- 支持多个条件变量

- 可重入

```java
public static void method1() {
    lock.lock();
    try {
        method2();
    } finally {
        lock.unlock();
    }
}
public static void method2() {
    lock.lock();
    try {

    } finally {
        lock.unlock();
    }
}
```

- 可打断

```java
//尝试去获取锁，当有竞争时，进入阻塞队列
//当其他线程调用打断是，抛出异常
lock.lockInterruptibly();
```

- 锁超时
  1. trylock也可以被打断，被打断抛出异常
  2. 超时未获取到锁则返回false

```java
lock.tryLock(1, TimeUnit.SECONDS)
```

## 公平锁

- 按照获取尝试获取锁的顺序给予资源
- 通过构造方法创建公平锁

```java
ReentrantLock lock = new ReentrantLock(true);
```

- 公平锁会降低并发度

## 条件变量

- 类似synchronized的wait
- 使用方式：

1. 创建某个条件（将来调用await方法的线程都会进入这个条件中阻塞）
2. 调用await方法，进入阻塞
3. 另外一个线程调用signal唤醒，线程去竞争锁

```
//一把锁可以创建多个条件
Condition condition1 = lock.newCondition();
Condition condition2 = lock.newCondition();

condition1.await();
//唤醒某一个锁
condition1.signal();
```

# 内存模型

## 可见性

- 问题：我们发现，代码中，并没有按照我们设想，线程暂停下来

```java
static boolean bool = true;
public static void main(String[] args) {
    new Thread(() -> {
        while (bool) {

        }
    }).start();
    sleep(1000);
    log.debug("暂停下来..");
    bool = false;
}
```

- 原因: JIT及时编译器会将bool拉到自己线程私有缓存的空间中（TLAB）,主线程修改的只是主堆空间的数据

解决方案

1. 关键字方式：volatile，可以保证获取bool不从高速缓存中获取

```java
volatile static boolean bool = true;
public static void main(String[] args) {
    new Thread(() -> {
        while (bool) {

        }
    }).start();
    sleep(1000);
    log.debug("暂停下来..");
    bool = false;
}
```

2. synchronized方式

```java
static boolean bool = true;
static Object lock = new Object();
public static void main(String[] args) {
    new Thread(() -> {
        while (true) {
            synchronized (lock) {
                if(!bool) {
                    break;
                }
            }
        }
    }).start();
    sleep(1000);
    log.debug("暂停下来..");
    bool = false;
}
```

### 读写屏障

- 写屏障
  - 保证写屏障之前的变量都同步主内存中
  - 能够保证写屏障之前的代码禁止指令重排

- 读屏障、
  - 保证读屏障之后的变量，都从主内存中读取

## 原子性

- volatile并不能保证原子性，它只保证了一个线程能及时的获取其他线程的值，但是并不能保证线程执行过程中，指令交错的问题
- synchronized能保证代码块的原子性，也同时保证了代码块里面的变量的可见性
- 所以volatile适合一个线程写，多个线程读的情况

## 有序性

- 在代码编译成字节码中，可能会产生指令重排的问题（JIT编译器做的优化）
- 如下可能发生指令重拍，因为不会影响执行结果

```java
int a = 1;
boolean b = true;
```

- volatile 可以禁止指令重排
  - 他可以禁止他之前的代码指令重排

```java
volatile boolean b 

int a = 1;
b = true;
```

## DCL(双端检锁)

- DCL是单例模式的一种方案，如下

```java
class SingletonDemo {
    private SingletonDemo singletonDemo;
    private SingletonDemo() {}
    private SingletonDemo getInstance() {
        if(singletonDemo == null) {
            synchronized (SingletonDemo.class) {
                if(singletonDemo == null) {
                    singletonDemo = new SingletonDemo();
                }
            }
        }
        return singletonDemo;
    }
}
```

- 此时，可能会发生指令重排的风险
- 解决方案：

```java
class SingletonDemo {
    private volatile SingletonDemo singletonDemo;
    private SingletonDemo() {}
    private SingletonDemo getInstance() {
        if(singletonDemo == null) {
            synchronized (SingletonDemo.class) {
                if(singletonDemo == null) {
                    singletonDemo = new SingletonDemo();
                }
            }
        }
        return singletonDemo;
    }
}
```

