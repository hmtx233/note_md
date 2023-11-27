#同步代码 #2023-10-27
# synchronized使用,同步代码块

## synchronized(obj) 静态obj

```java
  
public class ThreadTest {  
    public static void main(String args[]) {  
        MyThread thread1 = new MyThread();  
        MyThread thread2 = new MyThread();  
        thread1.start();  
        thread2.start();  
  
    }  
  
}  
  
  
class MyThread extends Thread {  
    String flag;  
    MyThread() {  
    }  
  
    MyThread(String flag) {  
        this.flag = flag;  
    }  
  
    static Object lock = new Object();  
  
    @Override  
    public void run() {  
        //隐私锁 jvm自己会加锁解锁 synchronized是同步阻塞的
        synchronized (lock) {  
            try {  
                for (int i = 0; i < 10; i++) {  
                    System.out.println(Thread.currentThread().getName() + ":" + i);  
                    sleep(1000);  
                }  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
        }  
    }  
}
# 打印结果
Thread-0:0
Thread-0:1
Thread-0:2
Thread-0:3
Thread-0:4
Thread-0:5
Thread-0:6
Thread-0:7
Thread-0:8
Thread-0:9
Thread-1:0
Thread-1:1
Thread-1:2
Thread-1:3
Thread-1:4
Thread-1:5
Thread-1:6
Thread-1:7
Thread-1:8
Thread-1:9
```

## synchronized(String)  不同的String值

```java
  
public class ThreadTest {  
    public static void main(String args[]) {  
        MyThread thread1 = new MyThread("123");  
        MyThread thread2 = new MyThread("456");  
        thread1.start();  
        thread2.start();  
  
    }  
  
}  
  
  
class MyThread extends Thread {  
    String flag;  
  
    MyThread() {  
    }  
  
    MyThread(String flag) {  
        this.flag = flag;  
    }  
  
  
  
    @Override  
    public void run() {  
        synchronized (flag) {  
            try {  
                for (int i = 0; i < 10; i++) {  
                    System.out.println(Thread.currentThread().getName() + ":" + i);  
                    sleep(1000);  
                }  
            } catch (Exception e) {  
                e.printStackTrace();  
            }  
        }  
    }  
}
Thread-1:0
Thread-0:0
Thread-1:1
Thread-0:1
Thread-0:2
Thread-1:2
Thread-0:3
Thread-1:3
Thread-1:4
Thread-0:4
Thread-1:5
Thread-0:5
Thread-1:6
Thread-0:6
Thread-0:7
Thread-1:7
Thread-0:8
Thread-1:8
Thread-1:9
Thread-0:9
```
# lock同步

```java
import java.util.concurrent.locks.Lock;  
import java.util.concurrent.locks.ReentrantLock;  
  
public class ThreadTest {  
    public static void main(String args[]) {  
        MyThread thread1 = new MyThread();  
        MyThread thread2 = new MyThread();  
        thread1.start();  
        thread2.start();  
  
    }  
  
}  
  
  
class MyThread extends Thread {  
    String flag;  
  
    MyThread() {  
    }  
  
    MyThread(String flag) {  
        this.flag = flag;  
    }  
  
    static Lock lock = new ReentrantLock();  
  
    @Override  
    public void run() {  
        lock.lock();  
        try {  
            for (int i = 0; i < 10; i++) {  
                System.out.println(Thread.currentThread().getName() + ":" + i);  
                sleep(1000);  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
        } finally { 
            //必须解锁 
            lock.unlock();  
        }  
    }  
}
# 打印结果
Thread-1:0
Thread-1:1
Thread-1:2
Thread-1:3
Thread-1:4
Thread-1:5
Thread-1:6
Thread-1:7
Thread-1:8
Thread-1:9
Thread-0:0
Thread-0:1
Thread-0:2
Thread-0:3
Thread-0:4
Thread-0:5
Thread-0:6
Thread-0:7
Thread-0:8
Thread-0:9
```

1. 当使用锁或者 synchronized关键字时，通常要用static修饰一下 才会有序和同步
2. lock.lock()在try层外，如果放在内层，未加锁成功就执行了释放锁的操作，从而导致了新的异常；
```
    static Lock lock = new ReentrantLock();  
	lock.lock();
    try{
    
    }catch(Exception e){
       doSomething();
    }finally{
	    lock.unlock()
    }

   static Object lock = new Object();  
    synchronized(lock){
       doSomething();
    }

```
