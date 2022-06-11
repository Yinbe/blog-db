> 当一个对象已经不需要使用，本该被回收时，有另外一个正在使用的对象持有它的引用，导致它不能被回收，停留在堆内存中，这就产生了内存泄露。

内存泄露是造成应用程序oom的主要原因之一。android系统为每个应用程序都分配了有限的内存，而当一个应用产生的内存泄露多时，会导致应用所需要的内存超过了这个系统分配的内存限额，导致了oom的crash。

1、单例造成内存泄露 由于单例的静态特性使得单例的生命周期和应用的生命周期一样长。如果一个对象已经不需要使用，而单例还持有该对象的引用，导致该对象不能被正常回收，导致内存泄露。

> 当创建一个单例时，需要传入一个Context。如果传入的是activity的context。当这个context所对应的activity退出时，它的内存不会被回收，因为单例持有该activity的引用。所以正确的做法是传入application的context。两者的生命周期一样长。

2、非静态内部类创建实例造成内存泄露 在activity内部创建一个非静态内部类(线程等)，内部类持有外部类的隐式引用，导致activity不能被回收。正确的做法是将内部类设为静态内部类。

3、handler造成内存泄露

handler的使用造成的内存泄露问题比较常见,如：

```
public class MainActivity extends AppCompatActivity {
    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            //...
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        loadData();
    }
    private void loadData(){
        //...request
        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }
}
```

这种创建handler的方式会造成内存泄露，由于mHandler是Handler的匿名内部类的实例，所以它持有外部类的引用。消息队列是在一个Looper线程中不断轮询处理消息，如果activity退出时消息队列中还有未处理的消息或者正在处理的消息，则消息队列持有mHandler实例的引用，mHandler又持有activity的引用，导致activity无法被回收

正确的做法是：

```
public class MainActivity extends AppCompatActivity {
    private MyHandler mHandler = new MyHandler(this);
    private TextView mTextView ;
    private static class MyHandler extends Handler {
        private WeakReference<Context> reference;
        public MyHandler(Context context) {
            reference = new WeakReference<>(context);
        }
        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = (MainActivity) reference.get();
            if(activity != null){
                activity.mTextView.setText("");
            }
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = (TextView)findViewById(R.id.textview);
        loadData();
    }

    private void loadData() {
        //...request
        Message message = Message.obtain();
        mHandler.sendMessage(message);
    }
}
```

创建一个静态Handler内部类，然后duiHandler持有的对象使用弱引用。这样避免了activity泄露。不过Looper线程的消息队列中可能还有待处理的消息，所以在activity被销毁时 调用Hanlder的remove方法，清除消息队列。

> mHandler.removeCallbacksAndMessages(null);

4、线程造成的内存泄露

对于线程造成的内存泄露也是比较常见的：

```
//——————test1
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                SystemClock.sleep(10000);
                return null;
            }
        }.execute();
//——————test2
        new Thread(new Runnable() {
            @Override
            public void run() {
                SystemClock.sleep(10000);
            }
        }).start();
```

上面的异步任务和Runnable都是一个匿名内部类，它们对当前的activity都有一个隐式引用。正确的做法还是使用静态内部类的方式。

5、资源未关闭造成的内存泄露 对于使用了BroadcastReceiver,ContentObserver,File,Cursor,Stream,Bitmap等资源的使用，应该在activity销毁时及时关闭，否则这些资源不会被回收，造成内存泄露。

6、static关键字所导致的内存泄漏 static变量所指向的内存引用，如果不把它设置为null，GC是永远不会回收这个对象的

Tip:
1、对于生命周期比activity长的对象优先考虑applicationContext。

> 注意：Dialog除外。dialog对象需要挂载在window上，所以只能使用activity的context。

2、对于需要在静态内部类中使用非静态外部成员变量（如：context、view）,可以在静态内部类中使用弱引用来引用外部的变量避免内存泄露。

3、对于不再使用的对象，显式赋值为null,比如使用bitmap后先调用recycle(),再赋值为null。

4、保持对对象生命周期的敏感，特别是注意单例、静态对象、全局性集合的生命周期。
