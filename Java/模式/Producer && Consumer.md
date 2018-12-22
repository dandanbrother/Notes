

# 生产者、消费者模型

**准备：**

```java
// 抽象出接口
public interface Consumer {
  void consume() throws InterruptedException;
}

public interface Producer {
  void produce() throws InterruptedException;
}

// 实现抽象类
abstract class AbstractConsumer implements Consumer, Runnable {
  @Override
  public void run() {
    while (true) {
      try {
        consume();
      } catch (InterruptedException e) {
        e.printStackTrace();
        break;
      }
    }
  }
}

abstract class AbstractProducer implements Producer, Runnable {
  @Override
  public void run() {
    while (true) {
      try {
        produce();
      } catch (InterruptedException e) {
        e.printStackTrace();
        break;
      }
    }
  }
}

// 不同的模型实现中，生产者、消费者的具体实现也不同，所以需要为模型定义抽象工厂方法：
public interface Model {
  Runnable newRunnableConsumer();
  Runnable newRunnableProducer();
}

// 任务对象
public class Task {
  public int no;
  public Task(int no) {
    this.no = no;
  }
}
```

#### 实现一：BlockingQueue

BlockingQueue的写法最简单。核心思想是，**把并发和容量控制封装在缓冲区中**。而BlockingQueue的性质天生满足这个要求。

```java
public class BlockingQueueModel implements Model {
  private final BlockingQueue<Task> queue;
  private final AtomicInteger increTaskNo = new AtomicInteger(0);
  public BlockingQueueModel(int cap) {
    // LinkedBlockingQueue 的队列不 init，入队时检查容量；ArrayBlockingQueue 在创建时 init
    this.queue = new LinkedBlockingQueue<>(cap);
  }
  @Override
  public Runnable newRunnableConsumer() {
    return new ConsumerImpl();
  }
  @Override
  public Runnable newRunnableProducer() {
    return new ProducerImpl();
  }
  private class ConsumerImpl extends AbstractConsumer implements Consumer, Runnable {
    @Override
    public void consume() throws InterruptedException {
      Task task = queue.take();
      // 固定时间范围的消费，模拟相对稳定的服务器处理过程
      Thread.sleep(500 + (long) (Math.random() * 500));
      System.out.println("consume: " + task.no);
    }
  }
  private class ProducerImpl extends AbstractProducer implements Producer, Runnable {
    @Override
    public void produce() throws InterruptedException {
      // 不定期生产，模拟随机的用户请求
      Thread.sleep((long) (Math.random() * 1000));
      Task task = new Task(increTaskNo.getAndIncrement());
      System.out.println("produce: " + task.no);
      queue.put(task);
    }
  }
  public static void main(String[] args) {
    Model model = new BlockingQueueModel(3);
    for (int i = 0; i < 2; i++) {
      new Thread(model.newRunnableConsumer()).start();
    }
    for (int i = 0; i < 5; i++) {
      new Thread(model.newRunnableProducer()).start();
    }
  }
}
```

**由于操作“出队/入队+日志输出”不是原子的，所以上述日志的绝对顺序与实际的出队/入队顺序有出入，但对于同一个任务号`task.no`，其consume日志一定出现在其produce日志之后，即：同一任务的消费行为一定发生在生产行为之后。**

#### 实现二：wait && notify

```java
public class WaitNotifyModel implements Model {
  private final Object BUFFER_LOCK = new Object();
  private final Queue<Task> buffer = new LinkedList<>();
  private final int cap;
  private final AtomicInteger increTaskNo = new AtomicInteger(0);
  public WaitNotifyModel(int cap) {
    this.cap = cap;
  }
  @Override
  public Runnable newRunnableConsumer() {
    return new ConsumerImpl();
  }
  @Override
  public Runnable newRunnableProducer() {
    return new ProducerImpl();
  }
    
  private class ConsumerImpl extends AbstractConsumer implements Consumer, Runnable {
    @Override
    public void consume() throws InterruptedException {
      synchronized (BUFFER_LOCK) {
        while (buffer.size() == 0) {
          BUFFER_LOCK.wait();
        }
        Task task = buffer.poll();
        assert task != null;
        // 固定时间范围的消费，模拟相对稳定的服务器处理过程
        Thread.sleep(500 + (long) (Math.random() * 500));
        System.out.println("consume: " + task.no);
        BUFFER_LOCK.notifyAll();
      }
    }
  }
    
  private class ProducerImpl extends AbstractProducer implements Producer, Runnable {
    @Override
    public void produce() throws InterruptedException {
      // 不定期生产，模拟随机的用户请求
      Thread.sleep((long) (Math.random() * 1000));
      synchronized (BUFFER_LOCK) {
        while (buffer.size() == cap) {
          BUFFER_LOCK.wait();
        }
        Task task = new Task(increTaskNo.getAndIncrement());
        buffer.offer(task);
        System.out.println("produce: " + task.no);
        BUFFER_LOCK.notifyAll();
      }
    }
  }
  public static void main(String[] args) {
    Model model = new WaitNotifyModel(3);
    for (int i = 0; i < 2; i++) {
      new Thread(model.newRunnableConsumer()).start();
    }
    for (int i = 0; i < 5; i++) {
      new Thread(model.newRunnableProducer()).start();
    }
  }
}
```

#### 实现三、四：创建锁实现线程管理。当使用两个锁时，需要互相通知。