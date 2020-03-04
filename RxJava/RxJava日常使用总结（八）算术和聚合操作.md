#### RxJava日常使用总结（八）算术和聚合操作

#### 本篇介绍算数和聚合的操作符。像average(求平均数)，min(求最小值)，max(求最大值)，sum(求和)，这3个操作符在RxJava1的Math拓展库（不支持RxJava2）才存在，而RxJava2中没有，平时也用得比较少，所以就不介绍了，介绍几个常用的操作符~。

#### count操作符

计算Observable发射的事件数量的个数。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-3fdf5dc0f5ce5ed9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如计算是否还可以加载更多，这里判断条件是，后端返回的数据个数，一般我们请求多少条就返回多少条，除非没有那么多了，才返回不够，所以就可以遍历后端返回的数据集合，调用count操作符，数量大于指定数量则判定为可以加载更多。

```
Observable
        .just(modelList)
        //先遍历多个接口的结果
        .flatMap(new Function<List<HttpModel<?>>, ObservableSource<HttpModel<?>>>() {
            @Override
            public ObservableSource<HttpModel<?>> apply(List<HttpModel<?>> httpModels) throws Exception {
                return Observable.fromIterable(httpModels);
            }
        })
        //取出每个结果的Data数据
        .map(new Function<HttpModel<?>, Object>() {
            @Override
            public Object apply(HttpModel<?> model) throws Exception {
                return model.getData();
            }
        })
        //只找老师列表的接口
        .filter(new Predicate<Object>() {
            @Override
            public boolean test(Object target) throws Exception {
                return target instanceof RecommendFastTestServerModel;
            }
        })
        //计算数量
        .count()
        //大于请求的数量，则代表有更多数据
        .map(new Function<Long, Boolean>() {
            @Override
            public Boolean apply(Long count) throws Exception {
                return count >= ApiUrl.PER_PAGE;
            }
        });
```

#### concat操作符

不交错的发射两个或多个Observable的发射物。其实就是将多个数据源发射的数据按顺序合并，所以他们是串联的，但不是聚合在一起再一起返回！需要聚合在一起需要使用toList操作符或collect操作符进行聚合。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-1a63046a01cbfb61.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如一个列表需要2个接口一起请求返回，就可以使用concat操作符，再配合collect操作符。

```
请求老师详情和老师服务，并同时返回
Observable
        .concat(getViewModel().getUserDetail(tag, uid), UCRequestManager.getTeacherService(context, tag, uid))
        //组合2个请求，再收集到一起再返回
        .collect(new Callable<ArrayList<HttpModel<? extends Serializable>>>() {
            @Override
            public ArrayList<HttpModel<? extends Serializable>> call() throws Exception {
                return new ArrayList<>();
            }
        }, new BiConsumer<ArrayList<HttpModel<? extends Serializable>>, HttpModel<? extends Serializable>>() {
            @Override
            public void accept(ArrayList<HttpModel<? extends Serializable>> modelList, HttpModel<? extends Serializable> httpModel) throws Exception {
                modelList.add(httpModel);
            }
        }).subscribe(new Consumer<ArrayList<HttpModel<? extends Serializable>>>() {
            @Override
            public void accept(ArrayList<HttpModel<? extends Serializable>> httpModels) throws Exception {
                //...
            }
        }, new Consumer<Throwable>() {
            @Override
            public void accept(Throwable throwable) throws Exception {
                //...
            }
        }))
```

#### reduce操作符

- reduce操作符多用于计算累计结果。reduce操作符会将一个数据源发出的所有事件全部聚合，多次回调BiFunction，上一次的结果会继续传入，最后统一输出到观察者的onNext

![image.png](https://upload-images.jianshu.io/upload_images/1641428-e6b65cf4d55e2058.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
Observable.just(1, 2, 3, 4, 5)
        .reduce(new BiFunction<Integer, Integer, Integer>() {
            @Override
            public Integer apply(Integer value, Integer value2) throws Exception {
                return value + value2;
            }
        })
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer result) throws Exception {
                //最后结果15...
            }
        });
```