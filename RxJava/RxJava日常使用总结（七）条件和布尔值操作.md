#### RxJava日常使用总结（七）条件和布尔值操作

#### 这篇，我们讲几个条件判断和布尔值操作相关的操作符。

#### All操作符。判定是否Observable发射的所有数据都满足某个条件。就是判断某个数据源发射的数据是否符合规则，然后输出一个Boolean类型的返回值。（这里有小伙伴就会问了，这效果和filter不是一样的吗？其实的确是一样的，只是filter不会改变上游的数据类型，而all操作符则强制输出类型为Boolean值类型，所以使用all还是filter，看场景。）

![image.png](https://upload-images.jianshu.io/upload_images/1641428-84b6685a03625dd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如，在一个问题列表中，点击条目，跳转到聊天室，自动发送多条问题消息。（有点像京东、天猫找客服，自动发一些确定的问题）

```
    /**
     * 自动批量发送消息
     */
    public Observable<Boolean> autoSendMsgList(ArrayList<AutoSendMsgModel> autoSendMsgList) {
        return Observable.fromIterable(autoSendMsgList)
                .flatMap(new Function<AutoSendMsgModel, ObservableSource<Boolean>>() {
                    @Override
                    public ObservableSource<Boolean> apply(AutoSendMsgModel model) throws Exception {
                        return sendTextMsg(model.getMsg());
                    }
                })
                .all(new Predicate<Boolean>() {
                    @Override
                    public boolean test(Boolean isSuccess) throws Exception {
                        return isSuccess;
                    }
                }).toObservable();
    }
```

- all操作符，判定每次发送的消息的返回结果都要为true，才算发送成功。

#### Amb操作符。给定两个或多个Observables，它只发射首先发射数据或通知的那个Observable的所有数据，其他被抛弃。意思就是竞争，谁先发送事件，谁就赢。不管是onNext()，还是onError或onCompleted()事件。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-f4aefc87c431c69c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 这个场景就很好理解了，例如我们请求接口前，先去找内存缓存，再找本地缓存，最后没有缓存可用，就请求接口数据，如果有缓存，那接口就不请求了。

```
Observable.ambArray(
                                Observable.create(new ObservableOnSubscribe<List<SearchTagModel>>() {
                                    @Override
                                    public void subscribe(ObservableEmitter<List<SearchTagModel>> emitter) throws Exception {
                                        if (mHotTagCacheList.size() > 0) {
                                            emitter.onNext(mHotTagCacheList);
                                            emitter.onComplete();
                                        }
                                    }
                                })
                                        .map(new Function<List<SearchTagModel>, BaseSearchModel>() {
                                            @Override
                                            public BaseSearchModel apply(List<SearchTagModel> tagModels) throws Exception {
                                                mHotTagList.clear();
                                                mHotTagList.addAll(tagModels);
                                                return new HotSearchModel(tagModels);
                                            }
                                        }),
                                //热门搜索
                                mClient.requestTagsInObservable(TAG)
                                        .filter(new Predicate<HttpModel<List<SearchTagModel>>>() {
                                            @Override
                                            public boolean test(HttpModel<List<SearchTagModel>> httpModel) throws Exception {
                                                return httpModel.getData() != null && httpModel.getData().size() > 0;
                                            }
                                        })
                                        .map(new Function<HttpModel<List<SearchTagModel>>, BaseSearchModel>() {
                                            @Override
                                            public BaseSearchModel apply(HttpModel<List<SearchTagModel>> httpModel) throws Exception {
                                                //缓存
                                                mHotTagCacheList.clear();
                                                mHotTagCacheList.addAll(httpModel.getData());
                                                mHotTagList.clear();
                                                mHotTagList.addAll(httpModel.getData());
                                                return new HotSearchModel(httpModel.getData());
                                            }
                                        }))
```

- 例如搜索功能，热门搜索，如果之前在前面一个界面已经请求过了，则可以直接带数据过来，直接使用，不再调接口拿。再或者前面没有请求过，没有缓存，则请求接口，再将数据缓存到集合（内存缓存）。

#### Contains操作符。判定一个Observable是否发射一个特定的值，并返回一个Boolean结果值。效果就是List集合.contains(object)的效果，判断某个值是否存在集合中，在RxJava中，Contains操作符就是判断某个数据集是否发出过指定的事件值。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-5fd2cb7c6a575655.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如，遍历列表数据集，判断是否存在某个类型的条目，存在或不存在做某些操作。

```
Object model = ...
        Items items = new Items();
        Observable.fromIterable(items)
                .contains(model)
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<Boolean>() {
                    @Override
                    public void accept(Boolean isContains) throws Exception {
                        //...
                    }
                });
```