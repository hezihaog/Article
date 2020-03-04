#### 过滤操作符，对数据流做条件过滤操作，就像流水间上的质检员，对不符合规则的零件剔除。

#### Distinct操作符。抑制（过滤掉）重复的数据项。简单来说就是去重。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-5906d2d76172f2e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如一个List集合中，存在多个数据重复，需要遍历去重，就可以使用Distinct操作符。

```
Observable.fromIterable(mItems)
                .distinct()
                .toList()
                .toObservable()
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<List<Object>>() {
                    @Override
                    public void accept(List<Object> objects) throws Exception {
                        //...
                    }
                });
```

#### ElementAt操作符。只发射第N项数据。类似集合的ArrayList.get(index)。取出对应位置的数据。在RxJava事件流中就是只发送指定位置的数据事件。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-cacf17129a9f5ae4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如RecyclerView的Adapter的数据集，我已经知道数据位置了，用角标查找出来，再更新列表控件。

```
int index = 0;
        Observable.fromIterable(mItems)
                .elementAt(index)
                .cast(SingleSelectOptionModel.class)
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<SingleSelectOptionModel>() {
                    @Override
                    public void accept(SingleSelectOptionModel optionModel) throws Exception {
                        optionModel.setSelect(true);
                        mAdapter.notifyItemChanged(index);
                    }
                });
```

#### firstElement操作符。

- 其实如果简单的想要第一项，还有一个操作符叫firstElement，和elementAt(index)一样，只不过他是固定的取出第一项。如果看它的源码，其实就是转调elementAt()，并传入了参数0。所以这2个操作符的效果是一样的，只不过RxJava提供了firstElement来做常见的取出第一个元素的操作。

```
Observable.fromIterable(mItems)
                .filter(new Predicate<Object>() {
                    @Override
                    public boolean test(Object itemModel) throws Exception {
                        return itemModel instanceof SingleSelectOptionModel;
                    }
                })
                .cast(SingleSelectOptionModel.class)
                .firstElement()
                .toObservable()
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<SingleSelectOptionModel>() {
                    @Override
                    public void accept(SingleSelectOptionModel singleSelectOptionModel) throws Exception {
                        //...
                    }
                });
```

#### filter操作符。只发射通过了谓词测试的数据项。其实就是一个过滤器，传入一个Predicate谓词接口实现参数，复写test方法，返回true代表通过，返回false则代表需要过滤掉。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-2a4c743c69d2d508.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如，搜索功能的搜索按钮，在点击时，判断是否有输入文字，文字是否为空或空串。使用filter操作符，即可过滤掉没有输入的文字，并且Toast提示用户。

```
RxUtil.click(vSearch)
                .map(new Function<Object, String>() {
                    @Override
                    public String apply(Object o) throws Exception {
                        return vSearchEdit.getText().toString().trim();
                    }
                })
                .filter(new Predicate<String>() {
                    @Override
                    public boolean test(String inputText) throws Exception {
                        boolean isEmpty = TextUtils.isEmpty(inputText);
                        if (isEmpty) {
                            showToast(R.string.recommend_search_input_empty_tip);
                        }
                        return !isEmpty;
                    }
                })
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(String inputText) throws Exception {
                        search(inputText);
                    }
                });
```

#### first和last操作符。first代表取出第一个事件发送，last则是取出最后一个事件发送，它们需要传一个defaultItem的参数，作为如果找不到时发送的默认值。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-91e4b2acca5fb97a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1641428-6637ed786d34cf51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如本地搜索一样的功能，查找数据集，判断指定关键字的数据是否存在（可能会有多条，但我只需要找到第一条就足够了），不存在则发送一个默认的数据。last也是一样。

```
Observable.fromIterable(mItems)
                .filter(new Predicate<Object>() {
                    @Override
                    public boolean test(Object itemModel) throws Exception {
                        return itemModel instanceof SingleSelectOptionModel;
                    }
                })
                .cast(SingleSelectOptionModel.class)
                .filter(new Predicate<SingleSelectOptionModel>() {
                    @Override
                    public boolean test(SingleSelectOptionModel model) throws Exception {
                        return "我们会怎么样".equals(model.getTitle());
                    }
                })
                .first(new SingleSelectOptionModel("找不到"))
                .toObservable()
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<SingleSelectOptionModel>() {
                    @Override
                    public void accept(SingleSelectOptionModel singleSelectOptionModel) throws Exception {
                        //...
                    }
                });
```

#### skip操作符，抑制Observable发射的前N项数据。就是跳过发送的事件的前多少个。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-ff55b2334946fb6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如观察的数据源一开始会发送一个默认值，可能我们并不需要，这时候我们就可以使用skip操作符，跳过第一个发送的事件。

```
RxLunarDataTimePicker.pickChange(mDataTimePopupDialog)
                //忽略掉一开始发送的初始化默认值
                .skip(1)
                .as(RxLifecycleUtil.bindLifecycle(this))
                .subscribe(new Consumer<LunarDataTimePickInfo>() {
                    @Override
                    public void accept(LunarDataTimePickInfo info) throws Exception {
                        int type = info.getType();
                        CalendarTypeEnum calendarTypeEnum;
                        if (DatePickerView.DATE_TYPE_SOLAR == type) {
                            calendarTypeEnum = CalendarTypeEnum.SOLAR;
                        } else if (DatePickerView.DATE_TYPE_LUNAR == type) {
                            calendarTypeEnum = CalendarTypeEnum.LUNAR;
                        } else {
                            calendarTypeEnum = CalendarTypeEnum.SOLAR;
                        }
                        mCalendar.set(Calendar.YEAR, info.getYear());
                        mCalendar.set(Calendar.MONTH, info.getMonthOfYear() - 1);
                        mCalendar.set(Calendar.DAY_OF_MONTH, info.getDayOfMonth());
                        mCalendar.set(Calendar.HOUR_OF_DAY, info.getHour());
                        mCalendar.set(Calendar.MINUTE, 0);
                        mCalendar.set(Calendar.MILLISECOND, 0);
                        updateBirthday(mCalendar.getTime().getTime(), calendarTypeEnum);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                        throwable.printStackTrace();
                    }
                });
```

#### skipLast操作符。和skip操作符一样，同样是跳过指定数量的事件，只是skip是跳过前面开始指定数量的事件，skipLast则是从后面开始，跳过指定数量的事件。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-fd5bdf3fea831789.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### take操作符。只发射前面的N项数据。就是从前面开始，保留指定数量的事件。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-47fb1c2d186993c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 例如，像微信发朋友圈添加图片的需求，在选择了多张图片后，需要将图片的path路径绑定到条目的Model中。但是Items中是所有的条目，会比我们要设置的已选择的图片条目要多，这时候我们就可以使用take操作符，即使遍历整批条目，我们也只要前面开始，指定图片数量的Model（为什么从前面？选择图片后要填充图片地址就是从前面开始的呀~），再配合zip操作符和选择图片path集合做一一配对设置，这样就完成了占位图转变到设置图片数据的过程啦~

```
//指定数量的，没有设置图片的占位符
        Observable<PressQuestionUploadImageModel> listItemObservable = Observable.fromIterable(mItems)
                .filter(new Predicate<Object>() {
                    @Override
                    public boolean test(Object target) throws Exception {
                        return target instanceof PressQuestionUploadImageModel
                                && ((PressQuestionUploadImageModel) target).getPicFilePath() == null;
                    }
                })
                .cast(PressQuestionUploadImageModel.class)
                //限制只发送未设置的图片的指定数量
                .take(imgPaths.size());
        return Observable.zip(listItemObservable, Observable.fromIterable(imgPaths), new BiFunction<PressQuestionUploadImageModel, String, PressQuestionUploadImageModel>() {
            @Override
            public PressQuestionUploadImageModel apply(PressQuestionUploadImageModel model, String path) throws Exception {
                model.setPicFilePath(path);
                return model;
            }
        })
```

#### takeLast操作符，和take一样，take是从前面开始保留指定数量的事件，而takeLast则是从后面开始，如果有需求是从后面开始填充的话，就可以使takeLast操作符。

![image.png](https://upload-images.jianshu.io/upload_images/1641428-907ac42a1bb7a6ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)