Android的程序运行在多线程的环境中，为了更方便的处理子线程和UI线程的交互，引入了AsyncTask。

> AsnyncTask内部实现简单概括为 线程池 + Handler

AysncTask的定义为:

> Params 参数类型；
> Progress 进度类型；
> Result 返回结果类型；

`public abstract class AsyncTask`<`Params,Progress,Result`>`{}<span> </span>`

* AsyncTask 主要的方法
  1、 execute(Params... params) 触发异步任务执行
  2、onPreExecute() 执行在UI线程，当任务执行之前开始调用此方法
  3、doInBackground(Params... params) 在onPreExecute()完成后执行，比较耗时的操作都可以放在这里。注意这里不能直接操作UI。此方法在后台线程执行，在执行过程中可以调用publicProgress(Progress…)来更新任务的进度。
  4、 onCancelled() 用户调用取消时，要做的操作
* 要注意的地方
  1、AsyncTask的实例必须在UI线程中创建
  2、 execute(Params... params) 必须在UI线程中调用
  3、doInBackground(Params... params)不能直接操作UI
  4、一个任务AsyncTask任务只能被执行一次

注意点解释：
1、AsyncTask的实例必须在UI线程中创建

```
 private static Handler getHandler() {
        synchronized (AsyncTask.class) {
            if (sHandler == null) {
                sHandler = new InternalHandler();
            }
            return sHandler;
        }
  }
  
 private static class InternalHandler extends Handler {
        public InternalHandler() {
            super(Looper.getMainLooper());
        }

        ...
 }
```

因为Handler使用的是UI线程的Looper

2、 execute(Params... params) 必须在UI线程中调用

```
	@MainThread
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }
```

同1，注解@MainThread ，且onPreExecute() 在里面调用

3、doInBackground(Params... params)不能直接操作UI

```
public AsyncTask() {
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);

                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked
                return postResult(doInBackground(mParams));
            }
        };

        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occured while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }
```

doInBackground(Params... params)运行在子线程中

4、一个任务AsyncTask任务只能被执行一次。

```
public enum Status {
        /**
         * Indicates that the task has not been executed yet.
         */
        PENDING,
        /**
         * Indicates that the task is running.
         */
        RUNNING,
        /**
         * Indicates that {@link AsyncTask#onPostExecute} has finished.
         */
        FINISHED,
    }

	@MainThread
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

       ...
    }
```

execute 调用时如果状态为 mStatus != Status.PENDING 则抛异常

5、AsyncTask的局限性和进阶使用

```
	private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final int KEEP_ALIVE = 1;
    private static final BlockingQueue<Runnable> sPoolWorkQueue =
            new LinkedBlockingQueue<Runnable>(128);

    /**
     * An {@link Executor} that can be used to execute tasks in parallel.
     */
    public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
Copy
```

AsyncTask的内部Handler和ThreadPoolExecutor都是static的，这么定义的变量属于类的，是进程范围内共享的，所以AsyncTask控制着进程范围内所有的子类实例，而且该类的所有实例都共用一个线程池和Handler。2.3以前的版本默认的最大并发只有5个，如果你的应用需要大量的后台线程去执行任务，那么你只能放弃使用AsyncTask。 3.0之后如果使用execute()提交的任务，默认使用序列线程池，即一次只执行一个线程任务。还新增了方法executeOnExecutor()。

> 允许开发者指定AsyncTask的运行模式
>
>> AsyncTask.SERIAL_EXECUTOR 使用序列线程池 AsyncTask.THREAD_POOL_EXECUTOR 使用默认线程池
>>

> 允许开发者自定义线程池
>
>> 默认线程池默认的最大并发数为CPU_COUNT * 2 + 1，队列最大128个，开发者可以自定义ThreadPoolExecutor，通过传入参数改变这些限制；
>>
>
