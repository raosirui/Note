# Java并发编程



## 基础

### 并行跟并发有什么区别？

- 就好像我们去食堂打饭，**并行**就是每个人对应一个阿姨，同时打饭；
- 而并发就是一个阿姨，轮流给每个人打饭。



#### 你对线程安全的理解是什么？

**①、原子性**：确保当某个线程修改共享变量时，没有其他线程可以同时修改这个变量，即这个操作是不可分割的。

原子性可以通过**互斥锁（如 synchronized）或原子操作（如 AtomicInteger** 类中的方法）来保证。

**②、可见性**：确保一个线程对共享变量的修改可以立即被其他线程看到。

**volatile** 关键字可以保证了变量的修改对所有线程立即可见，并防止编译器优化导致的可见性问题。

**③、活跃性问题**：要确保线程不会因为死锁、饥饿、活锁等问题导致无法继续执行。



### 说说什么是进程和线程？

- 进程说简单点就是我们在电脑上启动的一个个应用，进程是操作系统资源分配的最小单位，它包括了程序、数据和进程控制块等。

- 线程是操作系统中调度的最小单位，它是进程中的独立执行单元。


多个线程可以共享同一个进程的资源，如内存和文件句柄，但每个线程都有自己独立的栈和寄存器。与进程相比，线程的创建和上下文切换开销更小，因此在需要并发执行任务时，多线程是一种常用的解决方案。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171527140.png" alt="image-20241217152736077" style="zoom:67%;" />



#### 如何理解协程？

协程（Coroutine）是一种轻量级的线程，允许程序在执行过程中暂停并且可以在之后的某个时间点恢复执行。Kotlin、Go原生支持。 **协程不需要线程切换和上下文切换的开销**

协程的概念最核心的点其实就是函数或者一段程序能够被**挂起**（说暂停其实也没啥问题），待会儿再**恢复**

线程相关的概念就是**抢占式多任务**（Preemptive multitasking），而与协程相关的是**协作式多任务**。

协程缺点：

- 无法使用 CPU 的多核
- 处处都要使用非阻塞代码
  

#### 说说线程的共享内存？

线程之间想要进行通信，可以通过**消息传递**和**共享内存**两种方法来完成。

Java 采用的是**共享内存**的并发模型。这个模型被称为 Java 内存模型，也就是 **JMM**，JMM 决定了一个线程对共享变量的写入何时对另外一个线程可见。

线程之间的**共享变量**存储在**主内存**（main memory）中，每个线程都有一个**私有的本地内存**（local memory），本地内存中存储了**共享变量的副本**。当然了，本地内存是 JMM 的一个抽象概念，并不真实存在。

[![深入浅出 Java 多线程：JMM](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171604097.png)](https://camo.githubusercontent.com/5ee9eafb248977e8a65dff728540e735844c89efa2502170bc4b6e524fc40832/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a6176617468726561642d32303234303331353131313134332e706e67)

线程 A 与线程 B 之间如要通信的话，必须要经历下面 2 个步骤：

- 线程 A 把本地内存 A 中的共享变量副本刷新到主内存中。
- 线程 B 到主内存中读取线程 A 刷新过的共享变量，再同步到自己的共享变量副本中。

[![深入浅出 Java 多线程：线程间通信](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171604803.png)](https://camo.githubusercontent.com/49c897ff41c8c178eb348b8a38c7921af88b6f0f9788d3b9b01ec25f5e8e2434/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a6176617468726561642d32303234303331353131313133302e706e67)



### :star:说说线程有几种创建方式？

> [!IMPORTANT]
>
> **线程的创建实际上三种方法：继承Thread类、实现Runnable接口、实现Callable接口，三种方式都是实现了创建 Thread 对象，然后将代传参数传递给 Thread 对象，然后调用 `start()` 方法启动线程，区别是继承Thread类只能单继承，实现Runnable接口可以实现多个接口，而实现Callable接口可以获取线程的返回值**



**源码级别的Thread.start() 方法**

> [!IMPORTANT]
>
> **调用Thread.start()方法后底层会先检查JVM的线程状态、资源分配等，再通过JNI调用本地方法 start0()，start0()方法首先会调用JVM内部create_vm_thread 方法创建java线程对象，然后调用 os::start_thread 方法启动操作系统级别的线程，新线程执行 thread_func 函数，最终调用 Java 线程的 run 方法。**

1. 调用 Thread.start() 方法。

2. JVM 检查线程状态，并通过JNI调用**本地方法** **start0()**。
3. start0() 方法**调用 JVM** 内部的 Threads::create_vm_thread 方法**创建 Java 线程对象**。
4. 调用 os::start_thread 方法，通过 pthread_create **创建操作系统级别的线程**。
5. **新线程执行 thread_func 函数，最终调用 Java 线程的 run 方法。**





Java 中创建线程主要有三种方式，分别为继承 Thread 类、实现 Runnable 接口、实现 Callable 接口。

[![二哥的 Java 进阶之路](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171608913.png)](https://camo.githubusercontent.com/8e586dc08ccdcf961a8cd980963e740baa5bced066ab095949004f1436c84b5c/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a6176617468726561642d32303234303430373137323635322e706e67)

第一种，继承 Thread 类，重写 `run()`方法，调用 `start()`方法启动线程。

```java
class ThreadTask extends Thread {
    public void run() {
        System.out.println("看完二哥的 Java 进阶之路，上岸了!");
    }

    public static void main(String[] args) {
        ThreadTask task = new ThreadTask();
        task.start();
    }
}
```

这种方法的缺点是，由于 Java **不支持多重继承**，所以如果类已经继承了另一个类，就不能使用这种方法了。



第二种，实现 Runnable 接口，重写 `run()` 方法，然后创建 Thread 对象，将 Runnable 对象作为参数传递给 Thread 对象，调用 `start()` 方法启动线程。

```java
class RunnableTask implements Runnable {
    public void run() {
        System.out.println("看完二哥的 Java 进阶之路，上岸了!");
    }

    public static void main(String[] args) {
        RunnableTask task = new RunnableTask();
        Thread thread = new Thread(task);
        thread.start();
    }
}
```

这种方法的优点是可以避免 Java 的单继承限制，并且更符合面向对象的编程思想，因为 Runnable 接口将任务代码和线程控制的代码解耦了。



第三种，实现 Callable 接口，重写 `call()` 方法，然后创建 FutureTask 对象，参数为 Callable 对象；紧接着创建 Thread 对象，参数为 FutureTask 对象，调用 `start()` 方法启动线程。

```java
class CallableTask implements Callable<String> {
    public String call() {
        return "看完二哥的 Java 进阶之路，上岸了!";
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CallableTask task = new CallableTask();
        FutureTask<String> futureTask = new FutureTask<>(task);
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println(futureTask.get());
    }
}
```

这种方法的优点是**可以获取线程的执行结果**。



#### 一个 8G 内存的系统最多能创建多少线程?

线程在创建的时候会被分配一个**虚拟机栈**，在 64 位操作系统中，**默认大小为 1M**。

8GB = 8 _ 1024 MB = 8 _ 1024 _ 1024 KB，所以一个 8G 内存的系统可以创建的线程数为 8 _ 1024 = 8192 个。但操作系统本身的运行也需要消耗一定的内存，所以实际上可以创建的线程数肯定会比 8192 少一些。



#### 启动一个 Java 程序，你能说说里面有哪些线程吗？

首先是 **main 线程**，这是程序开始执行的入口。

然后是**垃圾回收线程**，它是一个后台线程，负责回收不再使用的对象。

还有**编译器线程**，在及时编译中（JIT），负责把一部分热点代码编译后放到 codeCache 中，以提升程序的执行效率。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171621207.png" alt="二哥的 Java 进阶之路：JIT" style="zoom: 67%;" />](https://camo.githubusercontent.com/9b3cd235001fa07b1f683d113846b462011893e97063cad63221205e9e9d0de3/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a69742d32303234303130353138303635352e706e67)

扩展：

- `Thread: main (ID=1)` - 主线程，Java 程序启动时由 JVM 创建。
- `Thread: Reference Handler (ID=2)` - 这个线程是用来处理引用对象的，如软引用（SoftReference）、弱引用（WeakReference）和虚引用（PhantomReference）。负责清理被 JVM 回收的对象。
- `Thread: Finalizer (ID=3)` - 终结器线程，负责调用对象的 finalize 方法。对象在垃圾回收器标记为可回收之前，由该线程执行其 finalize 方法，用于执行特定的资源释放操作。
- `Thread: Signal Dispatcher (ID=4)` - 信号调度线程，处理来自操作系统的信号，将它们转发给 JVM 进行进一步处理，例如响应中断、停止等信号。
- `Thread: Monitor Ctrl-Break (ID=5)` - 监视器线程，通常由一些特定的 IDE 创建，用于在开发过程中监控和管理程序执行或者处理中断。



### 调用 start()方法时会执行 run()方法，那怎么不直接调用 run()方法？

```java
class MyThread extends Thread {
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        t1.start(); // 正确的方式，创建一个新线程，并在新线程中执行 run()
        //t1.run(); // 仅在主线程中执行 run()，没有创建新线程
    }
}
```

**`start()` 方法** 是用于启动一个新线程的方法。它会将当前线程的控制权交给操作系统来创建一个新的线程，并在这个新的线程中**自动调用 `run()` 方法**。

**`run()` 方法** 是线程执行的具体内容（业务逻辑），但它本质上是一个普通的方法。**直接调用 `run()` 方法**：不会创建新线程，`run()` 方法只是作为一个普通的类方法在**当前线程**中执行。

也就是说，`start()` 方法的调用会**告诉 JVM 准备好**所有必要的新线程结构，分配其所需资源，并调用线程的 `run()` 方法在这个新线程中执行。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171627133.png" alt="image-20241217162737070" style="zoom:67%;" />



### 线程有哪些常用的调度方法？

[![三分恶面渣逆袭：线程常用调度方法](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171630342.png)](https://camo.githubusercontent.com/c99483ed19c3b69b77a404f83fb82a5a92907ae995e8fa12370fbaa35549e328/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d362e706e67)

#### 说说线程等待与通知？

在 **Object 类**中有一些方法可以用于线程的等待与通知。

①、`wait()`：当一个线程 A 调用一个共享变量的 `wait()` 方法时，线程 A 会被阻塞挂起，直到发生下面几种情况才会返回 ：

- 其他线程调用了**共享对象** `notify()`或者 `notifyAll()` 方法；
- 其他线程调用了线程 A 的 `interrupt()` 方法，线程 A 抛出 InterruptedException 异常返回。

②、`wait(long timeout)` ：这个方法相比 `wait()` 方法多了一个超时参数，它的不同之处在于，如果线程 A 调用共享对象的 `wait(long timeout)`方法后，**没有在指定的 timeout 时间内被其它线程唤醒，那么这个方法还是会因为超时而返回。**

③、`wait(long timeout, int nanos)`，其内部调用的是 `wait(long timout)` 方法。

#### 说说线程唤醒：

①、`notify()`：一个线程 A 调用共享对象的 `notify()` 方法后，会唤醒一个在这个共享变量上调用 wait 系列方法后被挂起的线程。

一个共享变量上可能会有多个线程在等待，**具体唤醒哪个等待的线程是随机的**。

②、`notifyAll()`：不同于在共享变量上调用 `notify()` 方法会唤醒被阻塞到该共享变量上的一个线程，notifyAll 方法会唤醒所有在该共享变量上调用 wait 系列方法而被挂起的线程。

Thread 类还提供了一个 `join()` 方法，意思是如果一个线程 A 执行了 `thread.join()`，当前线程 A 会等待 thread 线程终止之后才从 `thread.join()` 返回。

#### 说说线程休眠

`sleep(long millis)`：Thread 类中的静态方法，当一个执行中的线程 A 调用了 Thread 的 sleep 方法后，线程 A 会暂时让出指定时间的执行权。

但是线程 A 所拥有的监视器资源，比如锁，还是持有不让出的。指定的睡眠时间到了后该方法会正常返回，接着参与 CPU 的调度，获取到 CPU 资源后就可以继续运行。

#### 说说让出优先权

`yield()`：Thread 类中的静态方法，当一个线程调用 yield 方法时，实际是在暗示线程调度器，当前线程请求让出自己的 CPU，但是线程调度器可能会“装看不见”忽略这个暗示。

#### 说说线程中断

Java 中的线程中断是一种线程间的**协作模式**，通过设置线程的中断标志并不能直接终止该线程的执行。被中断的线程会根据中断状态自行处理。

- `void interrupt()` 方法：中断线程，例如，当线程 A 运行时，线程 B 可以调用线程 `interrupt()` 方法来设置线程的中断标志为 true 并立即返回。设置标志仅仅是设置标志, 线程 B 实际并没有被中断，会继续往下执行。
- `boolean isInterrupted()` 方法： 检测当前线程是否被中断。
- `boolean interrupted()` 方法： 检测当前线程是否被中断，与 isInterrupted 不同的是，**该方法如果发现当前线程被中断，则会清除中断标志。**

stop 方法用来强制线程停止执行，目前已经处于废弃状态，因为 stop 方法会导致线程立即停止，可能会在不一致的状态下释放锁，破坏对象的一致性，导致难以发现的错误和资源泄漏。



### :star:线程有几种状态？

也就是说，线程的生命周期可以分为五个主要阶段：**新建、可运行、运行中、阻塞/等待、和终止**。线程在运行过程中会根据状态的变化在这些阶段之间切换。：

[![三分恶面渣逆袭：Java线程状态变化](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171639930.png)](https://camo.githubusercontent.com/885f4cff9ddc3ae4a834dbd983b4d9466e0035f85ee5bb5ca21234f231cb2552/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d372e706e67)



### 什么是线程上下文切换？

为了让用户感觉多个线程是在同时执行的， CPU 资源的分配采用了**时间片轮转**也就是给每个线程分配一个时间片，线程在时间片内占用 CPU 执行任务。**当线程使用完时间片后，就会处于就绪状态并让出 CPU 让其他线程占用，这就是上下文切换。**

[![三分恶面渣逆袭：上下文切换时机](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171641959.png)](https://camo.githubusercontent.com/e546c6438534ebefb5f51906ff038a2b2d4fc33544eae53559f75dd07bb95071/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d392e706e67)

#### 线程可以被多核调度吗？

当然可以，**操作系统的调度器**负责将线程分配给可用的 CPU 核心，从而实现并行处理。

多核处理器提供了并行执行多个线程的能力。**每个核心可以独立执行一个或多个线程**，操作系统的任务调度器会根据策略和算法，如优先级调度、轮转调度等，决定哪个线程何时在哪个核心上运行。



### 守护线程了解吗？

Java 中的线程分为两类，分别为 **daemon 线程（守护线程）和 user 线程（用户线程）**。

那么守护线程和用户线程有什么区别呢？区别之一是**当最后一个非守护线程束时， JVM 会正常退出**，而不管当前是否存在守护线程，也就是说守**护线程是否结束并不影响 JVM 退出**。换而言之，只要有一个用户线程还没结束，正常情况下 JVM 就不会退出。



### :star:线程间有哪些通信方式？

使用共享对象、`wait()` 和 `notify()` 方法、Exchanger 和 CompletableFuture。

①、**使用共享对象**，多个线程可以访问和修改同一个对象，从而实现信息的传递，比如说 **volatile** 和 **synchronized** 关键字。

- **关键字 volatile用来修饰成员变量，告知程序任何对该变量的访问均需要从共享内存中获取，而对它的改变必须同步刷新回共享内存，保证所有线程对变量访问的可见性。**
- 关键字 synchronized可以修饰方法，或者以同步代码块的形式来使用，确保多个线程在同一个时刻，只能有一个线程在执行某个方法或某个代码块。



②、**使用 wait() 和 notify()**，例如，生产者-消费者模式中，生产者生产数据，消费者消费数据，通过 `wait()` 和 `notify()` 方法可以实现生产和消费的协调。

一个线程调用共享对象的 `wait()` 方法时，它会进入该对象的等待池，并释放已经持有的该对象的锁，进入等待状态。

一个线程调用共享对象的 `notify()` 方法时，它会唤醒在该对象等待池中等待的一个线程，使其进入锁池，等待获取锁。

Condition 也提供了类似的方法，`await()` 负责等待、`signal()` 和 `signalAll()` 负责通知。

通常与锁（特别是 [ReentrantLock](https://javabetter.cn/thread/reentrantLock.html)）一起使用，为线程提供了一种等待某个条件成真的机制，并允许其他线程在该条件变化时通知等待线程。更灵活、更强大。



③、**使用 Exchanger**，Exchanger 是一个同步点，可以在两个线程之间交换数据。一个线程调用 `exchange()` 方法，将数据传递给另一个线程，同时接收另一个线程的数据。



④、**使用 CompletableFuture**，CompletableFuture 是 Java 8 引入的一个类，支持异步编程，允许线程在完成计算后将结果传递给其他线程。



### :star:请说说 sleep 和 wait 的区别？

- sleep 会让当前线程休眠，不涉及对象类，也**不需要获取对象的锁**，属于 Thread 类的方法；
- wait 会让获得对象锁的线程实现等待，要**提前获得对象的锁**，属于 Object 类的方法。

1、锁行为不同

当线程执行 sleep 方法时，它不会释放任何锁。也就是说，如果一个线程在持有某个对象的锁时调用了 sleep，它在睡眠期间仍然会持有这个锁。

2、使用条件不同

- `sleep()` 方法可以在任何地方被调用。
- `wait()` 方法必须在同步代码块或同步方法中被调用，这是因为**调用 `wait()` 方法的前提是当前线程必须持有对象的锁**。否则会抛出 `IllegalMonitorStateException` 异常。

3、唤醒方式不同

- 调用 sleep 方法后，线程会进入 **TIMED_WAITING** 状态（定时等待状态）
- 调用 wait 方法后，线程会进入 **WAITING** 状态（无限期等待状态）

4、抛出异常不同

- `sleep()` 方法在等待期间，如果**线程被中断**，会抛出 `InterruptedException`。
- 如果**线程被中断或等待时间到期**时，`wait()` 方法同样会在等待期间抛出 `InterruptedException`。



### 怎么保证线程安全？

- 为了保证线程安全，可以使用 **synchronized 关键字或 ReentrantLock** 来保证共享资源的互斥访问。
- 对于简单的变量操作，可以使用 **Atomic 类**来实现无锁线程安全。
- 可以使用线程安全容器，如 **ConcurrentHashMap 或 CopyOnWriteArrayList**。
- 对于每个线程独立的数据，可以使用 **ThreadLocal** 来为每个线程提供独立的变量副本。
- 对于简单的状态标志，可以使用 **volatile** 关键字确保多线程间的可见性。



#### 场景:有个int的变量为0，十个线程轮流对其进行++操作（循环10000次），结果是大于小于还是等于10万，为什么？

最终的结果会小于 100000，原因在于多线程环境下，++ 操作不是一个原子操作，会出现线程安全问题。

int++ 实际上可以分解为三步：

1. 读取变量的值。
2. 将读取到的值加 1。
3. 将结果写回变量。

多个线程在并发执行 ++ 操作时，可能出现以下竞态条件：

- 线程 1 读取变量值为 0。
- 线程 2 也读取变量值为 0。
- 线程 1 进行加法运算并将结果 1 写回变量。
- 线程 2 进行加法运算并将结果 1 写回变量，覆盖了线程 1 的结果。

可以通过 synchronized 或 AtomicInteger 实现线程安全。

#### 场景:有一个 key 对应的 value 是一个json 结构，json 当中有好几个子任务，这些子任务如果对 key 进行修改的话,会不会存在线程安全的问题?如何解决?如果是多个节点的情况，应该怎么加锁?

会。   在单节点环境中，可以使用 synchronized 关键字或 ReentrantLock 来保证对 key 的修改操作是原子的。

在多节点环境中，可以使用分布式锁 Redisson 来保证对 key 的修改操作是原子的。



#### 说一个线程安全的使用场景？

一个常见的使用场景是在实现**单例模式**时确保线程安全。

```java
class Singleton {
    // 使用 volatile 确保实例变量的可见性和防止指令重排序
    private volatile static Singleton singleton;
    // 私有化构造器，防止外部直接实例化
    private Singleton() {}

    // 提供全局访问点
    public static Singleton getInstance() {
        if (singleton == null) { // 第一次检查
            synchronized (Singleton.class) {
                if (singleton == null) { // 第二次检查
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```



#### 能说一下 Hashtable 数据结构底层吗？

与 HashMap 类似，Hashtable 的底层数据结构也是一个数组加上链表的方式，然后通过 **synchronized 加锁**来保证线程安全。





## ThreadLocal

### ThreadLocal 是什么？

[ThreadLocal](https://javabetter.cn/thread/ThreadLocal.html) 是 Java 中提供的一种用于实现线程局部变量的工具类。它允许**每个线程都拥有自己的独立副本**，从而**实现线程隔离**，用于解决多线程中共享对象的线程安全问题。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171756942.png" alt="三分恶面渣逆袭：ThreadLocal线程副本" style="zoom:67%;" />](https://camo.githubusercontent.com/f5369c93485610457e0fd056ee10b6a5f4d069789c066f031ed7ecf50d9e0d15/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d31312e706e67)

- 在 Web 应用中，可以使用 ThreadLocal 存储用户会话信息，这样每个线程在处理用户请求时都能方便地访问当前用户的会话信息。

- 在数据库操作中，可以使用 ThreadLocal 存储数据库连接对象，每个线程有自己独立的数据库连接，从而避免了多线程竞争同一数据库连接的问题。
- 在格式化操作中，例如日期格式化，可以使用 ThreadLocal 存储 SimpleDateFormat 实例，避免多线程共享同一实例导致的线程安全问题。



使用 ThreadLocal 通常分为四步：

①、创建 ThreadLocal

```java
//创建一个ThreadLocal变量
public static ThreadLocal<String> localVariable = new ThreadLocal<>();
```

②、设置 ThreadLocal 的值

```java
//设置ThreadLocal变量的值
localVariable.set("沉默王二是沙雕");
```

③、获取 ThreadLocal 的值

```java
//获取ThreadLocal变量的值
String value = localVariable.get();
```

④、删除 ThreadLocal 的值

```java
//删除ThreadLocal变量的值
localVariable.remove();
```



#### ThreadLocal 有哪些优点？

**①、线程隔离**：每个线程访问的变量副本都是独立的，避免了共享变量引起的线程安全问题。由于 ThreadLocal 实现了变量的线程独占，使得变量不需要同步处理，因此能够避免资源竞争。

**②、数据传递方便**：ThreadLocal 常用于在跨方法、跨类时**传递上下文数据**（如用户信息等），而不需要在方法间传递参数。



### 你在工作中用到过 ThreadLocal 吗？

有用到过，用来存储用户信息。

假如在服务层和持久层也要用到用户信息，就可以在控制层拦截请求把用户信息存入 ThreadLocal。

这样我们在任何一个地方，都可以取出 ThreadLocal 中存的用户信息。

数据库连接池也可以用 ThreadLocal，将数据库连接池的连接交给 ThreadLocal 进行管理，能够保证当前线程的操作都是同一个 Connnection。



### :star:ThreadLocal 怎么实现的呢？

ThreadLocal 本身并不存储任何值，它只是作为一个映射，来映射线程的局部变量。当一个线程调用 ThreadLocal 的 set 或 get 方法时，实际上是访问线程自己的 ThreadLocal.**ThreadLocalMap**。

ThreadLocalMap 是 ThreadLocal 的静态内部类，它内部维护了一个 **Entry 数组**，**key 是 ThreadLocal 对象，value 是线程的局部变量本身。Map 的大小由 ThreadLocal 对象的多少决定。**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171819054.png" alt="image-20241217181951975" style="zoom:67%;" />

#### ThreadLocal的演化：

早期的 ThreadLocal 不是这样的，它的 ThreadLocalMap 中使用 Thread 作为 key，这也是最简单的实现方式。

![image-20241217182031611](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171820695.png)

#### TheadLocal从Map优化成TreadLocalMap的好处

- 优化后的方案有两个好处，一个是 **Map 中存储的键值对变少了**；

  **为什么Map中储存的键值对变少了是好处？**

  **在普通的`Map`中，键值对的数量可能与线程的数量相关，而在`ThreadLocalMap`中，键值对的数量与`ThreadLocal`实例的数量相关。由于`ThreadLocal`的数量通常远少于线程的数量，因此`ThreadLocalMap`中的键值对数量更少，从而降低了哈希冲突的概率。**

- 另一个是 **ThreadLocalMap 的生命周期和线程一样长**，线程销毁的时候，ThreadLocalMap 也会被销毁。



Entry 继承了 WeakReference，它限定了 key 是一个**弱引用**，弱引用的好处是当内存不足时，JVM 会回收 ThreadLocal 对象，并且将其对应的 Entry 的 value 设置为 null，这样在很大程度上**可以避免内存泄漏**。



#### 什么是弱引用，什么是强引用？

强引用，比如说 `User user = new User("沉默王二")` 中，user 就是一个**强引用**，`new User("沉默王二")` 就是一个**强引用对象**。

**当 user 被置为 null 时**（`user = null`），`new User("沉默王二")` **将会被垃圾回收**；**如果 user 不被置为 null，即便是内存空间不足，JVM 也不会回收** `new User("沉默王二")` 这个强引用对象，宁愿抛出 OutOfMemoryError。

弱引用，比如说下面这段代码：

```
ThreadLocal<User> userThreadLocal = new ThreadLocal<>();
userThreadLocal.set(new User("沉默王二"));
```

①、userThreadLocal 是一个强引用，`new ThreadLocal<>()` 是一个强引用对象；

②、`new User("沉默王二")` 是一个强引用对象。

③、在 ThreadLocalMap 中，`key = new ThreadLocal<>()` 是一个弱引用对象。**当 JVM 进行垃圾回收时，如果发现了弱引用对象，就会将其回收。**

[![三分恶面渣逆袭：ThreadLocal内存分配](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412171825246.png)](https://camo.githubusercontent.com/0c4293303ad02dbd4deb5d0637cde49e4fe6972067b064cbe51677f8027e85ce/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d31342e706e67)

#### 

其关系链就是：

- ThreadLocal 强引用 -> ThreadLocal 对象。
- Thread 强引用 -> ThreadLocalMap。
- `ThreadLocalMap[i]` 强引用了 -> Entry。
- Entry.key 弱引用 -> ThreadLocal 对象。
- Entry.value 强引用 -> 线程的局部变量对象。

#### ThreadLocal 被谁引用？

:exclamation:**ThreadLocal被Entry的key弱引用**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172004573.png" alt="image-20241217200450485" style="zoom: 67%;" />



### ThreadLocal 内存泄露是怎么回事？

通常情况下，随着线程 Thread 的结束，其内部的 ThreadLocalMap 也会被回收，从而避免了内存泄漏。

但**如果一个线程一直在运行，并且其 `ThreadLocalMap` 中的 Entry.value 一直指向某个强引用对象，那么这个对象就不会被回收，从而导致内存泄漏。**当 Entry 非常多时，可能就会引发更严重的内存溢出问题。



#### 那怎么解决内存泄漏问题呢？

很简单，使用完 ThreadLocal 后，及时调用 `remove()` 方法释放内存空间。

`remove()` 方法会将当前线程的 ThreadLocalMap 中的所有 key 为 null 的 **Entry 全部清除**，这样就能避免内存泄漏问题。



#### 那为什么 key 要设计成弱引用？

弱引用的好处是，当内存不足的时候，JVM 会主动回收掉弱引用的对象。

一旦 key 被回收，ThreadLocalMap 在进行 set、get 的时候就会对 key 为 null 的 Entry 进行清理。

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172000809.png" alt="img" style="zoom: 67%;" />](https://camo.githubusercontent.com/eec8b88b4d7ffe9640ef31a56967448263f6c75b08d59c04dcf25ffac9d36e66/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a6176617468726561642d32303234303430373231343631362e706e67)

总结一下，在 ThreadLocal 被垃圾收集后，下一次访问 ThreadLocalMap 时，Java 会自动清理那些键为 null 的条目（参照源码中的 replaceStaleEntry 方法），这个过程会在执行 ThreadLocalMap 相关操作（如 `get()`, `set()`, `remove()`）时触发。



#### 你了解哪些 ThreadLocal 的改进方案？

在 JDK 20 Early-Access Build 28 版本中，出现了 ThreadLocal 的改进方案，即 `ScopedValue`。

还有 Netty 中的 FastThreadLocal，它是 Netty 对 ThreadLocal 的优化，它内部维护了一个索引常量 index，每次创建 FastThreadLocal 中都会自动+1，用来取代 hash 冲突带来的损耗，用空间换时间。



### ThreadLocalMap 的源码看过吗？

ThreadLocalMap 虽然被叫做 Map，其实它是没有实现 Map 接口的，但是结构还是和 HashMap 比较类似的，主要关注的是两个要素：**元素数组**和**散列方法**。

[![ThreadLocalMap结构示意图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172005673.png)](https://camo.githubusercontent.com/391010ad020ead39a60c4b4f26cd7f247e65f42417b1bae37d66f9b3342dfb06/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d31352e706e67)

- 元素数组

  一个 table 数组，存储 Entry 类型的元素，Entry 是 ThreaLocal 弱引用作为 key，Object 作为 value 的结构。

- 散列方法

  散列方法就是怎么把对应的 key 映射到 table 数组的相应下标，ThreadLocalMap 用的是哈希取余法，取出 key 的 threadLocalHashCode，然后和 table 数组长度减一&运算（相当于取余）。

threadLocalHashCode 计算有点东西，每创建一个 ThreadLocal 对象，它就会新增`0x61c88647`，这个值很特殊，它是**斐波那契数** 也叫 **黄金分割数**。`hash`增量为 这个数字，带来的好处就是 `hash` **分布非常均匀**。

**为了让哈希码能均匀的分布在2的N次方的数组里, 即** `Entry[] table`



### ThreadLocalMap 怎么解决 Hash 冲突的？

ThreadLocalMap 没有使用链表，用的是另外一种方式——**线性探测法**。

简单来说，就是这个坑被人占了，那就接着去找空着的坑。

[![ThreadLocalMap解决冲突](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172010427.png)](https://camo.githubusercontent.com/e22062214c1f121c16f62252a7f5bad89b797a1bc0d8e087cc1fa100e39ad51b/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d31362e706e67)

如上图所示，如果我们插入一个 value=27 的数据，通过 hash 计算后应该落入第 4 个槽位中，而槽位 4 已经有了 Entry 数据，而且 Entry 数据的 key 和当前不相等。此时就会线性向后查找，一直找到 Entry 为 null 的槽位才会停止查找，把元素放到空的槽中。

在 get 的时候，也会根据 ThreadLocal 对象的 hash 值，定位到 table 中的位置，然后判断该槽位 Entry 对象中的 key 是否和 get 的 key 一致，如果不一致，就判断下一个位置。



### ThreadLocalMap 扩容机制了解吗？

在 ThreadLocalMap.set()方法的最后，如果执行完启发式清理工作后，**未清理到任何数据**，且**当前散列数组中`Entry`的数量已经达到了列表的扩容阈值**`(len*2/3)`，就开始执行`rehash()`逻辑：

再着看 rehash()具体实现：这里会**先去清理过期的 Entry**，然后还要根据条件判断`size >= threshold - threshold / 4` 也就是**`size >= threshold* 3/4`来决定是否需要 resize() 扩容**。

接着看看具体的`resize()`方法：

1. 创建两倍大小的新数组
2. 老数组元素散列到新数组 （线性探测法）
3. table引用指向新数组

![ThreadLocalMap扩容](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172014621.png)



### 父子线程怎么共享数据？

父线程能用 ThreadLocal 来给子线程传值吗？毫无疑问，不能。那该怎么办？

这时候可以用到另外一个类——`InheritableThreadLocal `。

使用起来很简单，在主线程的 InheritableThreadLocal 实例设置值，在子线程中就可以拿到了。

```java
public class InheritableThreadLocalTest {
    public static void main(String[] args) {
        final ThreadLocal threadLocal = new InheritableThreadLocal();
        // 主线程
        threadLocal.set("不擅技术");
        //子线程
        Thread t = new Thread() {
            @Override
            public void run() {
                super.run();
                System.out.println("鄙人三某 ，" + threadLocal.get());
            }
        };
        t.start();
    }
}
```

#### InheritableThreadLocal 原理

原理很简单，在 Thread 类里还有另外一个变量：

```java
ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
```

在 Thread.init 的时候，如果父线程的`inheritableThreadLocals`不为空，就把它赋给当前线程（子线程）的`inheritableThreadLocals `。



## Java 内存模型 JMM

### 说一下你对 Java 内存模型的理解？

Java 内存模型（Java Memory Model）是一种抽象的模型，简称 **JMM**，主要用**来定义多线程中变量的访问规则，用来解决变量的可见性、有序性和原子性问题，确保在并发环境中安全地访问共享变量。**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172024267.png" alt="image-20241217202406181" style="zoom: 67%;" />

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172024997.png" alt="image-20241217202418914" style="zoom:67%;" />

对于一个双核 CPU 的系统架构，每个核都有自己的控制器和运算器，其中控制器包含一组寄存器和操作控制器，运算器执行算术逻辅运算。

每个核都有**自己的一级缓存**，在有些架构里面还有一个所有 CPU **共享的二级缓存**。

Java 内存模型里面的本地内存，可能对应的是 L1 缓存或者 L2 缓存或者 CPU 寄存器。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172020521.png" alt="三分恶面渣逆袭：实际线程工作模型" style="zoom:67%;" />

#### 为什么线程要用自己的内存？

第一，在多线程环境中，如果所有线程都直接操作主内存中的共享变量，会引发更多的**内存访问竞争**，这不仅影响性能，还增加了线程安全问题的复杂度。通过让每个线程使用本地内存，可以减少对主内存的直接访问和竞争，从而提高程序的并发性能。

第二，现代 CPU 为了优化执行效率，可能会对**指令进行乱序执行**（指令重排序）。使用本地内存（CPU 缓存和寄存器）可以在不影响最终执行结果的前提下，使得 CPU 有更大的自由度来乱序执行指令，从而提高执行效率。



#### JMM中 栈、方法区、堆的主要区别

| **内存区域** | **存放内容**                             | **线程共享性** | **生命周期**                       |
| ------------ | ---------------------------------------- | -------------- | ---------------------------------- |
| 栈           | 方法调用栈帧、局部变量、操作数栈         | 线程私有       | 随线程开始和结束                   |
| 方法区       | 类的元数据、静态变量、常量池、方法字节码 | 线程共享       | JVM 启动到退出                     |
| 堆           | 对象实例、数组                           | 线程共享       | JVM 启动到退出，垃圾回收器动态管理 |



### 说说你对原子性、可见性、有序性的理解？

- **原子性**：指的是一个操作是不可分割的，要么全部执行成功，要么完全不执行。
- **可见性**：指的是**一个线程对共享变量的修改，能够被其他线程及时看见**。
- **有序性**：指的是程序代码的执行顺序与代码中的顺序一致。在没有同步机制的情况下，编译器可能会对指令进行**重排序**，以优化性能。这种重排序可能会导致多线程的执行结果与预期不符。



#### 原子性、可见性、有序性都应该怎么保证呢？

- 原子性：JMM 只能保证基本的原子性，如果要保证一个代码块的原子性，需要使用`synchronized `。
- 可见性：Java 是利用`volatile`关键字来保证可见性的，除此之外，`final`和`synchronized`也能保证可见性。
- 有序性：`synchronized`或者`volatile`都可以保证多线程之间操作的有序性。



### 那说说什么是指令重排？

在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序。重排序分 3 种类型。

1. **编译器优化**的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
2. **指令级并行**的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应 机器指令的执行顺序。
3. **内存系统**的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从 Java 源代码到最终实际执行的指令序列，会分别经历下面 3 种重排序，如图：

![image-20241217205632086](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172056176.png)

JMM 属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。



### 指令重排有限制吗？happens-before 了解吗？

指令重排也是有一些限制的，有两个规则`happens-before`和`as-if-serial`来约束。

- as-if-serial 规则 : 无论如何重排，程序的执行结果在单线程环境下不能改变。

- happens-before规则 : 在多线程环境中，Java 内存模型通过 `happens-before` 规则定义了操作间的执行顺序关系，限制指令重排。



happens-before 的定义：

- 如果一个操作 happens-before 另一个操作，那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。
- 两个操作之间存在 happens-before 关系，并不意味着 Java 平台的具体实现必须要按照 happens-before 关系指定的顺序来执行。如果重排序之后的执行结果，与按 happens-before 关系来执行的结果一致，那么这种重排序并不非法

happens-before 和我们息息相关的有六大规则：

[![happens-before六大规则](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172058478.png)](https://camo.githubusercontent.com/75b9257519c309a4e7b652b844bb5674fee9d928011c65ac6655d3bc56b2c101/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d32332e706e67)

- **程序顺序规则**：一个线程中的每个操作，happens-before 于该线程中的任意后续操作。
- **监视器锁规则**：对一个锁的解锁，happens-before 于随后对这个锁的加锁。
- **volatile 变量规则**：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。
- **传递性**：如果 A happens-before B，且 B happens-before C，那么 A happens-before C。
- **start()规则**：如果线程 A 执行操作 ThreadB.start()（启动线程 B），那么 A 线程的 ThreadB.start()操作 happens-before 于线程 B 中的任意操作。
- **join()规则**：如果线程 A 执行操作 ThreadB.join()并成功返回，那么线程 B 中的任意操作 happens-before 于线程 A 从 ThreadB.join()操作成功返回。



### as-if-serial 又是什么？单线程的程序一定是顺序的吗？

as-if-serial 语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），**单线程程序的执行结果不能被改变**。编译器、runtime 和处理器都必须遵守 as-if-serial 语义。

为了遵守 as-if-serial 语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。但是，如果**操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序**。



### :star:volatile 了解吗？

volatile 关键字主要有两个作用，一个是**保证变量的内存可见性**，一个是**禁止指令重排序**。它确保一个线程对变量的修改对其他线程立即可见，同时防止代码执行顺序被编译器或 CPU 优化重排。

#### volatile 怎么保证可见性的呢？

当一个变量被声明为 volatile 时，Java 内存模型会确保所有线程看到该变量时的值是一致的。

当线程对 volatile 变量进行**写操作**时，JMM 会在写入这个变量之后插入一个**写屏障指令**，这个指令会**强制将本地内存中的变量值刷新到主内存中。**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172126969.png" alt="image-20241217212654866" style="zoom: 67%;" />

在 x86 架构下，volatile 写操作会插入一个 lock 前缀指令，这个指令会将缓存行的数据写回到主内存中，确保内存可见性。

```asm
mov [a], 2          ; 将值 2 写入内存地址 a
lock add [a], 0     ; lock 指令充当写屏障，确保内存可见性
```



当线程对 volatile 变量进行**读操作**时，JMM 会插入一个 **读屏障指令**，这个指令会**强制让本地内存中的变量值失效，从而重新从主内存中读取最新的值。**

[<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172128366.png" alt="三分恶面渣逆袭：volatile写插入内存屏障后生成的指令序列示意图" style="zoom:80%;" />](https://camo.githubusercontent.com/432b9d93cfdd019c1e112be9d9e01283b6a6822745350eb25773d6b40f876beb/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d32392e706e67)



例如，我们声明一个 volatile 变量 x：

```java
volatile int x = 0
```

线程 A 对 x 写入后会将其最新的值刷新到主内存中，线程 B 读取 x 时由于本地内存中的 x 失效了，就会从主内存中读取最新的值，内存可见性达成！

[![三分恶面渣逆袭：volatile内存可见性](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172130811.png)](https://camo.githubusercontent.com/6078970176aeb673c5ef4638acb1a1174b80f91d3be73f138fc94c2d7b99d801/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d32362e706e67)



#### volatile 怎么保证有序性的呢？

在程序执行期间，为了提高性能，编译器和处理器会对指令进行重排序。但涉及到 volatile 变量时，它们必须遵循一定的规则：

- **写 volatile 变量的操作之前的操作不会被编译器重排序到写操作之后。**
- **读 volatile 变量的操作之后的操作不会被编译器重排序到读操作之前。**

这意味着 volatile 变量的写操作总是发生在任何后续读操作之前。



#### volatile 和 synchronized 的区别

volatile 关键字用于**修饰变量**，确保该变量的**更新操作对所有线程是可见的**，即一旦某个线程修改了 volatile 变量，其他线程会立即看到最新的值。

synchronized 关键字用于**修饰方法或代码块**，确保同一时刻**只有一个线程能够执行该方法或代码块**，从而实现互斥访问。



#### volatile 加在基本类型和对象上的区别？

- 当 `volatile` 用于**基本数据类型**时，能确**保该变量的读写操作是直接从主内存中读取或写入的**。

  ```java
  private volatile int count = 0;
  ```

- 当 `volatile` 用于**引用类型**时，它确保**引用本身的可见性**，即确保引用指向的对象地址是最新的。

但是，**`volatile` 并不能保证引用对象内部状态的线程安全性。**

```java
private volatile SomeObject obj = new SomeObject();
```

虽然 `volatile` 确保了 `obj` 引用的可见性，但对 `obj` 引用的具体对象的操作并不受 `volatile` 保护。如果需要保证引用对象内部状态的线程安全，需要使用其他同步机制（如 `synchronized` 或 `ReentrantLock`）。







## 锁

### synchronized 用过吗？怎么使用？

在 Java 中，synchronized 是最常用的锁，它使用简单，并且可以保证线程安全，避免多线程并发访问时出现数据不一致的情况。

synchronized 可以用在方法和代码块中。

①、修饰方法

```java
public synchronized void increment() {
    this.count++;
}
```

当在方法声明中使用了 synchronized 关键字，就表示**该方法是同步**的，也就是说，线程在执行这个方法的时候，**其他线程不能同时执行，需要等待锁释放。**

**如果是静态方法的话，锁的是这个类的 Class 对象，因为静态方法是属于类级别的。**

```java
public static synchronized void increment() {
    count++;
}
```

②、修饰代码块

```java
public void increment() {
    synchronized (this) {
        this.count++;
    }
}
```

同步代码块可以减少需要同步的代码量，颗粒度更低，更灵活。synchronized 后面的括号中指定了要锁定的对象，可以是 this，也可以是其他对象。





### synchronized 的实现原理？

#### synchronized 是怎么加锁的呢？

synchronized 是 JVM 帮我们实现的，因此在使用的时候不用手动去 lock 和 unlock，JVM 会帮我们自动加锁和解锁。

①、synchronized 修饰代码块时，JVM 会通过 `monitorenter`、`monitorexit` 两个汇编指令来实现同步：

- `monitorenter` 指向同步代码块的开始位置
- `monitorexit` 指向同步代码块的结束位置。

②、synchronized 修饰方法时，JVM 会通过 `ACC_SYNCHRONIZED` 汇编标记符来实现同步。



#### synchronized 锁住的是什么呢？

monitorenter、monitorexit 或者 ACC_SYNCHRONIZED 都是**基于 Monitor 实现**的。

实例对象结构里有对象头，对象头里面有一块结构叫 Mark Word，Mark Word 指针指向了**monitor**。

所谓的 Monitor 其实是一种**同步工具**，也可以说是一种**同步机制**。在 Java 虚拟机（HotSpot）中，Monitor 是由**ObjectMonitor 实现**的，可以叫做内部锁，或者 Monitor 锁。

ObjectMonitor 的工作原理：

- ObjectMonitor 有两个队列：_WaitSet、_EntryList，用来保存 ObjectWaiter 对象列表。
- _owner，获取 Monitor 对象的线程进入 _owner 区时， _count + 1。如果线程调用了 wait() 方法，此时会释放 Monitor 对象， _owner 恢复为空， _count - 1。同时该等待线程进入 _WaitSet 中，等待被唤醒。

![image-20241217214524917](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172145053.png)

- **门诊大厅**：所有待进入的线程都必须先在**入口 Entry Set**挂号才有资格；
- **就诊室**：就诊室**_Owner**里里只能有一个线程就诊，就诊完线程就自行离开
- **候诊室**：就诊室繁忙时，进入**等待区（Wait Set）**，就诊室空闲的时候就从**等待区（Wait Set）**叫新的线程

![image-20241217214539923](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172145052.png)

所以我们就知道了，**同步是锁住的什么东西**：

- monitorenter，在判断拥有同步标识 ACC_SYNCHRONIZED 抢先进入此方法的线程会优先拥有 Monitor 的 owner ，此时计数器 +1。
- monitorexit，当执行完退出后，计数器 -1，归 0 后被其他进入的线程获得。



#### 会不会牵扯到 os 层面呢？

会，synchronized 升级为重量级锁时，依赖于**操作系统的互斥量**（mutex）来实现，mutex 用于保证任何给定时间内，只有一个线程可以执行某一段特定的代码段。





### 除了原子性，synchronized 可见性，有序性，可重入性怎么实现？

#### synchronized 怎么保证可见性？

- 线程加锁前，将**清空工作内存中共享变量的值**，从而使用共享变量时需要从主内存中重新读取最新的值。
- 线程加锁后，**其它线程无法获取主内存中的共享变量**。
- 线程解锁前，**必须把共享变量的最新值刷新到主内存中**。

#### synchronized 怎么保证有序性？

synchronized 同步的代码块，具有排他性，一次只能被一个线程拥有，所以 synchronized 保证同一时刻，代码是**单线程执行**的。

因为 **as-if-serial 语义**的存在，单线程的程序能保证最终结果是有序的，但是不保证不会指令重排。

所以 synchronized 保证的有序是执行结果的有序性，而不是防止指令重排的有序性。

#### synchronized 怎么实现可重入的呢？

可重入意味着**同一个线程可以多次获得同一个锁，而不会被阻塞**。具体来说，如果一个线程已经持有某个锁，那么它可以再次进入该锁保护的代码块或方法，而不会被阻塞。

synchronized 之所以支持可重入，是因为 **Java 的对象头包含了一个 Mark Word，用于存储对象的状态，包括锁信息。**

当一个线程获取对象锁时，JVM 会将该线程的 ID 写入 Mark Word，并将锁计数器设为 1。

如果一个线程尝试再次获取已经持有的锁，JVM 会**检查 Mark Word 中的线程 ID**。如果 ID 匹配，表示的是同一个线程，锁计数器递增。

当线程退出同步块时，锁计数器递减。如果计数器值为零，JVM 将锁标记为未持有状态，并清除线程 ID 信息。



### :star:锁升级？synchronized 优化了解吗？

#### 什么是锁升级？

锁升级是 Java 虚拟机中的一个优化机制，用于提高多线程环境下 synchronized 的并发性能。锁升级涉及从较轻的锁状态（如无锁或偏向锁）逐步升级到较重的锁状态（如轻量级锁和重量级锁），以适应不同程度的竞争情况。

Java 对象头里的 `Mark Word` 会记录锁的状态，一共有四种状态：

①、**无锁状态**，在这个状态下，没有线程试图获取锁。

②、**偏向锁**，当第一个线程访问同步块时，锁会进入偏向模式。Mark Word 会被设置为偏向模式，并且存储了获取它的线程 ID。

偏向锁的目的是消除同一线程的后续锁获取和释放的开销。如果同一线程再次请求锁，就无需再次同步。

③、当有**多个线程竞争锁，但没有锁竞争的强烈迹象**（即线程交替执行同步块）时，偏向锁会升级为**轻量级锁。**

线程尝试通过**CAS 操作**（Compare-And-Swap）将对象头的 Mark Word 替换为指向锁记录的指针。如果成功，当前线程获取轻量级锁；如果失败，说明有竞争。

④、**重量级锁**，当**锁竞争激烈时**，轻量级锁会膨胀为重量级锁。

重量级锁通过将对象头的 Mark Word 指向监视器（Monitor）对象来实现，该对象包含了锁的持有者、锁的等待队列等信息。

[![三分恶面渣逆袭：Mark Word变化](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172156262.png)](https://camo.githubusercontent.com/cdda2caabf41c994b7c144506df44a5b3b5167abe079f0214cf67b020842afd1/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d33342e706e67)

#### synchronized 做了哪些优化？

在 JDK1.6 之前，synchronized 是**直接调用 ObjectMonitor 的 enter 和 exit 实现**的，这种锁也被称为**重量级锁**。这也是为什么很多声音说不要用 synchronized 的原因，有点“谈虎色变”的感觉。

从 JDK 1.6 开始，HotSpot 对 Java 中的锁进行优化，如增加了适应性自旋、锁消除、锁粗化、轻量级锁和偏向锁等优化策略，极大提升了 synchronized 的性能。

**①、偏向锁**：当一个线程首次获得锁时，JVM 会将锁标记为偏向这个线程，将锁的标志位设置为偏向模式，并且在对象头中记录下该线程的 ID。

之后，当相同的线程再次请求这个锁时，就无需进行额外的同步。如果另一个线程尝试获取这个锁，偏向模式会被撤销，并且锁会升级为轻量级锁。

**②、轻量级锁**：多个线程在不同时段获取同一把锁，即不存在锁竞争的情况，也就没有线程阻塞。针对这种情况，JVM 采用轻量级锁来避免线程的阻塞与唤醒。

当一个线程尝试获取轻量级锁时，它会在自己的栈帧中创建一个锁记录（Lock Record），然后尝试使用 CAS 操作将对象头的 Mark Word 替换为指向锁记录的指针。

如果成功，该线程持有锁；如果失败，表示有其他线程竞争，锁会升级为重量级锁。

**③、自旋锁**：当线程尝试获取轻量级锁失败时，它会进行自旋，即循环检查锁是否可用，以避免立即进入阻塞状态。

自旋的次数不是固定的，而是根据之前在同一个锁上的自旋时间和锁的状态动态调整的。

**④、锁粗化**：如果 JVM 检测到一系列连续的锁操作实际上是在单一线程中完成的，则会将多个锁操作合并为一个更大范围的锁操作，这可以减少锁请求的次数。

锁粗化主要针对循环内连续加锁解锁的情况进行优化。

**⑤、锁消除**：JVM 的即时编译器（[JIT](https://javabetter.cn/jvm/jit.html)）可以在运行时进行代码分析，如果发现某些锁操作不可能被多个线程同时访问，那么这些锁操作就会被完全消除。锁消除可以减少不必要的同步开销。

#### 锁升级的过程是什么样的？

[![三分恶面渣逆袭：锁升级简略过程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172205229.png)](https://camo.githubusercontent.com/92d3eee4f4162eb448692ba47d3dc2e8d2ebb5917df876fe8708914dc6dd2fb8/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d33362e706e67)

完整的升级过程：

[![三分恶面渣逆袭：synchronized 锁升级过程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172205768.png)](https://camo.githubusercontent.com/48b799256efe21fa2e25f99a464da65bdf3fc93f17fb5412f9c185e9767a3179/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d33372e706e67)

**①、从无锁到偏向锁：**

当一个线程首次访问同步块时，如果此对象无锁状态且偏向锁未被禁用，JVM 会将该对象头的锁标记改为偏向锁状态，并记录下当前线程的 ID。此时，对象头中的 Mark Word 中存储了持有偏向锁的线程 ID。

如果另一个线程尝试获取这个已被偏向的锁，JVM 会检查当前持有偏向锁的线程是否活跃。如果持有偏向锁的线程不活跃，则可以将锁重偏向至新的线程；如果持有偏向锁的线程还活跃，则需要撤销偏向锁，升级为轻量级锁。

**②、偏向锁到轻量级锁：**

进行偏向锁撤销时，会**遍历堆栈的所有锁记录，暂停拥有偏向锁的线程，并检查锁对象**。如果这个过程中发现有其他线程试图获取这个锁，JVM 会撤销偏向锁，并将锁升级为轻量级锁。

当有两个或以上线程竞争同一个偏向锁时，偏向锁模式不再有效，此时偏向锁会被撤销，对象的锁状态会升级为轻量级锁。

**③、轻量级锁到重量级锁：**

**轻量级锁通过线程自旋来等待锁释放**。如果**自旋超过预定次数**（自旋次数是可调的，并且自适应的），表明锁竞争激烈，轻量级锁的自旋已经不再高效。

当自旋等待失败，或者有线程在等待队列中等待相同的轻量级锁时，轻量级锁会升级为重量级锁。在这种情况下，JVM 会在**操作系统层面创建一个互斥锁（Mutex）**，所有进一步尝试获取该锁的线程将会被阻塞，直到锁被释放。



### :star:synchronized 和 ReentrantLock 的区别？

[synchronized](https://javabetter.cn/thread/synchronized-1.html) 是一个关键字，[ReentrantLock](https://javabetter.cn/thread/reentrantLock.html)是 Lock 接口的一个实现。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172212073.png" alt="image-20241217221224907" style="zoom: 67%;" />

它们都可以用来实现同步，但也有一些区别：

- ReentrantLock 可以实现**多路选择通知**（绑定多个 [Condition](https://javabetter.cn/thread/condition.html)），而 synchronized 只能通过 **wait 和 notify/notifyAll** 方法唤醒一个线程或者唤醒全部线程（单路通知）；

- **ReentrantLock 必须手动释放锁**。通常需要在 finally 块中调用 unlock 方法以确保锁被正确释放；synchronized 会自动释放锁，当同步块执行完毕时，由 JVM 自动释放，不需要手动操作。

- ReentrantLock 通常能提供更好的性能，因为它可以更**细粒度**控制锁；synchronized 只能同步代码快或者方法，随着 JDK 版本的升级，两者之间性能差距已经不大了。

  

#### 使用方式有什么不同？

```java
// synchronized 修饰方法
public synchronized void method() {
    // 业务代码
}
// synchronized 修饰代码块
synchronized (this) {
    // 业务代码
}

// ReentrantLock 加锁
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // 业务代码
} finally {
    lock.unlock();
}
```



#### 功能特点有什么不同？

如果需要更细粒度的控制（如可中断的锁操作、尝试非阻塞获取锁、超时获取锁或者使用公平锁等），可以使用 Lock。

- ReentrantLock 提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()`来实现这个机制。
- ReentrantLock 可以指定是公平锁还是非公平锁。
- ReentrantReadWriteLock 读写锁，读锁是共享锁，写锁是独占锁，读锁可以同时被多个线程持有，写锁只能被一个线程持有。这种锁的设计可以提高性能，特别是在读操作的数量远远超过写操作的情况下。

Lock 还提供了`newCondition()`方法来创建等待通知条件[Condition](https://javabetter.cn/thread/condition.html)，比 synchronized 与 `wait()`、 `notify()/notifyAll()`方法的组合更强大。

```java
ReentrantLock lock = new ReentrantLock();
Condition condition = lock.newCondition();
```



#### 并发量大的情况下，使用 synchronized 还是 ReentrantLock？

在并发量特别高的情况下，ReentrantLock 的性能可能会优于 synchronized，原因包括：

- ReentrantLock 提供了**超时和公平锁**等特性，可以更好地应对复杂的并发场景 。
- ReentrantLock 允许更**细粒度**的锁控制，可以有效减少锁竞争。
- ReentrantLock 支持条件变量 **Condition**，可以实现比 synchronized 更复杂的线程间通信机制。



#### Lock 了解吗？

Lock 是 Java `java.util.concurrent.locks` 包中的一个接口，**最常用的实现类是 ReentrantLock**，提供了比 synchronized 更多的功能，如可中断的锁操作、尝试非阻塞获取锁、超时获取锁或者使用公平锁等。

#### Lock.lock()的具体实现逻辑了解吗？

lock 方法用来获取锁，**如果锁已经被其他线程获取，当前线程会一直等待直到获取锁。**

具体实现由内部的 Sync 类来实现，ReentrantLock 有两种 Sync 实现：**NonfairSync 和 FairSync，分别对应非公平锁和公平锁。**

- **非公平锁**尝试让当前线程**直接通过 CAS 操作获取锁，如果获取失败则进入 AQS 队列等待**。
- **公平锁**会直接让线程进入**等待队列**，按顺序获取锁，**不允许插队**。

acquire 方法由 AQS（AbstractQueuedSynchronizer）提供，是整个锁机制的核心，管理着获取和释放锁的流程、队列等待、线程调度等复杂操作。







### :star:AQS 了解多少？

如果面试官问你 **“了解 AQS 吗？”**，你可以这样回答：

1. **AQS（`AbstractQueuedSynchronizer`）是 Java 并发工具的基础框架**，用于实现**锁（Lock）、信号量（Semaphore）、倒计时器（CountDownLatch）等同步器**。
2. AQS 采用 **`volatile int state` 作为同步变量**，通过 **CAS + 自旋 + 等待队列（CLH 队列）** 处理并发控制。
3. AQS 提供 **独占模式（如 `ReentrantLock`）和共享模式（如 `Semaphore`、`CountDownLatch`）**。
4. AQS 主要通过 **`LockSupport.park()` 挂起线程，`unpark()` 唤醒线程**，提高并发效率。
5. **实际应用**：
   - `ReentrantLock`：独占锁，基于 AQS 实现可重入锁。
   - `CountDownLatch`：倒计时器，`state == 0` 时唤醒所有等待线程。
   - `Semaphore`：信号量，允许多个线程同时访问资源。

💡 **加分项**：如果面试官深入问，可以提 AQS 的**公平锁/非公平锁**实现，或 AQS 在 `ReentrantReadWriteLock` 里的应用。





AQS，也就是**抽象队列同步器**，由 Doug Lea 设计，是 Java 并发包`java.util.concurrent`的核心框架类，许多同步类的实现都依赖于它，如 ReentrantLock、Semaphore、CountDownLatch 等。

AQS 的思想是，**如果被请求的共享资源空闲，则当前线程能够成功获取资源；否则，它将进入一个等待队列，当有其他线程释放资源时，系统会挑选等待队列中的一个线程，赋予其资源。**

整个过程通过维护一个 int 类型的状态和一个先进先出（FIFO）的**队列**，来实现对共享资源的管理。

[![三分恶面渣逆袭：AQS抽象队列同步器](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172240553.png)](https://camo.githubusercontent.com/2e6d753bf4a96649cae380293d7bf969074c47f2998086829a73da4106ff72aa/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d33392e706e67)

①、同步状态 state 由 volatile 修饰，保证了多线程之间的可见性；

```java
private volatile int state;
```

②、同步队列是通过内部定义的 Node 类来实现的，每个 Node 包含了等待状态、前后节点、线程的引用等。

```java
static final class Node {
    static final int CANCELLED =  1;
    static final int SIGNAL    = -1;
    static final int CONDITION = -2;
    static final int PROPAGATE = -3;

    volatile Node prev;

    volatile Node next;

    volatile Thread thread;
}
```



AQS 支持两种同步方式：

- 独占模式：这种方式下，每次只能有一个线程持有锁，例如 ReentrantLock。
- 共享模式：这种方式下，多个线程可以同时获取锁，例如 Semaphore 和 CountDownLatch。

子类可以通过继承 AQS 并实现它的方法来管理同步状态，这些方法包括：

- `tryAcquire`：独占方式尝试获取资源，成功则返回 true，失败则返回 false；
- `tryRelease`：独占方式尝试释放资源；
- `tryAcquireShared(int arg)`：共享方式尝试获取资源；
- `tryReleaseShared(int arg)`：共享方式尝试释放资源；
- `isHeldExclusively()`：该线程是否正在独占资源。

如果共享资源被占用，需要一种特定的阻塞等待唤醒机制来保证锁的分配，AQS 会将竞争共享资源失败的线程添加到一个 CLH 队列中。

[![三分恶面渣逆袭：CLH队列](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172242164.png)](https://camo.githubusercontent.com/d8cbfd18b7c869758b2a63326a33f0481afc8b8ee079c13a1b34eab06a283411/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d34302e706e67)

在 CLH 锁中，当一个线程尝试获取锁并失败时，它会将自己添加到队列的尾部并自旋，等待前一个节点的线程释放锁。

[![三分恶面渣逆袭：AQS变种CLH队列](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172242667.png)](https://camo.githubusercontent.com/49b0f402558626aeef2c59eb2b35f36292ece36eae4907c5f306faa72fbe5920/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d34312e706e67)





### ReentrantLock 实现原理？

Lock 接口提供了比 synchronized 关键字更灵活的锁操作。[ReentrantLock](https://javabetter.cn/thread/reentrantLock.html) 就是 Lock 接口的一个实现，它提供了与 synchronized 关键字类似的锁功能，但更加灵活。

ReentrantLock 是**可重入的独占锁**，只能有一个线程获取该锁，其它想获取该锁的线程会被阻塞。

`new ReentrantLock()` 默认创建的是**非公平锁** NonfairSync。在非公平锁模式下，锁可能会**授予刚刚请求它的线程，而不考虑等待时间。**

ReentrantLock 也支持公平锁，该模式下，锁会授予等待时间最长的线程。

ReentrantLock 内部通过一个计数器来跟踪锁的持有次数，线程首次获取锁时，计数器值变为 1；如果同一线程再次获取锁，计数器增加；每释放一次锁，计数器减 1。

![三分恶面渣逆袭：ReentrantLock 非公平锁加锁流程简图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172247790.png)



### ReentrantLock 怎么实现公平锁的？

ReentrantLock 的默认构造方法创建的是非公平锁 NonfairSync。可以通过有参构造方法传递 true 参数来创建公平锁 FairSync。

```java
ReentrantLock lock = new ReentrantLock(true);
// true 代表公平锁，false 代表非公平锁
```

FairSync、NonfairSync 都是 ReentrantLock 的内部类，分别实现了公平锁和非公平锁的逻辑。



#### 非公平锁和公平锁有什么不同？

①、公平锁意味着在多个线程竞争锁时，获取锁的顺序与线程请求锁的顺序相同，即**先来先服务**（FIFO）。

虽然能保证锁的顺序，但实现起来比较复杂，因为需要额外维护一个有序队列。

②、非公平锁不保证线程获取锁的顺序，当锁被释放时，任何请求锁的线程都有机会获取锁，而**不是按照请求的顺序。**





### CAS 了解多少？

CAS 是一种乐观锁的实现方式，全称为“比较并交换”（Compare-and-Swap），是一种无锁的原子操作。

CAS 是乐观锁，线程执行的时候不会加锁，它会假设此时没有冲突，然后完成某项操作；如果因为冲突失败了就重试，直到成功为止。

![image-20241217225235450](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172252627.png)

在 CAS 中，有这样三个值：

- V：要更新的变量(var)
- E：预期值(expected)
- N：新值(new)

比较并交换的过程如下：

判断 V 是否等于 E，如果等于，将 V 的值设置为 N；如果不等，说明已经有其它线程更新了 V，于是当前线程放弃更新，什么都不做。

这里的预期值 E 本质上指的是“旧值”。

这个**比较和替换的操作是原子的**，即不可中断，确保了数据的一致性。



可以借助 **AtomicInteger 类的 compareAndSet 方法**来实现。

compareAndSet 就是一个 CAS 方法，它调用的是 Unsafe 的 compareAndSwapInt。

为了保证CAS的原子性，CPU 提供了两种实现方式：

①、总线锁定，通过锁定 CPU 的总线，禁止其他 CPU 或设备访问内存。在进行操作时，CPU 发出一个 LOCK 信号，这会阻止其他处理器对内存地址进行操作，直到当前指令执行完成。

[![总线锁定：博客园的紫薇哥哥](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172254581.png)](https://camo.githubusercontent.com/4d47cf244326bfc0e01df467046d94aaba06c1ffbf353c0ef7491b997312ea3f/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a6176617468726561642d32303234313131353136313330352e706e67)

②、缓存锁定，当多个 CPU 操作同一块内存地址时，如果该内存地址已经被缓存到某个 CPU 的缓存中，缓存锁定机制会锁定该缓存行，防止其他 CPU 对这块内存进行修改。

现代CPU基本都支持和使用缓存锁定机制。



### CAS 有什么问题？如何解决？

CAS 存在三个经典问题。

[![三分恶面渣逆袭：CAS三大问题](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172301552.png)](https://camo.githubusercontent.com/5e2de5d0616a8ec0a01a66197bfff6654822a9ab6dcb983e2737eb6070543514/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d34342e706e67)



#### 什么是 ABA 问题？如何解决？

如果一个位置的值原来是 A，后来被改为 B，再后来又被改回 A，那么进行 CAS 操作的线程将无法知晓该位置的值在此期间已经被修改过。

可以使用 **版本号 **  /   **时间戳** 的方式来解决 ABA 问题。

Java 的 AtomicStampedReference 类就实现了版本号机制，它会同时检查引用值和 stamp 是否都相等。



#### 循环性能开销

自旋 CAS，如果一直循环执行，一直不成功，会给 CPU 带来非常大的执行开销。

在 Java 中，很多使用自旋 CAS 的地方，会有一个**自旋次数的限制**，超过一定次数，就停止自旋。



#### 只能保证一个变量的原子操作

CAS 保证的是对一个变量执行操作的原子性，如果对多个变量操作时，CAS 目前无法直接保证操作的原子性的。

- 可以考虑改用锁来保证操作的原子性
- 可以考虑合并多个变量，将多个变量封装成一个对象，通过 AtomicReference 来保证原子性。

**i = 2 , j = a 结合成 ij = 2a**



### Java 有哪些保证原子性的方法？如何保证多线程下 i++ 结果正确？

[![Java保证原子性方法](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172303574.png)](https://camo.githubusercontent.com/168721d0170f2ecc7e4d2bd82a95364e9e4641bd5572e3812a6f1f774814257b/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d34352e706e67)



### 原子操作类了解多少？

Atomic 包里一共提供了 13 个类，属于 4 种类型的原子更新方式，分别是原子更新基本类型、原子更新数组、原子更新引用和原子更新属性（字段）。

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172313300.png" alt="image-20241217231335149" style="zoom:67%;" />



### AtomicInteger 的原理？

一句话概括：**使用 CAS 实现**。

以 AtomicInteger 的添加方法为例：

```java
    public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
    }
```

通过`Unsafe`类的实例来进行添加操作，来看看具体的 CAS 操作：

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```

compareAndSwapInt 是一个 native 方法，基于 CAS 来操作 int 类型变量。其它的原子操作类基本都是大同小异。



### :star:线程死锁了解吗？该如何避免？

**互斥、占有且等待、不可剥夺、循环等待**

[![三分恶面渣逆袭：死锁产生必备四条件](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412172321456.png)](https://camo.githubusercontent.com/5cda66a0422e10e0fca3f94dc2cf3fa547f4bc1ace159c46227218cca2c25211/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d34382e706e67)

- **互斥条件**：资源不能被多个线程共享，一次只能由一个线程使用。如果一个线程已经占用了一个资源，其他请求该资源的线程必须等待，直到资源被释放。
- **持有并等待条件**：一个线程至少已经持有至少一个资源，且正在等待获取额外的资源，这些额外的资源被其他线程占有。
- **不可剥夺条件**：资源不能被强制从一个线程中抢占过来，只能由持有资源的线程主动释放。
- **循环等待条件**：存在一种线程资源的循环链，每个线程至少持有一个其他线程所需要的资源，然后又等待下一个线程所占有的资源。这形成了一个循环等待的环路。

#### 该如何避免死锁呢？

理解产生死锁的这四个必要条件后，就可以采取相应的措施来避免死锁，换句话说，就是**至少破坏死锁发生的一个条件**。

- **破坏互斥条件**：这通常**不可行**，因为加锁就是为了互斥。
- **破坏持有并等待条件**：一种方法是要求线程在开始执行前一次性地申请所有需要的资源。
- **破坏非抢占条件**：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以主动释放它占有的资源。
- **破坏循环等待条件**：对所有资源类型进行排序，强制每个线程按顺序申请资源，这样可以避免循环等待的发生。



### 那死锁问题怎么排查呢？

- Linux 生产环境中，可以先使用 **top ps** 等命令查看进程状态，看看是否有进程占用了过多的资源。

- 使用 JDK 自带的一些性能监控工具进行排查，比如说 jps、jstat、jinfo、jmap、jstack、jcmd 等等。


- 也可以使用一些可视化的性能监控工具，比如说 JConsole、VisualVM 等。




### 聊聊线程同步和互斥？

互斥，就是不同线程通过竞争进入临界区（共享数据或者硬件资源），为了防止冲突，在有限的时间内只允许其中一个线程独占使用共享资源。如不允许同时写。

同步，就是多个线程彼此合作，通过一定的逻辑关系来共同完成一个任务。一般来说，**同步关系中往往包含了互斥关系。**同时，临界区的资源会按照某种逻辑顺序进行访问。如先生产后使用。

锁要处理的问题大概有四种：

- 谁拿到了锁，可以是当前 class，可以是某个 lock 对象，或者实例的 markword；
- **抢占锁的规则，只能一个人抢 Mutex；能抢有限多次（Semaphore）；自己可以反复抢（可重入锁 ReentrantLock）；读可以反复抢，写只能一个人抢（读写锁ReadWriteLock）；**
- 抢不到怎么办，等待，等待的时候怎么等，自旋，阻塞，或者超时；
- 锁被释放了还有其他等待锁的怎么办？通知所有人一起抢或者只告诉一个人抢（Condition 的 signalAll 或者 signal）



所谓同步，即**协同步调，按预定的先后次序访问共享资源**，以免造成混乱。

当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到该线程完成操作，其他线程才能对该内存地址进行操作。

线程同步的实现方式有 6 种：互斥量、读写锁、条件变量、自旋锁、屏障、信号量。

- **互斥量**：互斥量（mutex）是一种最基本的同步手段，本质上是一把锁，在访问共享资源前先对互斥量进行加锁，访问完后再解锁。对互斥量加锁后，任何其他试图再次对互斥量加锁的线程都会被阻塞，直到当前线程解锁。
- **读写锁**：[读写锁](https://javabetter.cn/thread/ReentrantReadWriteLock.html)有三种状态，**读模式加锁、写模式加锁和不加锁**；一次只有一个线程可以占有写模式的读写锁，但是可以有多个线程同时占有读模式的读写锁。非常适合读多写少的场景。
- **条件变量**：[条件变量](https://javabetter.cn/thread/condition.html)是一种同步手段，它允许线程在满足特定条件时才继续执行，否则进入等待状态。条件变量通常与互斥量一起使用，以防止竞争条件的发生。
- **自旋锁**：自旋锁是一种锁的实现方式，它不会让线程进入睡眠状态，而是一直循环检测锁是否被释放。自旋锁适用于锁的持有时间非常短的情况。
- **信号量**：信号量（[Semaphore](https://javabetter.cn/thread/CountDownLatch.html)）本质上是一个计数器，用于为多个进程提供共享数据对象的访问。



#### 互斥和同步在时间上有要求吗？

互斥和同步在时间上是有一定要求的，因为它们都**涉及到对资源的访问顺序和时机控制**。

- 虽然互斥的重点不是线程执行的顺序，但它对访问的时间点有严格要求，以确保没有多个线程在同一时刻访问相同的资源。
- 同步强调的是线程之间的执行顺序和时间点的配合，特别是在多个线程需要依赖于彼此的执行结果时。





### :star:聊聊悲观锁和乐观锁？

- 乐观锁多用于“读多写少“的环境，避免频繁加锁影响性能；
- 悲观锁多用于”写多读少“的环境，避免频繁失败和重试影响性能。







## 并发工具类

### CountDownLatch（倒计数器）了解吗？

CountDownLatch 是 JUC 包中的一个同步工具类，用于**协调多个线程之间的同步**。它允许**一个或多个**线程等待，直到其他线程中执行的一组操作完成。

- 初始化：创建 CountDownLatch 对象时，指定计数器的初始值。
- 等待（await）：一个或多个线程调用 await 方法，进入等待状态，直到计数器的值变为零。
- 倒计数（countDown）：其他线程在完成各自任务后调用 countDown 方法，将计数器的值减一。当计数器的值减到零时，所有在 await 上等待的线程会被唤醒，继续执行。

![image-20241218091241727](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412180912871.png)

CountDownLatch 的**核心方法**也不多：

- `CountDownLatch(int count)`：创建一个带有给定计数器的 CountDownLatch。
- `void await()`：阻塞当前线程，直到计数器为零。
- `void countDown()`：递减计数器的值，如果计数器值变为零，则释放所有等待的线程。



### CyclicBarrier（同步屏障）了解吗？

CyclicBarrier 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一 组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续运行。

它和 CountDownLatch 类似，都可以协调多线程的结束动作，在它们结束后都可以执行特定动作，因为 CountDownLatch 的使用是**一次性**的，无法重复利用，而这里等待了两次。此时，我们用 CyclicBarrier 就可以实现，因为它可以**重复利用**。



CyclicBarrier 最最核心的方法，仍然是 await()：

- 如果当前线程不是第一个到达屏障的话，它将会进入等待，直到其他线程都到达，除非发生**被中断**、**屏障被拆除**、**屏障被重设**等情况；

![CyclicBarrier工作流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412180916585.png)

### **主要方法**

1. **`await()`**：
   - 每个线程调用此方法表示已到达屏障点。
   - 当所有线程都调用 `await()` 时，屏障打开，所有线程继续执行。
2. **`reset()`**：
   - 重置屏障到初始状态。如果有线程正在等待，会抛出 `BrokenBarrierException`。
3. **`getNumberWaiting()`**：
   - 返回当前在屏障点等待的线程数。
4. **`isBroken()`**：
   - 检查屏障是否已被破坏（由于超时或中断）。



### CyclicBarrier 和 CountDownLatch 有什么区别？

- CountDownLatch 是**一次性**的，而 CyclicBarrier 则可以多次设置屏障，实现**重复利用**；
- CountDownLatch 中的各个子线程**不可以等待其他线程**，只能完成自己的任务；而 CyclicBarrier 中的各个线程**可以等待其他线程**

| CyclicBarrier                                                | CountDownLatch                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| CyclicBarrier 是**可重用**的，其中的线程会等待所有的线程完成任务。届时，屏障将被拆除，并可以选择性地做一些特定的动作。 | CountDownLatch 是**一次性**的，不同的线程在同一个计数器上工作，直到计数器为 0. |
| CyclicBarrier 面向的是**线程数**                             | CountDownLatch 面向的是**任务数**                            |
| 在使用 CyclicBarrier 时，你必须在构造中**指定参与协作的线程数**，这些线程必须调用 await()方法 | 使用 CountDownLatch 时，则必须要**指定任务数**，至于这些任务由哪些线程完成无关紧要 |
| CyclicBarrier 可以在所有的线程释放后**重新使用**             | CountDownLatch 在计数器为 0 时**不能再使用**                 |
| 在 CyclicBarrier 中，如果某个线程遇到了中断、超时等问题时，则**处于 await 的线程都会出现问题** | 在 CountDownLatch 中，如果某个线程出现问题，其他线程不受影响 |



### Semaphore（信号量）了解吗？

Semaphore 的本质就是**协调多个线程对共享资源的获取**。

![image-20241218092357376](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412180923514.png)

**核心方法**：

- `acquire()`：获取一个许可，若无可用许可，线程阻塞。
- `release()`：释放一个许可。
- `availablePermits()`：返回当前可用的许可数量。
- `tryAcquire()`：尝试获取许可，不阻塞。



1. **限制数据库连接并发量** 如果连接池的最大连接数是 10，可以用 `Semaphore` 来控制同时发起的数据库连接数量。
2. **限制文件写操作的并发量** 如果多个线程要写入同一个文件，可以通过 `Semaphore` 限制同时写入的线程数，避免文件被频繁锁定。
3. **API 限流** 如果调用某个第三方 API 时有并发限制（如每秒只能处理 5 个请求），可以用 `Semaphore` 来限制每秒的请求数量。





### Exchanger 了解吗？

Exchanger 用于进行线程间的**数据交换**。`Exchanger` 只能在两个线程之间使用。提供一个**同步点**，在这个同步点，两个线程可以交换彼此的数据。

这两个线程通过 exchange 方法交换数据，如果第一个线程先执行 exchange()方法，它会**一直阻塞等待第二个线程**也执行 exchange 方法，当两个线程**都到达同步点时，这两个线程就可以交换数据**，将本线程生产出来的数据传递给对方。

Exchanger 可以用于**遗传算法**

#### 核心方法：

`exchange(T x)`：

- 线程调用此方法时会阻塞，直到另一个线程调用该方法。
- 方法会返回从对方线程接收到的对象。

`exchange(T x, long timeout, TimeUnit unit)`：

- 和上面的 `exchange()` 类似，但带有超时功能。如果在指定时间内另一个线程没有调用 `exchange()`，则抛出 `TimeoutException`。



### ConcurrentHashMap

#### 能说一下 ConcurrentHashMap 的实现吗？

在 JDK 7 时采用的是**分段锁机制**（Segment Locking），整个 Map 被分为若干段，每个段都可以独立地加锁。因此，不同的线程可以同时操作不同的段，从而实现并发访问。![初念初恋：JDK 7 ConcurrentHashMap](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412180934375.png)

在 JDK 8 及以上版本中，ConcurrentHashMap 的实现进行了优化，不再使用分段锁，而是使用了一种更加精细化的锁——**桶锁**，以及 **CAS 无锁**算法。每个桶（Node 数组的每个元素）都可以独立地加锁，从而实现更高级别的并发访问。

[![初念初恋：JDK 8 ConcurrentHashMap](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412180934462.png)](https://camo.githubusercontent.com/41ce1350bca68065c0ab736c990b1854691f809c515c8c971c0ba8f3c4c44ede/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6d61702d32303233303831363135353932342e706e67)

- 对于**读操作**，**通常不需要加锁**，可以直接读取，ConcurrentHashMap 内部使用了 **volatile** 变量来保证内存可见性。
- 对于**写操作**，ConcurrentHashMap 使用 **CAS** 操作来实现无锁的更新，这是一种乐观锁的实现，因为它假设没有冲突发生，在实际更新数据时才检查是否有其他线程在尝试修改数据，如果有，采用悲观的锁策略，如 synchronized 代码块来保证数据的一致性。






#### 说一下 JDK 7 中的 ConcurrentHashMap 的实现原理？

JDK 7 的 ConcurrentHashMap 是由 Segment 数组结构和 HashEntry 数组构成的。Segment 是一种可重入的锁 [ReentrantLock](https://javabetter.cn/thread/reentrantLock.html)，HashEntry 则用于存储键值对数据。

一个 ConcurrentHashMap 里包含一个 Segment 数组，Segment 的结构和 HashMap 类似，是一种数组和链表结构，一个 Segment 里包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素，每个 Segment 守护着一个 HashEntry 数组里的元素，当对 HashEntry 数组的数据进行修改时，必须首先获得它对应的 **Segment 锁。**

[![三分恶面渣逆袭：ConcurrentHashMap示意图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412180937815.png)](https://camo.githubusercontent.com/f6310b2f37a7e39863b98bc8dd2043f532f7b0fec6f21f6480a5c008f724d1b5/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d33312e706e67)

**①、put 流程**

ConcurrentHashMap 的 put 流程和 HashMap 非常类似，只不过是先定位到具体的 Segment，然后通过 ReentrantLock 去操作而已。

1. 计算 hash，定位到 segment，segment 如果是空就先初始化；
2. 使用 ReentrantLock 加锁，如果获取锁失败则尝试自旋，自旋超过次数就阻塞获取，保证一定能获取到锁；
3. 遍历 HashEntry，key 相同就直接替换，不存在就头插法插入，容量不足就扩容。
4. 释放锁。

[![三分恶面渣逆袭：JDK7 put 流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412180938943.png)](https://camo.githubusercontent.com/a347461b98ba1cdcf3b17fb650dfc41914123ae4957554de588c051f8125cf6c/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f73747574796d6f72652f6a6176617468726561642d32303234303332353131333335312e706e67)

**②、get 流程**

get 也很简单，通过 `hash(key)` 定位到 segment，再遍历链表定位到具体的元素上，需要注意的是 value 是 [volatile 的](https://javabetter.cn/thread/volatile.html)，所以 **get 是不需要加锁的**。





#### 说一下 JDK 8 中的 ConcurrentHashMap 的实现原理？

JDK 8 中的 ConcurrentHashMap 取消了 Segment 分段锁，采用 CAS + synchronized 来保证并发安全性，整个容器只分为一个 Segment，即 table 数组。

Node 和 JDK 7 一样，使用 volatile 关键字，保证多线程操作时，变量的可见性。

ConcurrentHashMap 实现线程安全的关键点在于 put 流程。

**①、put 流程**

> 一句话：通过计算键的哈希值确定存储位置，如果桶为空，使用 CAS 插入节点；如果存在冲突，通过链表或红黑树插入。在冲突时，如果 CAS 操作失败，会退化为 synchronized 操作。写操作可能触发扩容或链表转为红黑树。

[![三分恶面渣逆袭：Java 8 put 流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412180941046.jpeg)](https://camo.githubusercontent.com/6df94dd180aae46eada839b83dca109e8ff33c727d13bb5b775b3f7460c0173d/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f636f6c6c656374696f6e2d33322e6a7067)

**②、get 查询**

> 通过计算哈希值快速定位桶，在桶中查找目标节点，多个 key 值时链表遍历和红黑树查找。读操作是无锁的，依赖 volatile 保证线程可见性。



如果**hash值小于 0**，说明是个特殊节点，会调用节点的 **find 方法**进行查找，比如说 ForwardingNode 的 find 方法或者 TreeNode 的 find 方法。

**`ForwardingNode`** 和 **`TreeNode`** 是 `ConcurrentHashMap` 中的两种特殊节点类型。

- **`ForwardingNode`** **转发节点**，在**扩容过程中**用于指示元素已迁移到新哈希表，`find()` 方法帮助跳转到新哈希表查找。
- **`TreeNode`** 用于存储**红黑树**，处理大量哈希冲突的情况，`find()` 方法在树中查找元素。

`hash < 0` 是一个特殊标记，指示当前节点是 `ForwardingNode` 或 `TreeNode`，需要通过相应的 `find()` 方法来查找元素。





#### 总结一下 HashMap 和 ConcurrentHashMap 的区别？

- 首先是 hash 的计算方法上，ConcurrentHashMap 的 spread 方法接收一个已经计算好的 hashCode，然后将这个哈希码的高 16 位与自身进行异或运算，这里的 **HASH_BITS 是一个常数，值为 0x7fffffff，它确保结果是一个非负整数。**比 HashMap 的 hash 计算多了一个 `& HASH_BITS` 的操作。

  ```java
  static final int spread(int h) {
      return (h ^ (h >>> 16)) & HASH_BITS;
  }
  ```

- 另外，ConcurrentHashMap 对节点 Node 做了进一步的封装，比如说用 Forwarding Node 来表示正在进行扩容的节点。

- 最后就是 put 方法，通过 CAS + synchronized 来保证线程安全。



#### ConcurrentHashMap 扩容机制

与传统的 `HashMap` 类似，`ConcurrentHashMap` 也有 **扩容机制**，即当表的负载因子达到一定阈值时，哈希表会扩容。但在 `ConcurrentHashMap` 中，扩容过程是 **分段进行的**，即不**同段会并行进行扩容操作，而不会影响其他段的操作**。

- 在扩容过程中，只会锁定当前正在扩容的段，其他段仍然可以并行地进行操作。
- 这种分段扩容机制**能够避免全表扩容带来的性能瓶颈**。








#### 为什么 ConcurrentHashMap 在 JDK 1.7 中要用 ReentrantLock，而在 JDK 1.8 要用 synchronized

ConcurrentHashMap 在 JDK 1.7 和 JDK 1.8 中的实现机制不同，主要体现在锁的机制上。

JDK 1.7 中的 ConcurrentHashMap 使用了分段锁机制，即 Segment 锁，每个 Segment 都是一个 ReentrantLock，这样可以保证**每个 Segment 都可以独立地加锁**，从而实现更高级别的并发访问。

而在 JDK 1.8 中，ConcurrentHashMap 取消了 Segment 分段锁，采用了更加精细化的锁——桶锁，以及 CAS 无锁算法，**每个桶（Node 数组的每个元素）都可以独立地加锁**，从而实现更高级别的并发访问。

再加上 JVM 对 synchronized 做了大量优化，如锁消除、锁粗化、自旋锁和偏向锁等，在低中等的竞争情况下，synchronized 的性能并不比 ReentrantLock 差，并且使用 synchronized 可以简化代码实现。





### ConcurrentHashMap 怎么保证可见性？

ConcurrentHashMap 保证可见性主要通过使用 **volatile 关键字和 synchronized 同步块**。

Segment 数组和 Node 数组被声明为volatile。

此外，ConcurrentHashMap 还使用了 synchronized 同步块来保证复合操作的原子性。当一个线程进入 synchronized 同步块时，它会**获得锁，然后执行同步块内的代码。当它退出 synchronized 同步块时，它会释放锁，并将在同步块内对共享变量的所有修改立即刷新到主内存，**这样其他线程就可以看到这些修改了。





### 为什么 ConcurrentHashMap 比 Hashtable 效率高？

Hashtable 在任何时刻**只允许一个线程访问**整个 Map，通过**对整个 Map 加锁**来实现线程安全。

而 ConcurrentHashMap（尤其是在 JDK 8 及之后版本）通过锁分离和 CAS 操作实现更细粒度的锁定策略，允许更高的并发。





### 能说一下 CopyOnWriteArrayList 的实现原理吗？

CopyOnWriteArrayList 是一个线程安全的 ArrayList，它遵循**写时复制**（Copy-On-Write）的原则，即在写操作时，会先复制一个新的数组，然后在新的数组上进行写操作，写完之后再将原数组引用指向新数组。

[![CL0610：最终一致性](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412180953663.png)](https://camo.githubusercontent.com/09dfeac97f7318ad555a711970e1f7b67b4914f9c7e52b139ed60d94341749b6/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f7468726561642f436f70794f6e577269746541727261794c6973742d30312e706e67)

这样，**读操作总是在一个不变的数组版本上进行的，就不需要同步了。**





### 能说一下 BlockingQueue 吗？

[BlockingQueue](https://javabetter.cn/thread/BlockingQueue.html) 代表的是线程安全的队列，不仅可以**由多个线程并发访问**，还添加了**等待/通知**机制，以便在**队列为空时阻塞获取元素的线程，直到队列变得可用，或者在队列满时阻塞插入元素的线程，直到队列变得可用。**

BlockingQueue 接口的实现类有 ArrayBlockingQueue、DelayQueue、LinkedBlockingDeque、LinkedBlockingQueue、LinkedTransferQueue、PriorityBlockingQueue、SynchronousQueue 等。



#### 阻塞队列是如何实现的？

拿 ArrayBlockingQueue 来说，它是一个基于数组的有界阻塞队列，采用 **ReentrantLock 锁来实现线程的互斥**，而 ReentrantLock 底层采用的是 **AQS 实现的队列同步**，

线程的阻塞调用 LockSupport.**park**实现，唤醒调用 LockSupport.**unpark** 实现。







## 线程池

### 什么是线程池？

①、频繁地创建和销毁线程会消耗系统资源，线程池能够复用已创建的线程。

②、提高响应速度，当任务到达时，任务可以不需要等待线程创建就立即执行。

③、线程池支持定时执行、周期性执行、单线程执行和并发数控制等功能。



### 能说说工作中线程池的应用吗？

推荐阅读：[线程池在美团业务中的应用](https://tech.meituan.com/2020/04/02/java-pooling-pratice-in-meituan.html)

**①、快速响应用户请求**

使用线程池，可以预先创建一定数量的线程，当用户请求到来时，直接从线程池中获取一个空闲线程，执行用户请求，执行完毕后，线程不销毁，而是继续保留在线程池中，等待下一个请求。

注意：这种场景下需要调高 corePoolSize 和 maxPoolSize，尽可能多创建线程，**避免使用队列去缓存任务**。

比如说，在[技术派实战项目](https://javabetter.cn/zhishixingqiu/paicoding.html)中，当用户请求首页时，就使用了线程池去加载首页的热门文章、置顶文章、侧边栏、用户登录信息等。我们封装了一个异步类 AsyncUtil，内部的静态类 CompletableFutureBridge 是通过 [CompletableFuture](https://tech.meituan.com/2022/05/12/principles-and-practices-of-completablefuture.html) 实现的，其中的 `runAsyncWithTimeRecord()` 方法就是使用线程池去执行任务的。

其中线程池的初始化中，corePoolSize 为 CPU 核心数的两倍，因为技术派中的大多数任务都是 IO 密集型的，maxPoolSize 设置为 50，是一个比较理想的值，尤其是在本地环境中；阻塞队列为 SynchronousQueue，这意味着任务被创建后直接提交给等待的线程处理，而不是放入队列中。

**②、快速处理批量任务**

这种场景也需要处理大量的任务，但可能不需要立即响应，这时候就应该设置队列去缓冲任务，corePoolSize 不需要设置得太高，**避免线程上下文切换引起的频繁切换问题**。



### 能简单说一下线程池的工作流程吗？

第一步，创建线程池。

第二步，调用线程池的 `execute()`方法，提交任务。

- 如果正在运行的线程数量小于 corePoolSize，那么线程池会创建一个新的线程来执行这个任务；
- 如果正在运行的线程数量大于或等于 corePoolSize，那么线程池会将这个任务放入等待队列；
- 如果等待队列满了，而且正在运行的线程数量小于 maximumPoolSize，那么线程池会创建新的线程来执行这个任务；
- 如果等待队列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会执行拒绝策略。

[![三分恶面渣逆袭：线程池执行流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181011851.png)](https://camo.githubusercontent.com/3339b72aa8601a34e669587112401fdefbbd21b7a1820b8ba5286d9d6145cb53/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d36362e706e67)

第三步，线程执行完毕后，线程并不会立即销毁，而是继续保持在池中等待下一个任务。

第四步，当线程空闲时间超出指定时间，且当前线程数量大于核心线程数时，线程会被回收。





### :star:线程池主要参数有哪些？

线程池有 7 个参数，需要重点关注`corePoolSize`、`maximumPoolSize`、`workQueue`、`handler` 这四个。

[![三分恶面渣逆袭：线程池参数](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181014677.png)](https://camo.githubusercontent.com/497c21528ecded5c89ae7efa40772972cebd12722276fd82c13f7dabfb0848fe/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d36372e706e67)

**①、corePoolSize** 定义了线程池中的**核心线程数量**。即使这些线程处于空闲状态，它们也不会被回收。这是线程池保持在等待状态下的线程数。

**②、maximumPoolSize** 是线程池允许的**最大线程数量**。当工作队列满了之后，线程池会创建新线程来处理任务，直到线程数达到这个最大值。

**③、workQueue**用于存放待处理任务的**阻塞队列**。当所有核心线程都忙时，新任务会被放在这个队列里等待执行。

**④、handler**，**拒绝策略** RejectedExecutionHandler，定义了当线程池和工作队列都满了之后对新提交的任务的处理策略。常见的拒绝策略包括抛出异常、直接丢弃、丢弃队列中最老的任务、由提交任务的线程来直接执行任务等。

**⑤、threadFactory**指**创建新线程的工厂**。它用于创建线程池中的线程。可以通过自定义 ThreadFactory 来给线程池中的线程设置有意义的名字，或设置优先级等。

**⑥、keepAliveTime**指**非核心线程的空闲存活时间**。如果线程池中的线程数量超过了 corePoolSize，那么这些多余的线程在空闲时间超过 keepAliveTime 时会被终止。

**⑦、unit**，keepAliveTime 参数的**时间单位**：

- TimeUnit.DAYS; 天
- TimeUnit.HOURS; 小时
- TimeUnit.MINUTES; 分钟
- TimeUnit.SECONDS; 秒
- TimeUnit.MILLISECONDS; 毫秒
- TimeUnit.MICROSECONDS; 微秒
- TimeUnit.NANOSECONDS; 纳秒



#### 能简单说一下参数之间的关系吗？

①、corePoolSize 和 maximumPoolSize 共同定义了线程池的规模。

- 当提交的任务数不足以填满核心线程时，线程池只会创建足够的线程来处理任务。
- 当任务数增多，超过核心线程的处理能力时，任务会被加入 workQueue。
- 如果 workQueue 已满，而当前线程数又小于 maximumPoolSize，线程池会尝试创建新的线程来处理任务。

②、keepAliveTime 和 unit 决定了非核心线程可以空闲存活多久。这会影响了线程池的资源回收策略。

③、workQueue 的选择对线程池的行为有重大影响。不同类型的队列（如无界队列、有界队列）会导致线程池在任务增多时的反应不同。

④、handler 定义了线程池的饱和策略，即当线程池无法接受新任务时的行为。决定了系统在极限情况下的表现。



#### 核心线程数不够会怎么进行处理？

**当提交的任务数超过了 corePoolSize，但是小于 maximumPoolSize 时，线程池会创建新的线程来处理任务。**

**当提交的任务数超过了 maximumPoolSize 时，线程池会根据拒绝策略来处理任务。**

**核心线程会一直运行，而超出核心线程数的线程，如果空闲时间超过 keepAliveTime，将会被终止，直到线程池的线程数减少到 corePoolSize。**





### 线程池的拒绝策略有哪些？

主要有四种：

- AbortPolicy：这是**默认的拒绝策略**。该策略会**抛出**一个 RejectedExecutionException 异常。
- CallerRunsPolicy：该策略**不会抛出异常**，而是会**让提交任务的线程**（即调用 execute 方法的线程）**自己来执行这个任务。**
- DiscardOldestPolicy：策略会**丢弃队列中最老的一个任务**（即队列中等待最久的任务），然后**尝试重新提交被拒绝的任务。**
- DiscardPolicy：策略会**默默地丢弃被拒绝的任务，不做任何处理也不抛出异常。**

<img src="https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181022729.png" alt="image-20241218102209579" style="zoom: 50%;" />





### 线程池有哪几种阻塞队列？

[
![三分恶面渣逆袭：线程池常用阻塞队列](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181031434.png)](https://camo.githubusercontent.com/17995445e1e1ebf55082632349ff9a255d0f7d5ef713aa096414167435a70a90/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d36392e706e67)

①、ArrayBlockingQueue：一个有界的先进先出的阻塞队列，底层是一个数组，适合固定大小的线程池。

②、LinkedBlockingQueue：底层数据结构是链表，如果不指定大小，默认大小是 Integer.MAX_VALUE，相当于一个无界队列。

[技术派实战项目](https://javabetter.cn/zhishixingqiu/paicoding.html)中，就使用了 LinkedBlockingQueue 来配置 RabbitMQ 的消息队列。

③、PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。任务按照其自然顺序或通过构造器给定的 Comparator 来排序。

适用于需要按照给定优先级处理任务的场景，比如优先处理紧急任务。

④、DelayQueue：类似于 PriorityBlockingQueue，由**二叉堆**实现的无界优先级阻塞队列。

Executors 中的 `newScheduledThreadPool()` 就使用了 DelayQueue 来实现延迟执行。

⑤、SynchronousQueue：实际上它不是一个真正的队列，因为没有容量。每个插入操作必须等待另一个线程的移除操作，同样任何一个移除操作都必须等待另一个线程的插入操作。

`Executors.newCachedThreadPool()` 就使用了 SynchronousQueue，这个线程池会根据需要创建新线程，如果有空闲线程则会重复使用，线程空闲 60 秒后会被回收。





### 线程池提交 execute 和 submit 有什么区别？

1. execute 用于**提交不需要返回值的任务**
2. submit()方法用于**提交需要返回值的任务**。线程池会**返回一个 future 类型的对象**，通过这个 future 对象可以**判断任务是否执行成功**，并且可以通过 **future 的 get()方法来获取返回值**





### 线程池怎么关闭知道吗？

可以通过调用线程池的`shutdown`或`shutdownNow`方法来关闭线程池。

它们的原理是遍历线程池中的工作线程，然后**逐个调用线程的 interrupt 方法来中断线程**，所以无法响应中断的任务可能永远无法终止。

**shutdown() 将线程池状态置为 shutdown,并不会立即停止**：

1. 停止接收外部 submit 的任务
2. 内部正在跑的任务和队列里等待的任务，会执行完
3. 等到第二步完成后，才真正停止

**shutdownNow() 将线程池状态置为 stop。一般会立即停止，事实上不一定**：

1. 和 shutdown()一样，先停止接收外部提交的任务
2. 忽略队列里等待的任务
3. 尝试将正在跑的任务 interrupt 中断
4. 返回未执行的任务列表

shutdown 和 shutdownnow 简单来说区别如下：

- shutdownNow()能**立即停止线程池**，正在跑的和正在等待的任务都停下了。这样做立即生效，但是风险也比较大。
- shutdown()只是**关闭了提交通道**，用 submit()是无效的；而内部的任务该怎么跑还是怎么跑，跑完再彻底停止线程池。



### 线程池的线程数应该怎么配置？

分析线程池中执行的任务类型是 CPU 密集型还是 IO 密集型？

①、对于 **CPU 密集型**任务，我的目标是尽量**减少线程上下文切换**，以优化 CPU 使用率。一般来说，核心线程数设置为**处理器的核心数或核心数加一**（以备不时之需，如某些线程因等待系统资源而阻塞时）是较理想的选择。

②、对于 **IO 密集型**任务，由于线程经常处于等待状态（等待 IO 操作完成），可以设置更多的线程来提高并发性（比如说 **2 倍**），从而增加 CPU 利用率。



核心数可以通过 Java 的`Runtime.getRuntime().availableProcessors()`方法获取。

此外，每个线程都会占用一定的内存，因此我需要确保线程池的规模不会耗尽 JVM 内存，避免频繁的垃圾回收或内存溢出。

最后，我会根据业务需求和系统资源来调整线程池的参数，比如核心线程数、最大线程数、非核心线程的空闲存活时间、任务队列容量等。



#### 如何知道你设置的线程数多了还是少了？

可以先通过 top 命令观察 CPU 的使用率，如果 CPU 使用率较低，可能是线程数过少；如果 CPU 使用率接近 100%，但吞吐量未提升，可能是线程数过多。

然后再通过 JProfiler、VisualVM 或 Arthas 分析线程运行情况，查看线程的状态、等待时间、运行时间等信息，进一步调整线程池的参数。



### 有哪几种常见的线程池？

![image-20241218105353913](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181053081.png)

可以通过 Executors 工厂类来创建四种线程池：

- newFixedThreadPool (固定线程数目的线程池)
- newCachedThreadPool (可缓存线程的线程池)
- newSingleThreadExecutor (单线程的线程池)
- newScheduledThreadPool (定时及周期执行的线程池)





### 能说一下四种常见线程池的原理吗？

前三种线程池的构造**直接调用 ThreadPoolExecutor 的构造方法**。



#### newSingleThreadExecutor 单线程的线程池

**线程池特点**

- **核心线程数为 1**
- **最大线程数也为 1**
- 阻塞队列是无界队列 **LinkedBlockingQueue**，可能会导致 OOM（内存不足）
- **keepAliveTime 为 0**

[![SingleThreadExecutor运行流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181056505.png)](https://camo.githubusercontent.com/7eb2193315fb8ea48d08e8f0d178e86836f2b04617a1265f124b97556e591430/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d37322e706e67)

工作流程：

- 提交任务
- 线程池是否有一条线程在，如果没有，新建线程执行任务
- 如果有，将任务加到阻塞队列
- 当前的唯一线程，从队列取任务，执行完一个，再继续取，一个线程执行任务。

**适用场景**

适用于串行执行任务的场景，一个任务一个任务地执行。



#### newFixedThreadPool 固定线程数目的线程池

**线程池特点：**

- **核心线程数和最大线程数大小一样**
- 没有所谓的非空闲时间，即 **keepAliveTime 为 0**
- 阻塞队列为无界队列 **LinkedBlockingQueue**，可能会导致 OOM

[![FixedThreadPool](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181057415.png)](https://camo.githubusercontent.com/4f85938df5c81907a802dfc504ed61fbfab09cb23d841a43799aba598497178c/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d37332e706e67)

工作流程：

- 提交任务
- 如果线程数少于核心线程，创建核心线程执行任务
- 如果线程数等于核心线程，把任务添加到 LinkedBlockingQueue 阻塞队列
- 如果线程执行完任务，去阻塞队列取任务，继续执行。

**使用场景**

FixedThreadPool 适用于处理 **CPU 密集型**的任务，确保 CPU 在长期被工作线程使用的情况下，尽可能的少的分配线程，即适用执行长期的任务。



#### newCachedThreadPool 可缓存线程的线程池

**线程池特点：**

- **核心线程数为 0**
- **最大线程数为 Integer.MAX_VALUE，即无限大**，可能会因为无限创建线程，导致 OOM
- 阻塞队列是 **SynchronousQueue**
- 非核心线程**空闲存活时间为 60 秒**

当提交任务的速度大于处理任务的速度时，**每次提交一个任务，就必然会创建一个线程**。极端情况下会创建过多的线程，耗尽 CPU 和内存资源。由于空闲 60 秒的线程会被终止，长时间保持空闲的 CachedThreadPool 不会占用任何资源。

[![CachedThreadPool执行流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181059394.png)](https://camo.githubusercontent.com/8e9fc55bfce9bf96fe181991c485691990f00d579c23ccf9354a43e29b2ada1e/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d37342e706e67)

工作流程：

- 提交任务
- 因为没有核心线程，所以任务直接加到 SynchronousQueue 队列。
- 判断是否有空闲线程，如果有，就去取出任务执行。
- 如果没有空闲线程，就新建一个线程执行。
- 执行完任务的线程，还可以存活 60 秒，如果在这期间，接到任务，可以继续活下去；否则，被销毁。

**适用场景**

用于**并发执行大量短期的小任务。**



#### newScheduledThreadPool 定时及周期执行的线程池

**线程池特点**

- **最大线程数为 Integer.MAX_VALUE**，也有 OOM 的风险
- 阻塞队列是 **DelayedWorkQueue**
- **keepAliveTime 为 0**
- scheduleAtFixedRate() ：按某种速率**周期执行**
- scheduleWithFixedDelay()：在某个**延迟后执行**

[![ScheduledThreadPool执行流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181100023.png)](https://camo.githubusercontent.com/b500c940ff51e9b53caa429db792cf0c3892526dc1dc8eea33ef0da3f8af6cfb/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d37352e706e67)

**工作机制**

- 线程从 DelayQueue 中获取已到期的 ScheduledFutureTask（DelayQueue.take()）。到期任务是指 ScheduledFutureTask 的 time 大于等于当前时间。
- 线程执行这个 ScheduledFutureTask。
- 线程修改 ScheduledFutureTask 的 time 变量为下次将要被执行的时间。
- 线程把这个修改 time 之后的 ScheduledFutureTask 放回 DelayQueue 中（DelayQueue.add()）。

[![ScheduledThreadPoolExecutor执行流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181100317.png)](https://camo.githubusercontent.com/dbbeb0b4c1c99e2ebe2607b71976ce3e1abd2ec1a81e099fb979e64bf7dc053f/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d37362e706e67)

**使用场景**

**周期性执行任务**的场景，**需要限制线程数量**的场景



#### 使用无界队列的线程池会导致什么问题吗？

例如 newFixedThreadPool 使用了无界的阻塞队列 LinkedBlockingQueue，如果线程获取一个任务后，任务的执行时间比较长，会导致队列的任务越积越多，导致机器内存使用不停飙升，最终导致 OOM。





### 线程池异常怎么处理知道吗？

常见的异常处理方式：

[![线程池异常处理](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181103555.png)](https://camo.githubusercontent.com/93c7c2707de7180478c03e8243ba07af1749f48929eb08ef3765a6c4c44c81c5/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d37372e706e67)





### 能说一下线程池有几种状态吗？

线程池各个状态切换图：

[![线程池状态切换图](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181111077.png)](https://camo.githubusercontent.com/4aacb4fe81688033eb832385ee2532e597f49d532113fb418770efe0b521f370/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d37382e706e67)





### 线程池如何实现参数的动态修改？

线程池提供了几个 setter 方法来设置线程池的参数。

![image-20241218111446681](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181114848.png)

这里主要有两个思路：

[![动态修改线程池参数](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181113810.png)](https://camo.githubusercontent.com/6208ad694a0f26f5658dbac09c37a111e535b2bbf8ea3debe5c8c5e2205f111d/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d38302e706e67)

- 在我们微服务的架构下，可以利用配置中心如 Nacos、Apollo 等等，也可以自己开发配置中心。业务服务读取线程池配置，获取相应的线程池实例来修改线程池的参数。
- 如果限制了配置中心的使用，也可以自己去扩展**ThreadPoolExecutor**，重写方法，监听线程池参数变化，来动态修改线程池参数。



### 线程池调优了解吗？

![image-20241218111530863](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181115032.png)具体的调优案例可以查看参考[7]美团技术博客。



### 线程池在使用的时候需要注意什么？

我认为比较重要的关注点有 3 个：

①、**选择合适的线程池大小**

- **过小**的线程池可能会导致任务一直在排队
- **过大**的线程池可能会导致大家都在竞争 CPU 资源，增加上下文切换的开销

可以根据业务是 IO 密集型还是 CPU 密集型来选择线程池大小：

- CPU 密集型：指的是任务主要使用来进行大量的计算，没有什么导致线程阻塞。一般这种场景的线程数设置为 CPU 核心数+1。
- IO 密集型：当执行任务需要大量的 io，比如磁盘 io，网络 io，可能会存在大量的阻塞，所以在 IO 密集型任务中使用多线程可以大大地加速任务的处理。一般线程数设置为 2*CPU 核心数。

②、**任务队列的选择**

- 使用有界队列可以避免资源耗尽的风险，但是可能会导致任务被拒绝
- 使用无界队列虽然可以避免任务被拒绝，但是可能会导致内存耗尽

一般需要设置有界队列的大小，比如 LinkedBlockingQueue 在构造的时候可以传入参数来限制队列中任务数据的大小，这样就不会因为无限往队列中扔任务导致系统的 oom。

③、**尽量使用自定义的线程池，而不是使用 Executors 创建的线程池**，

因为 newFixedThreadPool 线程池由于使用了 LinkedBlockingQueue，队列的容量默认无限大，实际使用中出现任务过多时会导致内存溢出；newCachedThreadPool 线程池由于核心线程数无限大，当任务过多的时候会导致创建大量的线程，可能机器负载过高导致服务宕机。





### 你能设计实现一个线程池吗？

线程池的设计需要考虑这几个关键因素：

1. 核心线程池类：包含核心线程数、最大线程数。
2. 工作线程：线程池中实际工作的线程，从任务队列中获取任务并执行。
3. 任务队列：存放待执行任务的队列，可以使用阻塞队列实现。
4. 拒绝策略：当任务队列满时，处理新任务的策略。

[![三分恶面渣逆袭：线程池主要实现流程](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181117924.png)](https://camo.githubusercontent.com/e09f11b2ae230032cd538ac6092860189064c0ba97ddb6e9cdc3928a00a19f70/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d38332e706e67)



#### 写一个数据库连接池，你现在可以写一下？

数据库连接池的核心功能主要包括：

- 连接的获取和释放
- 限制最大连接数，避免资源耗尽
- 连接的复用，避免频繁创建和销毁连接

。。。。。。？？？？？？



### 单机线程池执行断电了应该怎么处理？

- 对阻塞队列持久化；
- 正在处理任务事务控制；
- 断电之后正在处理任务的回滚，通过日志恢复该次操作；
- 服务器重启后阻塞队列中的数据再加载。







## 并发容器和框架

### Fork/Join 框架了解吗？

Fork/Join 框架是 Java7 提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

要想掌握 Fork/Join 框架，首先需要理解两个点，**分而治之**和**工作窃取算法**。

**分而治之**

Fork/Join 框架的定义，其实就体现了分治思想：将一个规模为 N 的问题分解为 K 个规模较小的子问题，这些子问题相互独立且与原问题性质相同。求出子问题的解，就可得到原问题的解。

[![Fork/Join分治算法](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181126332.png)](https://camo.githubusercontent.com/63164ec7dd1507e4e889633d2ff91368bfe7309eb651f6587d2fdca45072cfa4/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d38352e706e67)

**工作窃取算法**

大任务拆成了若干个小任务，把这些小任务放到不同的队列里，各自创建单独线程来执行队列里的任务。

那么问题来了，有的线程干活块，有的线程干活慢。干完活的线程不能让它空下来，得让它去帮没干完活的线程干活。它**去其它线程的队列里窃取一个任务来执行**，这就是所谓的**工作窃取**。

工作窃取发生的时候，它们会**访问同一个队列**，为了减少窃取任务线程和被窃取任务线程之间的竞争，通常任务会使用**双端队列**，被窃取任务线程永远从双端队列的头部拿，而窃取任务的线程永远从双端队列的尾部拿任务执行。

[![工作窃取](https://raw.githubusercontent.com/raosirui/Picture/main/markdown/202412181126995.png)](https://camo.githubusercontent.com/5ccc0bc2479a286a9ea3e03cc0b3a6d2714842410eb55a1303ad87d92e08ba22/68747470733a2f2f63646e2e746f62656265747465726a61766165722e636f6d2f746f62656265747465726a61766165722f696d616765732f736964656261722f73616e66656e652f6a6176617468726561642d38362e706e67)

