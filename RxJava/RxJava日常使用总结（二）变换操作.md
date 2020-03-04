#### RxJava日常使用总结（二）变换操作

#### 在使用RxJava的过程中，变换是很常用的，从一个数据类型转换到另外一个数据类型，再或者从一个Observable（数据源、管道）转接到另外一个Observable。

### flatMap操作符
将一个发射数据的Observable变换为多个Observables，然后将它们发射的数据合并后放进一个单独的Observable。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-627018f4f69f0c7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 日常工作中，flatMap一般用于发送一个事件的Observable，转换成发送多个事件的Observable，再或者将一个发送多个事件的Observable转换为发送一个事件的Observable。

- 举一个很常见的例子，批量上传图片。后端只提供单个上传的接口，图片需要压缩。如果不用RxJava的话，我们一般会这么做：遍历图片path集合，创建对应的File集合，再交给图片压缩框架，设置callback，批量压缩，获取压缩后的图片path集合，再遍历上传，设置callback，多个结果聚合在一起，返回给外部。多个for循环和callback嵌套就会很难看，相信大家都写过这样的代码，那么用Rx要怎么写呢？

```
public Observable<List<Images>> startUploadMultipleImage(Activity activity,
                                                             String tag,
                                                             List<String> filePaths) {
        return Observable.just(filePaths)
                .flatMap(new Function<List<String>, ObservableSource<String>>() {
                    @Override
                    public ObservableSource<String> apply(List<String> filePaths) throws Exception {
                        //1、压缩图片，得到压缩文件的path，遍历
                        List<String> compressFilePaths = Flora
                                .with(activity)
                                .bitmapConfig(Bitmap.Config.ARGB_8888)
                                .compressTaskNum(filePaths.size())
                                .load(filePaths)
                                .compressSync();
                        return Observable.fromIterable(compressFilePaths);
                    }
                })
                .flatMap(new Function<String, ObservableSource<Images>>() {
                    @Override
                    public ObservableSource<Images> apply(String path) throws Exception {
                        //2、对每张压缩图片进行上传
                        return startUploadImage(tag, path);
                    }
                })
                //3、结果堆积为List
                .toList()
                .toObservable();
    }
```

- 可以看到，先just创建一个发送一个多个图片的path集合（只发送一次事件），再使用flatMap操作符，压缩多张图片后，Observable.fromIterable()，遍历发送压缩后的图片文件的path（发送多次事件），再使用一次flatMap进行图片上传操作（因为是多个事件发送，这里的flatMap也是被调用多次），最后的结果使用toList操作符聚合在一起（其实就是将发送的数据放到一个List再发出），最后的toObservable()只是将结果转换类型为Observable（toList()返回的Single）。一条龙下来，没有任何的callback嵌套，没一步都包裹在每个操作符的回调中，for循环嵌套也不会出现了~


- 其实flatMap对数据的合并，因为内部是使用merge，merge是合并数据时是可能有无序的，如果想有序，就需要使用concatMap。所以这里将concatMap也介绍一下。

#### concatMap操作符

和flatMap类似，不过内部合并数据是使用concat，而不像flatMap是使用merge，导致顺序可能会不一致。我们使用concatMap改造上面图片上传的例子吧~

```
Observable.just(filePaths)
			  //其实只改了这里
            .concatMap(new Function<List<String>, ObservableSource<String>>() {
                @Override
                public ObservableSource<String> apply(List<String> filePaths) throws Exception {
						//...省略压缩代码
                    return Observable.fromIterable(compressFilePaths);
                }
            })
            //...
            .toObservable();
```

#### map操作符

对Observable发射的每一项数据应用一个函数，执行变换操作。简单理解就是将数据做处理转换，就像流水线工人，对流到工位的工件做处理，再输出给下个工位的流水线工人。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-ae858cd707310a45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 还是图片上传的例子，针对单图上传，我们可能会得到一个图片的path，然而如果图片压缩库需要传入的是file对象，这时候就需要将path转换为file对象。

```
public Observable<Images> startUploadImage(String tag, String path) {
        FragmentActivity activity = getActivity();
        return Observable
                .just(path)
                .map(new Function<String, File>() {
                    @Override
                    public File apply(String path) throws Exception {
                        return new File(path);
                    }
                })
                .map(new Function<File, File>() {
                    @Override
                    public File apply(File file) throws Exception {
                        String compressFile = Flora.with(activity)
                                .bitmapConfig(Bitmap.Config.ARGB_8888)
                                .load(file).compressSync();
                        return new File(compressFile);
                    }
                })
                .flatMap(new Function<File, ObservableSource<HttpModel<Images>>>() {
                    @Override
                    public ObservableSource<HttpModel<Images>> apply(File file) throws Exception {
                        return UploadRequestManager.uploadImageFile(activity, tag, file);
                    }
                }).map(new Function<HttpModel<Images>, Images>() {
                    @Override
                    public Images apply(HttpModel<Images> model) throws Exception {
                        return model.getData();
                    }
                });
    }
```

#### cast操作符
操作符将原始Observable发射的每一项数据都强制转换为一个指定的类型，然后再发射 数据，它是 map 的一个特殊版本。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-d7c4e604236fbc4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 简单理解，cast其实就是对上游发送过来的事件类型做一个强制类型转换。

- 举一个例子，我们对RecyclerView做条目更新时，需要遍历Adapter的数据集，找到我们需要的条目的Model类，更新里面的数据，在notifyDataSetChanged()。

```
Object selectItemModel = onObtainCurrentSelectItemModel();
        Observable.fromIterable(mItems)
                .filter(new Predicate<Object>() {
                    @Override
                    public boolean test(Object itemModel) throws Exception {
                        return itemModel instanceof SingleSelectOptionModel;
                    }
                })
                .cast(SingleSelectOptionModel.class)
                .map(new Function<SingleSelectOptionModel, SingleSelectOptionModel>() {
                    @Override
                    public SingleSelectOptionModel apply(SingleSelectOptionModel optionModel) throws Exception {
                        if (optionModel.getItemData() == selectItemModel) {
                            optionModel.setSelect(true);
                        } else {
                            optionModel.setSelect(false);
                        }
                        return optionModel;
                    }
                })
                .all(new Predicate<SingleSelectOptionModel>() {
                    @Override
                    public boolean test(SingleSelectOptionModel optionModel) throws Exception {
                        return optionModel != null;
                    }
                })
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<Boolean>() {
                    @Override
                    public void accept(Boolean isOk) throws Exception {
                        if (isOk) {
                            mAdapter.notifyDataSetChanged();
                            vStatusView.showContent();
                        }
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                        vStatusView.showError();
                    }
                });
```

- 在上面，mItems是一个Object类型的集合，Observable.fromIterable()遍历数据集，再filter过滤操作符，将SingleSelectOptionModel类型的数据都过滤掉了，剩下的都是SingleSelectOptionModel类型的数据，后面都要使用这个类型时，就可以直接使用cast(SingleSelectOptionModel.class)操作符，强制将上游的数据做类型转换~下游就可以直接使用了。