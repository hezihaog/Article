#### RxJava日常使用总结（四）结合操作

#### 在RxJava中，除了创建、过滤、变换之外，还有一个十分常用的特性，就是结合，多个数据源的结合操作，得出相应的结果，再进行相关的处理。

#### CombineLatest操作符。当两个Observables中的任何一个发射了数据时，使用一个函数结合每个Observable发射的最 近数据项，并且基于这个函数的结果发射数据。其实就是，多个数据源，其中任意一个发射了数据，就会拿去其他数据源最近发射的数据结合进行处理、比较。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-e77944d29a57437c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 在Android中，我们经常会做一些表单验证，例如登录，要监听账号输入框和密码输入框的输入变化，在回调中判断2个输入框中的文本是否不为空，是否合法等，才能点亮登录按钮或提交按钮。在RxJava中，如果将2个输入框都当成是数据源，每次输入改变都是一次发送事件，我们就可以使用CombineLatest操作符来做表单验证~

```
//监听2个输入框输入切换提交按钮
Observable.combineLatest(RxTextView.textChanges(vTopicTitleEdit), RxTextView.textChanges(vTopicDescriptionEdit),
                new BiFunction<CharSequence, CharSequence, Boolean>() {
                    @Override
                    public Boolean apply(CharSequence titleInputText, CharSequence descriptionInputText) throws Exception {
                        return InputVerifyUtil.verifyTitle(titleInputText.toString().trim())
                                && InputVerifyUtil.verifyDesc(descriptionInputText.toString().trim());
                    }
                })
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<Boolean>() {
                    @Override
                    public void accept(Boolean isReady) throws Exception {
                        if (isReady) {
                            commitBtn.setEnabled(true);
                        } else {
                            commitBtn.setEnabled(false);
                        }
                    }
                });
```

- 这里用到了RxBinding2对2个输入框的输入改变做监听，并转换为事件发出，配合combineLatest操作符，在每次改变时，都做一次验证和比对，返回true，则将按钮设置为可用，返回false则设置为不可用。

#### Merge操作符。合并多个Observables的发射物，就是交错合并数据，最后输出一个交错后的数据。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-06bcb2bda0145918.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如现在有2组数据，产品要求2组数据的条目交错组合显示在RecyclerView上，不使用RxJava的话，就需要一个for循环，用角标index取出2个集合中的数据，再add到另外一个输出集合，还要考虑角标越界的问题。如果使用RxJava，使用Merge操作符就能轻松实现了。

```
Items itemsOne = new Items();
        //...添加数据操作
        Items itemsTwo = new Items();
        //...添加数据操作
        Observable.merge(Observable.fromIterable(itemsOne), Observable.fromIterable(itemsTwo))
                .toList()
                .toObservable()
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<List<Object>>() {
                    @Override
                    public void accept(List<Object> items) throws Exception {
                        //交错、综合的Items
                    }
                });
```

#### StartWith操作符。在数据序列的开头插入一条指定的项。类似集合List的add(index, object)，插入元素到最前面，在RxJava的StartWith中就是插入一个事件到最前面。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-efcc2d0ebdb899ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如评论列表，第一个条目Item是多个评论标签的Tag条目，后面就是这个Tag对应的评论列表。请求的时候是一起请求的，所以先请求Tag数据，再取出Tag数据中的第一个作为拉取评论列表的参数，最后再将Tags的数据插入到最前面。

```
//Tags的数据集
Observable<HttpModel<HttpListModel<TeacherEvaluateTagModel>>> tagsObservable = getTeacherEvaluateTags(context, tag, teacherUid);
        //请求列表必须先Tag标签接口再评论列表
        return tagsObservable.flatMap(new Function<HttpModel<HttpListModel<TeacherEvaluateTagModel>>, ObservableSource<HttpModel<?>>>() {
            @Override
            public ObservableSource<HttpModel<?>> apply(HttpModel<HttpListModel<TeacherEvaluateTagModel>> model) throws Exception {
                return Observable.create(new ObservableOnSubscribe<HttpModel<HttpListModel<TeacherEvaluateTagModel>>>() {
                    @Override
                    public void subscribe(ObservableEmitter<HttpModel<HttpListModel<TeacherEvaluateTagModel>>> emitter) throws Exception {
                        if (model == null || model.getData() == null || model.getData().getList() == null) {
                            emitter.onError(new NullPointerException("TeacherEvaluateTags return data is null"));
                        } else {
                            emitter.onNext(model);
                            emitter.onComplete();
                        }
                    }
                })
                        .flatMap(new Function<HttpModel<HttpListModel<TeacherEvaluateTagModel>>, ObservableSource<HttpModel<?>>>() {
                            @Override
                            public ObservableSource<HttpModel<?>> apply(HttpModel<HttpListModel<TeacherEvaluateTagModel>> model) throws Exception {
                                return getTeacherEvaluateList(context, tag, teacherUid, model.getData().getList().get(0).getId(), page)
                                        .map(new Function<HttpModel<?>, HttpModel<?>>() {
                                            @Override
                                            public HttpModel<?> apply(HttpModel<?> model) throws Exception {
                                                return model;
                                            }
                                        });
                            }
                        });
            }
        })//最后将tag数据放到最前面
                .startWith(tagsObservable)
                .toList()
                .toObservable();
```

#### Zip操作符。通过一个函数将多个Observables的发射物结合到一起，基于这个函数的结果为每个结合体发 射单个数据项。作用就是：俩俩配对组合出一个数据，注意，如果2个数据集的数量不一致，那么配对会以最少的来作为基准进行配对，就是说比较多数据的那个数据集中无法找到配对对象的数据就会被抛弃掉。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-2dc6e741d63f3bd8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 还是原来的第一篇的例子，遍历条目数据，将对应的Index角标和条目数据组合成一个DataIndexModel数据集。后续可以用来排序操作等。或者使用index角标找到这个数据集。

```
//包裹数据和角标位置的模型，一般用于Rx，遍历数据，过滤指定类型数据后，需要用到数据和角标的情况
public class DataIndexModel<T> {
    private T data;
    private int index;

    public DataIndexModel(T data, int index) {
        this.data = data;
        this.index = index;
    }

    public T getData() {
        return data;
    }

    public int getIndex() {
        return index;
    }
}
```

```
Observable.zip(Observable.range(0, mItems.size() - 1), Observable.fromIterable(mItems),
                new BiFunction<Integer, Object, DataIndexModel<Object>>() {
            @Override
            public DataIndexModel<Object> apply(Integer index, Object data) throws Exception {
                return new DataIndexModel<>(data, index);
            }
        })
                .filter(new Predicate<DataIndexModel<Object>>() {
                    @Override
                    public boolean test(DataIndexModel<Object> model) throws Exception {
                        return model.getIndex() != 15;
                    }
                })
                .map(new Function<DataIndexModel<Object>, Object>() {
                    @Override
                    public Object apply(DataIndexModel<Object> model) throws Exception {
                        return model.getData();
                    }
                })
                .subscribe();
```