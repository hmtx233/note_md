

#线程池 #网上博客

>在《阿里巴巴java开发手册》中指出了线程资源必须通过线程池提供，不允许在应用中自行显示的创建线程，这样一方面是线程的创建更加规范，可以合理控制开辟线程的数量；另一方面线程的细节管理交给线程池处理，优化了资源的开销。而线程池不允许使用Executors去创建，而要通过ThreadPoolExecutor方式，这一方面是由于jdk中Executor框架虽然提供了如newFixedThreadPool()、newSingleThreadExecutor()、newCachedThreadPool()等创建线程池的方法，但都有其局限性，不够灵活；另外由于前面几种方法内部也是通过ThreadPoolExecutor方式实现，使用ThreadPoolExecutor有助于大家明确线程池的运行规则，创建符合自己的业务场景需要的线程池，避免资源耗尽的风险。

>下面我们就对ThreadPoolExecutor的使用方法进行一个详细的概述。

>首先看下ThreadPoolExecutor的构造函数

```java
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }

```

> 构造函数的参数含义如下：


- **corePoolSize:指定了线程池中的线程数量，它的数量决定了添加的任务是开辟新的线程去执行，还是放到**workQueue任务队列中去；****

- **maximumPoolSize:指定了线程池中的最大线程数量，这个参数会根据你使用的**workQueue任务队列的类型，决定线程池会开辟的最大线程数量；****

- **keepAliveTime:当线程池中空闲线程数量超过corePoolSize时，多余的线程会在多长时间内被销毁；**

- **unit:keepAliveTime的单位**

- **workQueue:任务队列，被添加到线程池中，但尚未被执行的任务；它一般分为直接提交队列、有界任务队列、无界任务队列、优先任务队列几种；**

- **threadFactory:线程工厂，用于创建线程，一般用默认即可；**

- **handler:拒绝策略；当任务太多来不及处理时，如何拒绝任务；**

> 接下来我们对其中比较重要参数做进一步的了解：

### 一、 **workQueue任务队列**

上面我们已经介绍过了，**它一般分为直接提交队列、有界任务队列、无界任务队列、优先任务队列；**

1、**直接提交队列**：设置为SynchronousQueue队列，SynchronousQueue是一个特殊的BlockingQueue，它没有容量，没执行一个插入操作就会阻塞，需要再执行一个删除操作才会被唤醒，反之每一个删除操作也都要等待对应的插入操作。

```java
public class ThreadPool {
    private static ExecutorService pool;
    public static void main( String[] args )
    {
        //maximumPoolSize设置为2 ，拒绝策略为AbortPolic策略，直接抛出异常
        pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new SynchronousQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
        for(int i=0;i<3;i++) {
            pool.execute(new ThreadTask());
        }   
    }
}

public class ThreadTask implements Runnable{
    
    public ThreadTask() {
        
    }
    
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}
```


> 输出结果为

```java
pool-1-thread-1
pool-1-thread-2
Exception in thread "main" java.util.concurrent.RejectedExecutionException: Task com.hhxx.test.ThreadTask@55f96302 rejected from java.util.concurrent.ThreadPoolExecutor@3d4eac69[Running, pool size = 2, active threads = 0, queued tasks = 0, completed tasks = 2]
    at java.util.concurrent.ThreadPoolExecutor$AbortPolicy.rejectedExecution(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.reject(Unknown Source)
    at java.util.concurrent.ThreadPoolExecutor.execute(Unknown Source)
    at com.hhxx.test.ThreadPool.main(ThreadPool.java:17)

```


> 可以看到，当任务队列为SynchronousQueue，创建的线程数大于maximumPoolSize时，直接执行了拒绝策略抛出异常。

>使用SynchronousQueue队列，提交的任务不会被保存，总是会马上提交执行。如果用于执行任务的线程数量小于maximumPoolSize，则尝试创建新的进程，如果达到maximumPoolSize设置的最大值，则根据你设置的handler执行拒绝策略。因此这种方式你提交的任务不会被缓存起来，而是会被马上执行，在这种情况下，你需要对你程序的并发量有个准确的评估，才能设置合适的maximumPoolSize数量，否则很容易就会执行拒绝策略；

### 2、**有界的任务队列**：有界的任务队列可以使用ArrayBlockingQueue实现，如下所示

```java
pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(10),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
```

使用ArrayBlockingQueue有界任务队列，若有新的任务需要执行时，线程池会创建新的线程，直到创建的线程数量达到corePoolSize时，则会将新的任务加入到等待队列中。若等待队列已满，即超过ArrayBlockingQueue初始化的容量，则继续创建线程，直到线程数量达到maximumPoolSize设置的最大线程数量，若大于maximumPoolSize，则执行拒绝策略。在这种情况下，线程数量的上限与有界任务队列的状态有直接关系，如果有界队列初始容量较大或者没有达到超负荷的状态，线程数将一直维持在corePoolSize以下，反之当任务队列已满时，则会以maximumPoolSize为最大线程数上限。

### 3、**无界的任务队列**：有界任务队列可以使用LinkedBlockingQueue实现，如下所示

```java
pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
```

>使用无界任务队列，线程池的任务队列可以无限制的添加新的任务，而线程池创建的最大线程数量就是你corePoolSize设置的数量，也就是说在这种情况下maximumPoolSize这个参数是无效的，哪怕你的任务队列中缓存了很多未执行的任务，当线程池的线程数达到corePoolSize后，就不会再增加了；若后续有新的任务加入，则直接进入队列等待，当使用这种任务队列模式时，一定要注意你任务提交与处理之间的协调与控制，不然会出现队列中的任务由于无法及时处理导致一直增长，直到最后资源耗尽的问题。

### 4、优先任务队列：**优先任务队列通过PriorityBlockingQueue实现，下面我们通过一个例子演示下

```java
public class ThreadPool {
    private static ExecutorService pool;
    public static void main( String[] args )
    {
        //优先任务队列
        pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new PriorityBlockingQueue<Runnable>(),Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
          
        for(int i=0;i<20;i++) {
            pool.execute(new ThreadTask(i));
        }    
    }
}

public class ThreadTask implements Runnable,Comparable<ThreadTask>{
    
    private int priority;
    
    public int getPriority() {
        return priority;
    }

    public void setPriority(int priority) {
        this.priority = priority;
    }

    public ThreadTask() {
        
    }
    
    public ThreadTask(int priority) {
        this.priority = priority;
    }

    //当前对象和其他对象做比较，当前优先级大就返回-1，优先级小就返回1,值越小优先级越高
    public int compareTo(ThreadTask o) {
         return  this.priority>o.priority?-1:1;
    }
    
    public void run() {
        try {
            //让线程阻塞，使后续任务进入缓存队列
            Thread.sleep(1000);
            System.out.println("priority:"+this.priority+",ThreadName:"+Thread.currentThread().getName());
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    
    }
}
```



> 我们来看下执行的结果情况
```java
priority:0,ThreadName:pool-1-thread-1
priority:9,ThreadName:pool-1-thread-1
priority:8,ThreadName:pool-1-thread-1
priority:7,ThreadName:pool-1-thread-1
priority:6,ThreadName:pool-1-thread-1
priority:5,ThreadName:pool-1-thread-1
priority:4,ThreadName:pool-1-thread-1
priority:3,ThreadName:pool-1-thread-1
priority:2,ThreadName:pool-1-thread-1
priority:1,ThreadName:pool-1-thread-1
```


> 大家可以看到除了第一个任务直接创建线程执行外，其他的任务都被放入了优先任务队列，按优先级进行了重新排列执行，且线程池的线程数一直为corePoolSize，也就是只有一个。
   通过运行的代码我们可以看出PriorityBlockingQueue它其实是一个特殊的无界队列，它其中无论添加了多少个任务，线程池创建的线程数也不会超过corePoolSize的数量，只不过其他队列一般是按照先进先出的规则处理任务，而PriorityBlockingQueue队列可以自定义规则根据任务的优先级顺序先后执行。

### **二、拒绝策略**

> 一般我们创建线程池时，为防止资源被耗尽，任务队列都会选择创建有界任务队列，但种模式下如果出现任务队列已满且线程池创建的线程数达到你设置的最大线程数时，这时就需要你指定ThreadPoolExecutor的RejectedExecutionHandler参数即合理的拒绝策略，来处理线程池"超载"的情况。ThreadPoolExecutor自带的拒绝策略如下：

#### **1、AbortPolicy策略：该策略会直接抛出异常，阻止系统正常工作；**

#### **2、CallerRunsPolicy策略：如果线程池的线程数量达到上限，该策略会把任务队列中的任务放在调用者线程当中运行；**

#### **3、DiscardOledestPolicy策略：该策略会丢弃任务队列中最老的一个任务，也就是当前任务队列中最先被添加进去的，马上要被执行的那个任务，并尝试再次提交；**

#### **4、DiscardPolicy策略：该策略会默默丢弃无法处理的任务，不予任何处理。当然使用此策略，业务场景中需允许任务的丢失；**

>**以上内置的策略均实现了**RejectedExecutionHandler接口，**当然你也可以自己扩展RejectedExecutionHandler接口，定义自己的拒绝策略，我们看下示例代码：**

```java
public class ThreadPool {
    private static ExecutorService pool;
    public static void main( String[] args )
    {
        //自定义拒绝策略
        pool = new ThreadPoolExecutor(1, 2, 1000, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(5),
                Executors.defaultThreadFactory(), new RejectedExecutionHandler() {
            public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                System.out.println(r.toString()+"执行了拒绝策略");
                
            }
        });
          
        for(int i=0;i<10;i++) {
            pool.execute(new ThreadTask());
        }    
    }
}

public class ThreadTask implements Runnable{    
    public void run() {
        try {
            //让线程阻塞，使后续任务进入缓存队列
            Thread.sleep(1000);
            System.out.println("ThreadName:"+Thread.currentThread().getName());
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    
    }
}
```


> 输出结果：

```java
com.hhxx.test.ThreadTask@33909752执行了拒绝策略
com.hhxx.test.ThreadTask@55f96302执行了拒绝策略
com.hhxx.test.ThreadTask@3d4eac69执行了拒绝策略
ThreadName:pool-1-thread-2
ThreadName:pool-1-thread-1
ThreadName:pool-1-thread-1
ThreadName:pool-1-thread-2
ThreadName:pool-1-thread-1
ThreadName:pool-1-thread-2
ThreadName:pool-1-thread-1
```


> 可以看到由于任务加了休眠阻塞，执行需要花费一定时间，导致会有一定的任务被丢弃，从而执行自定义的拒绝策略；

### **三、ThreadFactory自定义线程创建**

 线程池中线程就是通过ThreadPoolExecutor中的ThreadFactory，线程工厂创建的。那么通过自定义ThreadFactory，可以按需要对线程池中创建的线程进行一些特殊的设置，如命名、优先级等，下面代码我们通过ThreadFactory对线程池中创建的线程进行记录与命名

```java
public class ThreadPool {
    private static ExecutorService pool;
    public static void main( String[] args )
    {
        //自定义线程工厂
        pool = new ThreadPoolExecutor(2, 4, 1000, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(5),
                new ThreadFactory() {
            public Thread newThread(Runnable r) {
                System.out.println("线程"+r.hashCode()+"创建");
                //线程命名
                Thread th = new Thread(r,"threadPool"+r.hashCode());
                return th;
            }
        }, new ThreadPoolExecutor.CallerRunsPolicy());
          
        for(int i=0;i<10;i++) {
            pool.execute(new ThreadTask());
        }    
    }
}

public class ThreadTask implements Runnable{    
    public void run() {
        //输出执行线程的名称
        System.out.println("ThreadName:"+Thread.currentThread().getName());
    }
}
```


> 我们看下输出结果

```java
线程118352462创建
线程1550089733创建
线程865113938创建
ThreadName:threadPool1550089733
ThreadName:threadPool118352462
线程1442407170创建
ThreadName:threadPool1550089733
ThreadName:threadPool1550089733
ThreadName:threadPool1550089733
ThreadName:threadPool865113938
ThreadName:threadPool865113938
ThreadName:threadPool118352462
ThreadName:threadPool1550089733
ThreadName:threadPool1442407170

```

> 可以看到线程池中，每个线程的创建我们都进行了记录输出与命名。

### **四、ThreadPoolExecutor扩展**

>ThreadPoolExecutor扩展主要是围绕beforeExecute()、afterExecute()和terminated()三个接口实现的，

#### **1.beforeExecute：线程池中任务运行前执行**
#### **2.afterExecute：线程池中任务运行完毕后执行**
#### **3.terminated：线程池退出后执行**

>通过这三个接口我们可以监控每个任务的开始和结束时间，或者其他一些功能。
>下面我们可以通过代码实现一下

```java

public class ThreadPool {
    private static ExecutorService pool;
    public static void main( String[] args ) throws InterruptedException
    {
        //实现自定义接口
        pool = new ThreadPoolExecutor(2, 4, 1000, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<Runnable>(5),
                new ThreadFactory() {
            public Thread newThread(Runnable r) {
                System.out.println("线程"+r.hashCode()+"创建");
                //线程命名
                Thread th = new Thread(r,"threadPool"+r.hashCode());
                return th;
            }
        }, new ThreadPoolExecutor.CallerRunsPolicy()) {
    
            protected void beforeExecute(Thread t,Runnable r) {
                System.out.println("准备执行："+ ((ThreadTask)r).getTaskName());
            }
            
            protected void afterExecute(Runnable r,Throwable t) {
                System.out.println("执行完毕："+((ThreadTask)r).getTaskName());
            }
            
            protected void terminated() {
                System.out.println("线程池退出");
            }
        };
          
        for(int i=0;i<10;i++) {
            pool.execute(new ThreadTask("Task"+i));
        }    
        pool.shutdown();
    }
}

public class ThreadTask implements Runnable{    
    private String taskName;
    public String getTaskName() {
        return taskName;
    }
    public void setTaskName(String taskName) {
        this.taskName = taskName;
    }
    public ThreadTask(String name) {
        this.setTaskName(name);
    }
    public void run() {
        //输出执行线程的名称
        System.out.println("TaskName"+this.getTaskName()+"---ThreadName:"+Thread.currentThread().getName());
    }
}

```
> 我看下输出结果

 ```java
线程118352462创建
线程1550089733创建
准备执行：Task0
准备执行：Task1
TaskNameTask0---ThreadName:threadPool118352462
线程865113938创建
执行完毕：Task0
TaskNameTask1---ThreadName:threadPool1550089733
执行完毕：Task1
准备执行：Task3
TaskNameTask3---ThreadName:threadPool1550089733
执行完毕：Task3
准备执行：Task2
准备执行：Task4
TaskNameTask4---ThreadName:threadPool1550089733
执行完毕：Task4
准备执行：Task5
TaskNameTask5---ThreadName:threadPool1550089733
执行完毕：Task5
准备执行：Task6
TaskNameTask6---ThreadName:threadPool1550089733
执行完毕：Task6
准备执行：Task8
TaskNameTask8---ThreadName:threadPool1550089733
执行完毕：Task8
准备执行：Task9
TaskNameTask9---ThreadName:threadPool1550089733
准备执行：Task7
执行完毕：Task9
TaskNameTask2---ThreadName:threadPool118352462
TaskNameTask7---ThreadName:threadPool865113938
执行完毕：Task7
执行完毕：Task2
线程池退出

``` 
可以看到通过对beforeExecute()、afterExecute()和terminated()的实现，我们对线程池中线程的运行状态进行了监控，在其执行前后输出了相关打印信息。另外使用shutdown方法可以比较安全的关闭线程池， 当线程池调用该方法后，线程池中不再接受后续添加的任务。但是，此时线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出。

### **五、线程池线程数量**

线程吃线程数量的设置没有一个明确的指标，根据实际情况，只要不是设置的偏大和偏小都问题不大，结合下面这个公式即可

```java
   /**
	 * Nthreads=CPU数量
	 * Ucpu=目标CPU的使用率，0<=Ucpu<=1
	 * W/C=任务等待时间与任务计算时间的比率
	 */
	Nthreads = Ncpu*Ucpu*(1+W/C)
```
          

>以上就是对ThreadPoolExecutor类从构造函数、拒绝策略、自定义线程创建等方面介绍了其详细的使用方法，从而我们可以根据自己的需要，灵活配置和使用线程池创建线程，其中如有不足与不正确的地方还望指出与海涵。

[java线程池ThreadPoolExecutor类使用详解 - DaFanJoy - 博客园 (cnblogs.com)](https://www.cnblogs.com/dafanjoy/p/9729358.html)