#### LruCache源码分析

LruCache的源码分析已经很多了，看了很多遍，但是自己走一遍分析，才是真正的掌握，将知识转化到自身。

#### 用途

LruCache的用途就是缓存，但是缓存并不能一直无限量的缓存，设备内存始终是有限的，所以缓存需要有一个删除策略。

#### 常见例子

图片加载基本是每个App的必备项了，虽然我们使用都是第三方加载框架，但是加载框架内部也会缓存图片Bitmap对象，也是使用LruCache来缓存，所以我们了解LruCache，对加载框架的缓存逻辑也能知道核心了。

- 启动一个AsyncTask任务，OkHttp加载图片Bitmap，从网络获取前先从LruCache中查询是否有缓存，有则直接返回，无则进行网络请求，请求完毕再保存到LruCache缓存中，下次再去获取则会命中缓存，复用缓存而无需请求。

1. 创建图片缓存，缓存大小为设备最大内存容量的8分之一。创建LruCache，我们一般需要复写sizeOf()，LruCache会回调sizeOf()来获取缓存对象占用的内存大小。

还有2个可选复写的方法：
	- entryRemoved()为缓存对象被删除时的回调。
	- create()为获取不到缓存时调用，我们可以新建缓存对象，但在图片缓存中我们并不需要，返回null或者不复写即可（默认就是返回null），请求完网络后再保存，或者在这里请求再返回，但是不建议，毕竟每个缓存对象的获取方式不同。

```
/**
 * 创建图片缓存
 */
private void createImageCache() {
    //取设备内存最大容量的8分之一作为缓存大小
    long maxMemorySize = Runtime.getRuntime().maxMemory();
    int cacheSize = (int) (maxMemorySize / 8);
    mLruCache = new LruCache<String, Bitmap>(cacheSize) {
        @Override
        protected int sizeOf(String key, Bitmap value) {
            //计算Bitmap占用的内存
            return getBitmapSize(value);
        }

        @Override
        protected void entryRemoved(boolean evicted, String key, Bitmap oldValue, Bitmap newValue) {
            super.entryRemoved(evicted, key, oldValue, newValue);
            Log.d(TAG, "缓存对象被删除");
        }

        @Override
        protected Bitmap create(String key) {
            //get()方法获取不到缓存时调用，我们不返回数据，让去请求网络获取
            return super.create(key);
        }
    };
}
```

2. 新建AsyncTask异步任务，doInBackground()中，先查询缓存是否存在，不存在再让请求网络，再保存到网络，如果缓存存在，则直接复用缓存。

```
/**
 * 兼容获取Bitmap大小
 */
private int getBitmapSize(Bitmap bitmap) {
    //API 19
    if (Build.VERSION.SDK_INT == Build.VERSION_CODES.KITKAT) {
        return bitmap.getAllocationByteCount();
    }
    //API 12
    if (Build.VERSION.SDK_INT == Build.VERSION_CODES.HONEYCOMB_MR1) {
        return bitmap.getByteCount();
    }
    // Earlier Version
    return bitmap.getRowBytes() * bitmap.getHeight();
}

private static class DownloadImageTask extends AsyncTask<String, Void, Bitmap> {
    private OkHttpClient mClient;
    private Callback mCallback;

    /**
     * 回调接口
     */
    public interface Callback {
        /**
         * 执行前回调
         */
        void onStart();

        /**
         * 执行后回调
         *
         * @param bitmap 执行结果
         */
        void onFinish(Bitmap bitmap);

        /**
         * 尝试获取缓存
         */
        Bitmap getCache(String key);
    }

    public DownloadImageTask(OkHttpClient client, Callback callback) {
        mClient = client;
        mCallback = callback;
    }

    @Override
    protected void onPreExecute() {
        super.onPreExecute();
        mCallback.onStart();
    }

    @Override
    protected Bitmap doInBackground(String... urls) {
        try {
            String url = urls[0];
            //1.先从内存缓存中找
            Bitmap cacheBitmap = mCallback.getCache(url);
            if (cacheBitmap != null) {
                Log.d(TAG, "缓存命中");
                return cacheBitmap;
            }
            Log.d(TAG, "缓存不命中，请求网络");
            //2.缓存找不到，请求网络
            Request request = new Request.Builder()
                    .url(url)
                    .build();
            Call call = mClient.newCall(request);
            Response response = call.execute();
            return BitmapFactory.decodeStream(response.body().byteStream());
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    protected void onPostExecute(Bitmap bitmap) {
        super.onPostExecute(bitmap);
        mCallback.onFinish(bitmap);
    }
}
```

#### 原理

LruCache的缓存是使用的最近最少使用的策略，当访问元素时，将元素移动到表尾，当缓存容量到达最大值时移除表头元素（最少使用的元素）。

使用Key-Value的方式缓存数据自然想到Map这种键值对的数据结构，而LruCache的策略则使用了LinkedHashMap。

为什么使用LinkedHashMap呢？因为LinkedHashMap的构造方法，有一个accessOrder参数，默认为false，则为按插入顺序排序，true则使用访问顺序排序，访问越多，越排得后。使用LinkedHashMap则天然支持最近最少使用的策略。


#### 构造方法。

- 指定最大容量为参数创建。不允许配置小于等于0.
- 并创建了一个LinkedHashMap，指定最后的accessOrder字段为true，则代表按访问顺序排序，将经常访问的元素放到表尾。

```
/**
 * @param maxSize 最大缓存容量
 */
public LruCache(int maxSize) {
    //最大缓存的对象大小，不能小于等于0，否则抛出异常
    if (maxSize <= 0) {
        throw new IllegalArgumentException("maxSize <= 0");
    }
    this.maxSize = maxSize;
    //缓存映射，最后的accessOrder字段设置为true，则按照访问的顺序排序，否则以插入的顺序排序
    this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
}
```

#### 添加缓存元素。
- key、value都不能为null，否则抛出异常。
- 同步锁，处理并发，safeSizeOf()就是调用到我们的sizeOf()获取缓存对象的占用大小。并将占用大小和已占用大小累加。
- 通过map.put()添加缓存元素。put()方法有一个返回值，意思是如果之前已经有key保存了，则将之前保存的value返回，再覆盖上新的value。
- 如果put()有返回值，则将他的占用大小和累加值相减。
- 由于删除了旧值，所以回调entryRemoved()。
- put()操作改变了缓存，调用trimToSize()调整内存。

```
/**
 * 添加缓存
 *
 * @param key   缓存Key
 * @param value 缓存值
 */
public final V put(K key, V value) {
    //不能存储null键和null值
    if (key == null || value == null) {
        throw new NullPointerException("key == null || value == null");
    }
    //以前的值，只有当map里面已经记录了key，通过put方法就会返回之前缓存的value
    V previous;
    synchronized (this) {
        //插入的数量自增
        putCount++;
        //将已缓存的对象大小和本次添加的缓存对象的大小相加
        size += safeSizeOf(key, value);
        //添加缓存
        previous = map.put(key, value);
        //之前添加过，将大小减回去
        if (previous != null) {
            size -= safeSizeOf(key, previous);
        }
    }
    //由于本次缓存会覆盖之前的值，所以相当于删除之前的值，回调entryRemoved通知元素被删除
    if (previous != null) {
        entryRemoved(false, key, previous, value);
    }
    //每次调整缓存，都去判断如果缓存满了，按照LRU最近最少使用策略删除缓存
    trimToSize(maxSize);
    return previous;
}
```

####  修整缓存占用
- 开启一个死循环，不断检查当前占用内存是否大于最大值，否则一直循环删除最近最少使用的对象，直到内存占用小于最大值才停止。
- map遍历，找出表头元素。
- 通过map.remove()移除最近最少使用的元素。
- 由于删除了元素，调用safeSizeOf()调整内存计数。
- 回调entryRemoved()提示删除了元素。

```
/**
 * 整理缓存，如果内存超出，删除表头元素
 */
private void trimToSize(int maxSize) {
    //开启死循环
    while (true) {
        K key;
        V value;
        synchronized (this) {
            //参数检查
            if (size < 0 || (map.isEmpty() && size != 0)) {
                throw new IllegalStateException(getClass().getName()
                        + ".sizeOf() is reporting inconsistent results!");
            }
            //一直死循环，直到缓存的对象内存小于最大值
            if (size <= maxSize) {
                break;
            }
            //不断获取第一个元素，不断while循环删除，直到内存大小比最大内存大小小才停下
            Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
            //如果为null不处理
            if (toEvict == null) {
                break;
            }
            //获取本次要删除的元素键值
            key = toEvict.getKey();
            value = toEvict.getValue();
            //移除元素
            map.remove(key);
            //将删除的元素的内存减去
            size -= safeSizeOf(key, value);
            //回收次数自增
            evictionCount++;
        }
        //调用entryRemoved()提醒删除了元素
        entryRemoved(true, key, value, null);
    }
}
```

#### 获取缓存元素

- 不允许key为null，否则抛出异常。
- 同步锁包保证并发获取。
- 通过map.get(key)获取缓存元素。
- 获取不到缓存元素，则调用create(key)来新建元素。
- create()获取新建的元素不为null，则将元素保存到map，处理就和上面的put()处理流程是一致的。
- 操作完毕，调用trimToSize()，检查是否需要清理内存。

```
/**
 * 获取缓存，会将本次获取的元素放到表尾
 */
public final V get(K key) {
    //不允许key为null
    if (key == null) {
        throw new NullPointerException("key == null");
    }

    V mapValue;
    synchronized (this) {
        //从map中获取value，由于我们设置了accessOrder为true，所以会将元素移动到表尾
        mapValue = map.get(key);
        if (mapValue != null) {
            hitCount++;
            return mapValue;
        }
        missCount++;
    }

    //map中获取不到元素，调用create()方法创建一个
    V createdValue = create(key);
    if (createdValue == null) {
        return null;
    }
    //将对象缓存到map和put()方法是一样的
    synchronized (this) {
        createCount++;
        //put添加缓存，但是如果返回了旧值，则将旧值保存回去（意外）
        mapValue = map.put(key, createdValue);
        if (mapValue != null) {
            //保存原来的值
            map.put(key, mapValue);
        } else {
            size += safeSizeOf(key, createdValue);
        }
    }
    //有旧值，从map中删除掉
    if (mapValue != null) {
        entryRemoved(false, key, createdValue, mapValue);
        return mapValue;
    } else {
        //每次调整缓存，都去判断如果缓存满了，按照LRU最近最少使用策略删除缓存
        trimToSize(maxSize);
        return createdValue;
    }
}
```

#### 移除缓存元素

- 同样key不能为null，否则抛出异常。
- 通过map.remove(key)移除元素
- 调用safeSizeOf()计算内存占用大小。
- 调用entryRemoved()通知缓存元素被删除。

```
/**
 * 移除元素
 *
 * @return 元素Key
 */
public final V remove(K key) {
    //不允许key为null
    if (key == null) {
        throw new NullPointerException("key == null");
    }
    //移除的元素
    V previous;
    synchronized (this) {
        //从map中移除元素
        previous = map.remove(key);
        //改变缓存的大小
        if (previous != null) {
            size -= safeSizeOf(key, previous);
        }
    }
    //回调entryRemoved告知元素被删除
    if (previous != null) {
        entryRemoved(false, key, previous, null);
    }
    return previous;
}
```

#### 完整源码和注释

```
public class LruCache<K, V> {
    private final LinkedHashMap<K, V> map;

    /**
     * 当前缓存的对象的总内存
     */
    private int size;
    /**
     * 配置的最大缓存容量
     */
    private int maxSize;

    /**
     * 调用put()方法缓存对象的次数
     */
    private int putCount;
    /**
     * 调用create()方法创建对象的次数
     */
    private int createCount;
    /**
     * 调用trimToSize()方法回收对象的次数
     */
    private int evictionCount;
    /**
     * 取调用get()命中缓存的次数
     */
    private int hitCount;
    /**
     * 调用get()方法不命中的次数
     */
    private int missCount;

    /**
     * @param maxSize 最大缓存容量
     */
    public LruCache(int maxSize) {
        //最大缓存的对象大小，不能小于等于0，否则抛出异常
        if (maxSize <= 0) {
            throw new IllegalArgumentException("maxSize <= 0");
        }
        this.maxSize = maxSize;
        //缓存映射，最后的accessOrder字段设置为true，则按照访问的顺序排序，否则以插入的顺序排序
        this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
    }

    /**
     * 重新设置最大缓存大小
     *
     * @param maxSize 新的缓存大小
     */
    public void resize(int maxSize) {
        if (maxSize <= 0) {
            throw new IllegalArgumentException("maxSize <= 0");
        }
        //同步锁同步设置最大值
        synchronized (this) {
            this.maxSize = maxSize;
        }
        trimToSize(maxSize);
    }

    /**
     * 获取缓存，会将本次获取的元素放到表尾
     */
    public final V get(K key) {
        //不允许key为null
        if (key == null) {
            throw new NullPointerException("key == null");
        }

        V mapValue;
        synchronized (this) {
            //从map中获取value，由于我们设置了accessOrder为true，所以会将元素移动到表尾
            mapValue = map.get(key);
            if (mapValue != null) {
                hitCount++;
                return mapValue;
            }
            missCount++;
        }

        //map中获取不到元素，调用create()方法创建一个
        V createdValue = create(key);
        if (createdValue == null) {
            return null;
        }
        //将对象缓存到map和put()方法是一样的
        synchronized (this) {
            createCount++;
            //put添加缓存，但是如果返回了旧值，则将旧值保存回去（意外）
            mapValue = map.put(key, createdValue);
            if (mapValue != null) {
                //保存原来的值
                map.put(key, mapValue);
            } else {
                size += safeSizeOf(key, createdValue);
            }
        }
        //有旧值，从map中删除掉
        if (mapValue != null) {
            entryRemoved(false, key, createdValue, mapValue);
            return mapValue;
        } else {
            //每次调整缓存，都去判断如果缓存满了，按照LRU最近最少使用策略删除缓存
            trimToSize(maxSize);
            return createdValue;
        }
    }

    /**
     * 添加缓存
     *
     * @param key   缓存Key
     * @param value 缓存值
     */
    public final V put(K key, V value) {
        //不能存储null键和null值
        if (key == null || value == null) {
            throw new NullPointerException("key == null || value == null");
        }
        //以前的值，只有当map里面已经记录了key，通过put方法就会返回之前缓存的value
        V previous;
        synchronized (this) {
            //插入的数量自增
            putCount++;
            //将已缓存的对象大小和本次添加的缓存对象的大小相加
            size += safeSizeOf(key, value);
            //添加缓存
            previous = map.put(key, value);
            //之前添加过，将大小减回去
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }
        //由于本次缓存会覆盖之前的值，所以相当于删除之前的值，回调entryRemoved通知元素被删除
        if (previous != null) {
            entryRemoved(false, key, previous, value);
        }
        //每次调整缓存，都去判断如果缓存满了，按照LRU最近最少使用策略删除缓存
        trimToSize(maxSize);
        return previous;
    }

    /**
     * 整理缓存，如果内存超出，删除表头元素
     */
    private void trimToSize(int maxSize) {
        //开启死循环
        while (true) {
            K key;
            V value;
            synchronized (this) {
                //参数检查
                if (size < 0 || (map.isEmpty() && size != 0)) {
                    throw new IllegalStateException(getClass().getName()
                            + ".sizeOf() is reporting inconsistent results!");
                }
                //一直死循环，直到缓存的对象内存小于最大值
                if (size <= maxSize) {
                    break;
                }
                //不断获取第一个元素，不断while循环删除，直到内存大小比最大内存大小小才停下
                Map.Entry<K, V> toEvict = map.entrySet().iterator().next();
                //如果为null不处理
                if (toEvict == null) {
                    break;
                }
                //获取本次要删除的元素键值
                key = toEvict.getKey();
                value = toEvict.getValue();
                //移除元素
                map.remove(key);
                //将删除的元素的内存减去
                size -= safeSizeOf(key, value);
                //回收次数自增
                evictionCount++;
            }
            //调用entryRemoved()提醒删除了元素
            entryRemoved(true, key, value, null);
        }
    }

    /**
     * 移除元素
     *
     * @return 元素Key
     */
    public final V remove(K key) {
        //不允许key为null
        if (key == null) {
            throw new NullPointerException("key == null");
        }
        //移除的元素
        V previous;
        synchronized (this) {
            //从map中移除元素
            previous = map.remove(key);
            //改变缓存的大小
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }
        //回调entryRemoved告知元素被删除
        if (previous != null) {
            entryRemoved(false, key, previous, null);
        }
        return previous;
    }

    /**
     * 告知元素被删除，我们可以复写这个方法
     */
    protected void entryRemoved(boolean evicted, K key, V oldValue, V newValue) {
    }

    /**
     * 获取不到元素，告知我们创建一个
     */
    protected V create(K key) {
        return null;
    }

    /**
     * 实际就是调用了sizeOf()方法
     */
    private int safeSizeOf(K key, V value) {
        int result = sizeOf(key, value);
        if (result < 0) {
            throw new IllegalStateException("Negative size: " + key + "=" + value);
        }
        return result;
    }

    /**
     * 返回对象的内存占用大小，默认为，一般我们都会重写
     */
    protected int sizeOf(K key, V value) {
        return 1;
    }

    /**
     * 清除所有缓存
     */
    public final void evictAll() {
        //-1代表清除所有元素
        trimToSize(-1);
    }

    /**
     * 获取已缓存的对象总大小
     */
    public synchronized final int size() {
        return size;
    }

    /**
     * 获取配置的对象缓存最大大小
     */
    public synchronized final int maxSize() {
        return maxSize;
    }

    /**
     * 获取调用get()命中缓存的次数
     */
    public synchronized final int hitCount() {
        return hitCount;
    }

    /**
     * 获取调用get()方法不命中的次数
     */
    public synchronized final int missCount() {
        return missCount;
    }

    /**
     * 获取调用create()方法创建对象的次数
     */
    public synchronized final int createCount() {
        return createCount;
    }

    /**
     * 获取调用put()方法缓存对象的次数
     */
    public synchronized final int putCount() {
        return putCount;
    }

    /**
     * 获取调用trimToSize()方法回收对象的次数
     */
    public synchronized final int evictionCount() {
        return evictionCount;
    }

    /**
     * 获取缓存map的一个副本
     */
    public synchronized final Map<K, V> snapshot() {
        return new LinkedHashMap<K, V>(map);
    }

    @Override
    public synchronized final String toString() {
        int accesses = hitCount + missCount;
        int hitPercent = accesses != 0 ? (100 * hitCount / accesses) : 0;
        return String.format("LruCache[maxSize=%d,hits=%d,misses=%d,hitRate=%d%%]",
                maxSize, hitCount, missCount, hitPercent);
    }
}
```

#### 总结

LruCache的最近最少使用的策略是通过LinkedHashMap，将LinkedHashMap的构造方法的accessOrder字段设置true，让LinkedHashMap按访问顺序排序，不断将常用的元素放到表尾，不常用的元素放到表头。每次增、删、改都判断缓存内存是否大于最大缓存值，如果大于则用一个死循环不断将表头元素移除，直到内存小于最大内存。