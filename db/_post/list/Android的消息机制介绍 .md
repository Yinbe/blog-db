```
Android是通过Looper、handler、message来实现消息机制的。Android不允许在子线程更新UI，所以Android的消息机制常用于更新UI。其实每个线程都可以有自己的消息队列和消息循环。  

从系统的HandlerThread说起
```

```java
public class HandlerThread extends Thread {
    int mPriority;
    int mTid = -1;
    Looper mLooper;

    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }
  

  
    protected void onLooperPrepared() {
    }

    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
  
    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }
        // If the thread has been started, wait until the looper has been created.
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

  
}
```

```
HandlerThread继承了Thread，在run方法中调用了Looper.prepare()和Looper.loop();此时HandlerThread就是有自己消息队列和消息循环的线程。  

接下来看看Looper.prepare()做了什么   
```

```
public final class Looper {
    private static final String TAG = "Looper";

    // sThreadLocal.get() will return null unless you've called prepare().
    static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
    private static Looper sMainLooper;  // guarded by Looper.class
	...
	//创建一个Looper，并存放在了sThreadLocal中
	private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
	 }
	 
	 public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
	 
	 public static Looper getMainLooper() {
        synchronized (Looper.class) {
            return sMainLooper;
        }
    }

    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                return;
            }

            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            msg.target.dispatchMessage(msg);
		...
            msg.recycleUnchecked();
        }
    }

    public static Looper myLooper() {
        return sThreadLocal.get();
    }

    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
	  
}
```

```
每一次调用Looper.prepare()，会先确认在该线程中是否有Looper存在，如果不存在，就创建一个Looper保存在ThreadLocal中，可以看到在Looper的构造方法中创建了MessageQueue。Looper.loop()开了个循环，会不断取出消息队列中的msg，并调用了msg.target.dispatchMessage(msg)。
这里需要介绍一下ThreadLocal 
```

```
public class ThreadLocal<T> {

    public T get() {
        // Optimized for the fast path.
        Thread currentThread = Thread.currentThread();
        Values values = values(currentThread);
        if (values != null) {
            Object[] table = values.table;
            int index = hash & values.mask;
            if (this.reference == table[index]) {
                return (T) table[index + 1];
            }
        } else {
            values = initializeValues(currentThread);
        }

        return (T) values.getAfterMiss(this);
    }

  
    public void set(T value) {
        Thread currentThread = Thread.currentThread();
        Values values = values(currentThread);
        if (values == null) {
            values = initializeValues(currentThread);
        }
        values.put(this, value);
    }

}
```

```
其实ThreadLocal类似于map，保存Looper时会保存对应当前线程作为key，这样也保证了在一个线程中只有一个Looper存在。


HandlerThread的经典应用是在IntentService中，我们都知道普通的service是运行在主线程中，而IntentService的主要实现方法onHandleIntent是运行在子线程中，而且在运行任务结束后IntentService会结束自己。IntentService是对Android消息机制应用很好的例子。  
我们来看看IntentService的关键代码
```

```
public abstract class IntentService extends Service {
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler;
    private String mName;
    private boolean mRedelivery;

    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            onHandleIntent((Intent)msg.obj);
            stopSelf(msg.arg1);
        }
    }

    ...

    @Override
    public void onCreate() {
        super.onCreate();
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

    @Override
    public void onStart(Intent intent, int startId) {
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }

    ...
    protected abstract void onHandleIntent(Intent intent);
}
```

```
可以看到IntentService在onCreate时start了一个HandlerThread并new了一个handler(),传入的Looper为HandlerThread线程的Looper。在onStart时创建了一个message，并通过sendMessage发送出去，ServiceHandler的handleMessage接收到消息并执行onHandleIntent方法，执行完结束自己。onHandleIntent就是在HandlerThread线程中被调用的。   

来看看handler的代码
```

```
public class Handler {
    ...
    public void handleMessage(Message msg) {
    }
  
    /**
     * Handle system messages here.
     */
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }

    public Handler(Callback callback, boolean async) {
		...

        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }

  
    public Handler(Looper looper, Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
  
    public final boolean sendMessage(Message msg)
    {
        return sendMessageDelayed(msg, 0);
    }

  
    public final boolean postDelayed(Runnable r, long delayMillis)
    {
        return sendMessageDelayed(getPostMessage(r), delayMillis);
    }
  
    ...
    public final boolean sendMessageAtFrontOfQueue(Message msg) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, 0);
    }

    private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
        msg.target = this;
        if (mAsynchronous) {
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }
   
    public final Looper getLooper() {
        return mLooper;
    }

    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }

    private static void handleCallback(Message message) {
        message.callback.run();
    }
    
}
```

```
我们将HandlerThread线程的Looper传入了handler中，这样就能拿到该线程Looper的消息队列。再来看看IntentService中消息是如何被发送出去的，是调用了handler的sendMessage方法，这个方法最终调到enqueueMessage方法，这个方法都做了什么，msg.target = this;将自身赋值给msg.target，然后把msg插入messageQueue中。插入的消息在Looper的loop方法中被取出，取出msg后执行了msg.target.dispatchMessage(msg)，即调用了handler的dispatchMessage方法，这里是消息切换到对应线程被处理的真正地方，dispatchMessage方法对消息的不同来源进行了分发处理。   
由handler的源码可知msg.callback是Handler的post()的Runnable;
mCallback是Handler的回调接口Handler.Callback;而handleMessage这个就是我们常用的继承Handler的实现方法。IntentService中就是调到了这个方法。  
以上就是以IntentService为例的Android消息机制的执行过程了。

接下来说一下系统是如何使用消息机制来将数据更新到UI线程的   
```

```
public final class ActivityThread {
  
    public static void main(String[] args) {
		...
        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
}
```

```
其实系统在Android的入口ActivityThread类的main方法中，就已经做了Looper的初始化工作Looper.prepareMainLooper();这个方法主要做了两件事，在主线程创建looper对象并给静态变量sMainLooper引用，这样我们就可以全局获取主线程的looper了。  
接下来就只要在创建handler时传入主线程的looper，实现处理结果的方法(handleMessage等)。把其他线程处理结果数据保存到msg中，通过sendMessage把msg插入到主线程的messageQueue中，然后等待loop调度就行了。  
我们在Activity创建handler时并没有传入looper，那是因为当一个handler被new的时候，如果没有传入Looper,则会调用Looper.myLooper();这个方法会返回本线程的Looper对象。如果需要在子线程创建handler并更新主线程，则要主动传入主线程的looper了。

```
