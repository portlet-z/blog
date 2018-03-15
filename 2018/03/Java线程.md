## Java线程

#### 创建线程的方式及实现

* 继承Thread类创建线程。Thread是实现了Runnable接口的一个实例。通过start方法启动一个新的线程，并执行run方法

* 实现Runnable接口创建线程

* 实现Callable接口，通过FutureTask包装器来创建Thread线程

  ```java
  public interface Callable<V> {
      V call() throws Exception;
  }

  public class SomeCallable<V> extends OtherClass implement Callable<V> {
      @Override
      public V call() throws Exception{
          // TODO
          return null;
      }
  }

  Callable<V> oneCallable = new SomeCallable<V>();
  //由Callable<Integer>创建一个FutureTask<Integer>对象
  FutureTask<V> oneTask = new FutureTask<V>(oneCallable);
  Thread oneThread = new Thread(oneTask);
  oneThread.start();
  ```



####  sleep(),wait(),join(),yield()有什么区别

* sleep()
  - Thread类的静态方法，也是native方法
  - 需要捕获InterruptedException异常
  - 让当前正在执行的线程在指定的时间内暂停执行，进入阻塞状态
* wait()
  - Object类中的方法
  - 会释放对象的"锁标志"
  - 会使当前线程暂停执行，并将当前线程放入对象等待池中，直到调用了notify()方法后，将从对象等待池中移出任意一个线程并放入锁标志等待池中，并准备争夺锁的拥有权
  - 只能在synchronized语句中使用
* yield()
  - Thread类的静态方法
  - 不泡异常
  - 使当前线程"让步"，重新回到就绪(Runnable)状态
  - 只能使同优先级或者高优先级的线程得到执行机会
* join()
  - Thread对象的实例方法
  - 会使当前线程等待调用join()方法的线程结束后才能继续执行



#### CountDownLatch原理



