## Thread 怎么用？
- new Thread().run()  不常用
- ExecutorService 线程池，这个常用

## ExecutorService 有哪几种？
- Executors.newSingleThreadExecutor() 单个线程的线程池，有任务的话就排队。
- Executors.newFixedThreadPool(10); 固定线程的线程池，如果没有线程可用的话，有任务就排队，并且线程用完不会回收。
- Executors.newCachedThreadPool(); 用的最多的，带缓存的线程池，有个核心线程数是不会被回收的，其他的用完后会被缓存住，再没有人用就回收。

## 在一个线程中怎么关闭另外一个线程？
- Thread.stop() 强制性的
- Thread.interrupt():温和式终结:不立即、不强制。需要代码配合的，interrupt()和isInterrupt()配合。
- 关于InterruptedException? 调用Thread.sleep();就会出现，这个是如果在sleep期间，调用了Thread.interrupt()方法的话就会抛出这个异常，因为每必要等了啊，大兄弟。

## 线程通讯的本质是啥？Synchronized加锁到底是锁了个什么？
### Synchronized monitor
- 本质：保护方法内部或者代码块内部资源（数据）的互斥访问，即同一时间，由同一个Montior监视的代码最多只允许一个线程访问。
- 这个关键字是强行把一系列操作变成原子性操作（即CPU一定不会在这一段操作中把线程切走）。Synchronized保护的是数据。
- 如果写在了方法上，那么Synchronized的monitor就是this,静态方法就是这个class，如果是代码块的话，就可以自己指定。
- 一个线程进入了Synchronized方法，那么他就会暂时拿到monitor，只有拿到了monitor才可以执行这个方法，如果没有monitor那就必须要排队拿monitor。
- 在一个拿到monitor的线程内调用wait()方法，可以让这个兄弟暂时让出monitor, 其他线程调用notify()或者notifyAll()方法就可以再次激活这个线程，这个线程会从wait()调用的地方开始执行，但是！！如果此时monitor有人拿了他还是要去排队。

### 死锁
- 上代码！
```java
private final Object lock1 = new Object();
    private final Object lock2 = new Object();

    public void test() {
        synchronized (lock1) {
            // 干事
            // 他拿到lock1后需要拿到lock2才能把方法走完并且释放monitor
            // 但是如果这个时候lock2被别人拿了，自己拿不到，同时那个人
            // 需要拿到lock1才能结束，于是这两个傻子就一起等
            // 就死锁了
            synchronized (lock2) {
                // 干完这个事后就结束
            }
        }
    }

    public void test2() {
        synchronized (lock2) {
            // 干事
            synchronized (lock1) {
                // 干完这个事后就结束
            }
        }
    }
```

### 读写锁
- 用的少，用于更精细的访问


## Android的多线程应用
### Hanler Looper
- 就是里面有一个不断循环的线程, 如果调用了post方法，就执行任务，差不多就是这样。
- Handler包含了Looper，Looper是负责循环的，Handler是负责任务的定制和线程间的传递的。
- Looper原理伪代码
```java
class Looper{
    var task: Runnable? = null
    
    fun post(task: Runnable) {
        this.task = task
    }
    
    fun run() {
        Thread{
            while (true) {
               if (task != null) {
                   task!!.run()
                   task = null
               }
            }
        }.run()
    }
}
```
### AsyncTask
- 内存泄漏？是有可能的，因为AsyncTask会持有外部Activity的引用。具体的原因是因为执行中的线程（GC ROOT）不会被线程回收。
- GC ROOT 的概念。1:运行中的线程。2：静态对象。3: 来自native code中的引用。
- 所以其实只要是线程，就可能会造成内存泄漏，如果AsyncTask的任务时间不长，其实可以不做内存泄漏的处理的。
- 弱引用防范,下面是伪代码。
```java
static class loadResAsyncTask extends AsyncTask<Void, Void, xxxx>> {
    private WeakReference<PuzzleActivity> activityReference;
}
```
### Executor
- 如果在界面组建中创建Executor，记得要关闭
```java

protected void onDestroy() { 
    super.onDestroy(); 
    executor.shutdown();
}
```

### Executor AsyncTask Handler IntentService怎么选择
- 能用Executor 就用 Executor
- 如果需要 后台线程推送任务到Ui线程，考虑AsyncTask Handler
- IntentService, 处理线程起来没啥优势，方面拿上下文。



