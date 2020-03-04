#### Java8新特性之Stream

函数式编程中还有一个重要的东西-Stream。Stream是对集合Collection的增强。将传统的for、while等操作宏观化，即使再复杂的操作都能像流水线一样书写。

#### Stream的特点

- Stream不会存储数据。Stream不是集合，Stream是操作集合中的数据，不存储数据。

- Stream不会改变源数据，Stream的操作相当于将集合复制一份再进行一系列的操作，操作不会影响原来的数据。

- Stream是惰性的。Stream的各种中间操作并不会马上执行，只有当遇到终止操作时，才开始执行。目的是为了提高效率。

- Stream只能被消费一次，重复使用Stream会抛出异常。

#### Stream使用的步骤

1. 创建一个Stream
2. 进行一系列的中间操作，对数据做处理
3. 终止操作，最终输出一个结果

#### 创建Stream

- 通过集合Collection生成一个Stream

```
Stream<String> streamByList = Arrays.asList("wally", "barry", "rose", "barry").stream();
```

- 通过Stream.of创建

```
Stream<String> streamByOf = Stream.of("wally", "barry", "rose", "barry");
```

- 生成指定类型的Stream，为了方便使用，Java8除了Stream，还提供了IntStream，LongStream，DoubleStream等

```
IntStream intStream = IntStream.of(1, 2, 3);
LongStream longStream = LongStream.of(1000, 2000, 3000);
DoubleStream doubleStream = DoubleStream.of(0.1, 0.2, 0.3, 0.4);
```

- 无限流

1. Stream.iterate。起始位置+步长

```
//从0开始，下一个数是上一个数+1所得
Stream<Integer> integerStream = Stream.iterate(0, integer -> integer + 1);
        integerStream.forEach(System.out::println);
```

2. Stream.generate。每次生成一个元素。

```
//每次生成一个随机数
Stream<Double> doubleStream = Stream.generate(Math::random);
        doubleStream.forEach(System.out::println);
```

##### 中间操作

中间操作不会立即执行，只有出现终止操作时才执行。

#### filter过滤

filter中的条件，返回true的才会保留

```
//过滤，只保存rose，结果聚集为List
List<String> result = streamByOf.filter(s -> s.equals("rose")).collect(Collectors.toList());
System.out.println(result);

//过滤，只保存rose，直接遍历
streamByOf.filter(s -> s.equals("rose")).forEach(System.out::println);
```

#### distinct去重

distinct是调用元素的hashCode和equals方法进行判断重复来去重

```
Stream<String> streamByOf = Stream.of("wally", "barry", "rose", "barry");
//去重，barry存在2个，去重后只有一个
List<String> distinctResult = streamByOf.distinct().collect(Collectors.toList());
System.out.println(distinctResult);
```

#### map，数据变换

```
Stream<String> streamByOf = Stream.of("wally", "barry", "rose", "barry");
//每个元素进行大写转换
List<String> collectResult = streamByOf.map(String::toUpperCase).collect(Collectors.toList());
System.out.println(collectResult);
```

#### flatMap，摊平

组合多个流为一个流

```
Stream<List<String>> concatStream = Stream.of(Arrays.asList("wally", "barry", "rose"), Arrays.asList("jack", "ben"));
        concatStream.flatMap(new Function<List<String>, Stream<String>>() {
            @Override
            public Stream<String> apply(List<String> names) {
                return names.stream();
            }
        }).forEach(System.out::println);
```

#### sorted，排序

```
Stream<Integer> integerStream = Stream.of(1, 4, 8, 2, 0, -1);
integerStream.sorted().forEach(System.out::println);
```

#### reduce，累计

每次进行处理的处理会再下一次回调中传入。例如这里的value1和value2进行相加得出result，下一次的value1就是result的值，而value2则为下一个要处理的数，以此类推，下一轮回调时value1的值就为上一次的result + value2。

```
Stream<Integer> integerStream = Stream.of(1, 2, 3, 4, 5);
Integer value = integerStream.reduce(new BinaryOperator<Integer>() {
    @Override
    public Integer apply(Integer value1, Integer value2) {
        System.out.println("value1 - " + value1 + " " + "value2 - " + value2);
        int result = value1 + value2;
        System.out.println("> result - " + result);
        return result;
    }
}).orElseGet(new Supplier<Integer>() {
    @Override
    public Integer get() {
        return -1;
    }
});
//value1 - 1 value2 - 2
//> result - 3
//value1 - 3 value2 - 3
//> result - 6
//value1 - 6 value2 - 4
//> result - 10
//value1 - 10 value2 - 5
//> result - 15
//15
System.out.println(value);
```

#### 终止操作

终止操作，上面的一系列操作都必须在终止操作调用时才开始进行处理，所以Stream是惰性的。

#### collect，聚合数据为一个集合

- toList，聚合到一个List集合

```
Stream<Integer> integerStream = Stream.of(1, 2, 3, 4, 5);
List<Integer> result = integerStream.collect(Collectors.toList());
```

- toSet，聚合到一个Set集合

```
Stream<Integer> integerStream = Stream.of(1, 2, 3, 4, 5);
Set<Integer> set = integerStream.collect(Collectors.toSet());
```

#### forEach，遍历

```
Stream<String> streamByOf = Stream.of("wally", "barry", "rose");
streamByOf.forEach(System.out::println);
```

#### anyMatch

判断，只要一个匹配，就返回true

```
Stream<String> streamByOf = Stream.of("wally", "barry", "rose");
boolean result = streamByOf.anyMatch(s -> s.length() >= 5);
//true，wally和barry长度都等于5，即使rose的长度为4不符合也返回true
System.out.println("是否存在一个长度大于等于5的名字: " + result);
```

#### allMatch

判断，所有都匹配，才返回true

```
Stream<String> streamByOf = Stream.of("wally", "barry", "rose");
boolean result = streamByOf.allMatch(s -> s.length() >= 5);
//返回false，因为rose的长度为4
System.out.println("是否所有名字的长度大于等于5: " + result);
```

#### sum，求和

创建流，发射0到5的数，sum进行求和。(0+1+2+3+4+5=15)

```
int sum = IntStream.rangeClosed(0, 5).sum();
//输出15
System.out.println("sum: " + sum);
```

#### count，统计元素个数

- IntStream.rangeClosed，闭区间，生成指定区间的元素，0,1,2,3,4,5
- IntStream.range，半开半闭区间，0,1,2,3,4

```
long count = IntStream.rangeClosed(0, 5).count();
//输出6
System.out.println("count: " + count);
```

#### Max，查找最大值

```
Stream<Integer> integerStream = Stream.of(1, 4, 8, 2, 0, -1);
Integer maxValue = integerStream.max(Integer::compareTo).orElse(-1);
System.out.println("maxValue: " + maxValue);
```

#### Min，查找最少值

```
Stream<Integer> integerStream = Stream.of(1, 4, 8, 2, 0, -2);
Integer minValue = integerStream.min(Integer::compare).orElse(-1);
System.out.println("minValue: " + minValue);
```

#### averagingInt，求平均数

结果：2 + 4 + 6 / 3 = 4

```
Stream<Integer> integerStream = Stream.of(2, 4, 6);
Double averagingValue = integerStream.collect(Collectors.averagingInt(new ToIntFunction<Integer>() {
    @Override
    public int applyAsInt(Integer value) {
        return value;
    }
}));
System.out.println("平均数: " + averagingValue);
```