BitmapFactory解析Bitmap的静态工厂方法常用的有：

> decodeFile、decodeResource、decodeStream

在不匹配options的情况下：
1、decodeFile、decodeStream在解析时不会对Bitmap进行一系列的屏幕适配，解析出来的将是原始大小的图片
2、decodeResource在解析时会对Bitmap根据当前设备屏幕像素密度densityDpi的值进行缩放适配，使得解析出来的Bitmap与当前的设备的分辨率匹配。

匹配options的情况下： 设置inJustDecodeBounds属性为true可以在解码返回一个null的bitmap，但可以获取outWidth，outHeight与outMimeType

1、按屏幕像素按比例调整图片大小，适当减少采样率inSampleSize 2、在非ui线程处理bitmap

使用asyncTask

```
class BitmapWorkerTask extends AsyncTask {
    private final WeakReference imageViewReference;
    private int data = 0;

    public BitmapWorkerTask(ImageView imageView) {
        // Use a WeakReference to ensure the ImageView can be garbage collected
        imageViewReference = new WeakReference(imageView);
    }

    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        data = params[0];
        return decodeSampledBitmapFromResource(getResources(), data, 100, 100));
    }

    // Once complete, see if ImageView is still around and set bitmap.
    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            if (imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
```

开始异步加载位图，只需要创建一个新的任务并执行它即可:

```
public void loadBitmap(int resId, ImageView imageView) {
    BitmapWorkerTask task = new BitmapWorkerTask(imageView);
    task.execute(resId);
}
```

3、使用(LruCache)内存缓存bitmap

```

	LruCache<String,Bitmap> mMemoryCache;
	public void mMemoryCache(){
	
		final int maxMemory=(int)(Runtime.getRuntime().maxMemory()/1024);
		final int cacheSize=maxMemory/8;
	
		mMemoryCache=new LruCache<String,Bitmap>(cacheSize){
	
			@Override
			protected int sizeOf(String key,Bitmap bitmap){
				return bitmap.getByteCount()/1024;
			}
		};
	
	}

	public void add BitmapToMemoryCache(String key,Bitmap bitmap){
		if(getBitmapFromMemCache(key)==null){
			mMemoryCache.put(key,bitmap);
		}
	}

	public Bitmap getBitmapFromMemCache(String key){
		return mMemoryCache.get(key);
	}
```

```
	public void loadBitmap(int resId,ImageView imageView){
		final String imageKey= String.valueOf(resId);
		final Bitmap bitmap=getBitmapFromMemCache(imageKey);
		if(bitmap!=null){
			mImageView.setImageBitmap(bitmap);
		}else{
			mImageView.setImageResource(R.drawable.bg);
			BitmapWorkerTask task=new BitmapWorkerTask(mImageView);
			task.execute(resId);
		}
	}
```

4、使用磁盘缓存Bitmap 内存缓存能提高最近访问过的Bitmap的速度，但无法保证Bitmap都能保存在缓存中。磁盘缓存可以用来保存那些已经处理过的Bitmap，减少从网络下载的次数。

```
private DiskLruCache mDiskLruCache;
private final Object mDiskCacheLock = new Object();
private boolean mDiskCacheStarting = true;
private static final int DISK_CACHE_SIZE = 1024 * 1024 * 10; // 10MB
private static final String DISK_CACHE_SUBDIR = "thumbnails";

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Initialize memory cache
    ...
    // Initialize disk cache on background thread
    File cacheDir = getDiskCacheDir(this, DISK_CACHE_SUBDIR);
    new InitDiskCacheTask().execute(cacheDir);
    ...
}

class InitDiskCacheTask extends AsyncTask<File, Void, Void> {
    @Override
    protected Void doInBackground(File... params) {
        synchronized (mDiskCacheLock) {
            File cacheDir = params[0];
            mDiskLruCache = DiskLruCache.open(cacheDir, DISK_CACHE_SIZE);
            mDiskCacheStarting = false; // Finished initialization
            mDiskCacheLock.notifyAll(); // Wake any waiting threads
        }
        return null;
    }
}

class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final String imageKey = String.valueOf(params[0]);

        // Check disk cache in background thread
        Bitmap bitmap = getBitmapFromDiskCache(imageKey);

        if (bitmap == null) { // Not found in disk cache
            // Process as normal
            final Bitmap bitmap = decodeSampledBitmapFromResource(
                    getResources(), params[0], 100, 100));
        }

        // Add final bitmap to caches
        addBitmapToCache(imageKey, bitmap);

        return bitmap;
    }
    ...
}

public void addBitmapToCache(String key, Bitmap bitmap) {
    // Add to memory cache as before
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }

    // Also add to disk cache
    synchronized (mDiskCacheLock) {
        if (mDiskLruCache != null && mDiskLruCache.get(key) == null) {
            mDiskLruCache.put(key, bitmap);
        }
    }
}

public Bitmap getBitmapFromDiskCache(String key) {
    synchronized (mDiskCacheLock) {
        // Wait while disk cache is started from background thread
        while (mDiskCacheStarting) {
            try {
                mDiskCacheLock.wait();
            } catch (InterruptedException e) {}
        }
        if (mDiskLruCache != null) {
            return mDiskLruCache.get(key);
        }
    }
    return null;
}

// Creates a unique subdirectory of the designated app cache directory. Tries to use external
// but if not mounted, falls back on internal storage.
public static File getDiskCacheDir(Context context, String uniqueName) {
    // Check if media is mounted or storage is built-in, if so, try and use external cache dir
    // otherwise use internal cache dir
    final String cachePath =
            Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) ||
                    !isExternalStorageRemovable() ? getExternalCacheDir(context).getPath() :
                            context.getCacheDir().getPath();

    return new File(cachePath + File.separator + uniqueName);
}
Copy
```

5、利用BitmapRegionDecoder实现显示高清大图片
如果不压缩按原尺寸加载原图，屏幕肯定不够大，并且考虑到内存的情况，不可能一次性整图加载到内存，所以使用BitmapRegionDecoder来实现局部加载。

```
BitmapRegionDecoder bitmapRegionDecoder=BitmapRegionDecoder.newInstance(inputStream,false);  
bitmapRegionDecoder=BitmapRegionDecoder.decodeRegion(rect,options);
```

rect指定大图片要显示的区域，options设置图片参数
