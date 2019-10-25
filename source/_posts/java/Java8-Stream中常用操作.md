---
title: Java8-Stream中的常用操作
date: 2019-05-16 22:23:11
tags:
    - Java8
    - stream
categories: 
    - java
notshow: true
---
# 1.前言
Java8提供了很多新特性，其中有一个就是基于流Stream的操作。Stream是一组用来处理数组，集合的API
## 1.1 特性
<!-- more -->
- 不是数据结构，没有内部存储。
- 不支持索引访问。
- 延迟计算
- 支持并行
- 很容易生成数据或集合
- 支持过滤，查找，转换，汇总，聚合等操作


## 1.2 运行机制
Stream分为源source，中间操作，终止操作。
- 流的源可以是一个数组，集合，生成器方法，I/O通道等等。
- 一个流可以有零个或多个中间操作，每一个中间操作都会返回一个新的流，供下一个操作使用，一个流只会有一个终止操作。
- Stream只有遇到终止操作，它的源才会开始执行遍历操作。

## 1.3 Stream的创建
`Stream`的创建其实有很多方式，但是我们在平时用到最多的可能就是基于数组的Stream.of()和集合的stream()方法其实它还有很多种的创建方式，下面将一一列出，并且列举相关实例。
- 1.通过数组,Stream.of()
- 2.通过集合
- 3.通过Stream.generate方法来创建
- 4.通过Stram.iterate方法
- 5.其他API
```java 
public class CreateStream {
    // 1.通过数组,Stream.of()
    static void create1(){
        String[] str = {"a","b","c"};
        Stream<String> str1 = Stream.of(str);
    }
    // 2.通过集合
    static void create2(){
        List<String> strings = Arrays.asList("a", "b", "c");
        Stream<String> stream = strings.stream();
    }
    // 3.通过Stream.generate方法来创建
    static void create3(){
        //这是一个无限流，通过这种方法创建在操作的时候最好加上limit进行限制
        Stream<Integer> generate = Stream.generate(() -> 1);
        generate.limit(10).forEach(x -> System.out.println(x));
    } 
    // 4.通过Stram.iterate方法
    static void create4(){
        Stream<Integer> iterate = Stream.iterate(1, x -> x +1);
        iterate.forEach(x -> System.out.println(x));
    }
    // 5.其他API
    static void create5(){
        String str = "abc";
        IntStream chars = str.chars();
        chars.forEach(x -> System.out.println(x));
    }
}
```

# 2.Stream的常用操作(API)
## 2.1 中间操作
### 2.1.1 filter过滤
接受一个谓词（一个返回boolean的函数）作为参数，并返回一个包括所有符合谓词的元素的流。说白了就是给一个条件，filter会根据这个条件截取流中得数据。
```java
public static void testFilter(){
    List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
    //截取所有能被2整除得数据
    List<Integer> collect = integers.stream().filter(i -> i % 2 == 0).collect(Collectors.toList());
    System.out.println("collect = " + collect);
}
// 结果: collect = [2, 4, 6, 8, 10]
```

### 2.1.2 distinct去重
返回一个元素各异（根据流所生成元素的hashCode和equals方法实现）的流。
```java
public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
    List<Integer> collect = numbers.stream().distinct().collect(Collectors.toList());
    System.out.println("collect = " + collect);
}
// 结果: collect = [1, 2, 3, 4]
```

### 2.1.3 sorted排序
对流中得数据进行排序，可以以自然序或着用Comparator接口定义的排序规则来排序一个流。Comparator能使用lambada表达式来初始化，还能够逆序一个已经排序的流。
```java
public static void main(String[] args) {
    List<Integer> integers = Arrays.asList(5, 8, 2, 6, 41, 11);
    //排序默认为顺序  顺序 = [2, 5, 6, 8, 11, 41]
    List<Integer> sorted = integers.stream().sorted().collect(Collectors.toList());
    System.out.println("顺序 = " + sorted);
    //逆序    逆序 = [41, 11, 8, 6, 5, 2]
    List<Integer> reverseOrder = integers.stream().sorted(Comparator.reverseOrder()).collect(Collectors.toList());
    System.out.println("逆序 = " + reverseOrder);
    //也可以接收一个lambda
    List<Integer> ages = integers.stream().sorted(Comparator.comparing(User::getAge)).collect(Collectors.toList());
}
```

### 2.1.4 limit截取
会返回一个不超过给定长度的流。
```java
public static void testLimit(){
    List<Integer> integers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
    List<Integer> collect = integers.stream().limit(3).collect(Collectors.toList());
    System.out.println("collect = " + collect);
}
// 结果: collect = [1, 2, 1]
```

### 2.1.5 skip舍弃
会返回一个扔掉了前面n个元素的流。如果流中元素不足n个，则返回一个空流。
```java
public static void testSkip(){
    List<Integer> integers = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
    //丢掉流中得前三个元素  
    List<Integer> collect = integers.stream().skip(3).collect(Collectors.toList());
    System.out.println("collect = " + collect);
}
// 结果: collect = [3, 3, 2, 4]
```

### 2.1.6 map归纳
接受一个函数作为参数，这个函数会被应用到每个元素上，并将其映射成一个新的元素。就是根据指定函数获取流中得每个元素得数据并重新组合成一个新的元素。
```java
public static void main(String[] args) {
    //自己建好得一个获取对象list得方法
    List<Dish> dishList = Dish.getDishList();
    //获取每一道菜得名称  并放到一个list中
    List<String> collect = dishList.stream().map(Dish::getName).collect(Collectors.toList());
    //collect = [pork, beef, chicken, french fries, rice, season fruit, pizza, prawns, salmon]
    System.out.println("collect = " + collect);
}
```

### 2.1.7 flatMap扁平化
该方法key可以让你把一个流中的每个值都换成另一个流，然后把所有的流都链接起来成为一个流。
```java
public static void main(String[] args) {
    String[] words = {"Hello", "World"};
    List<String> collect = Stream.of(words).        //数组转换流
            map(w -> w.split("")).  //去掉“”并获取到两个String[]
            flatMap(Arrays::stream).        //方法调用将两个String[]扁平化为一个stream
            distinct().                     //去重    
            collect(Collectors.toList());
    //collect = [H, e, l, o, W, r, d]
    System.out.println("collect = " + collect);
}
```

### 2.1.8 peek
peek的设计初衷就是在流的每个元素恢复运行之前，插入执行一个动作。
```java
public static void main(String[] args) {
    List<Integer> numbers = Arrays.asList(2, 3, 4, 5);
    List<Integer> result =
            numbers.stream()
                    .peek(x -> System.out.println("from stream: " + x))
                    .map(x -> x + 17)
                    .peek(x -> System.out.println("after map: " + x))
                    .filter(x -> x % 2 == 0)
                    .peek(x -> System.out.println("after filter: " + x))
                    .limit(3)
                    .peek(x -> System.out.println("after limit: " + x))
                    .collect(Collectors.toList());
}
// 结果：
//     from stream: 2
//     after map: 19
//     from stream: 3
//     after map: 20
//     after filter: 20
//     after limit: 20
//     from stream: 4
//     after map: 21
//     from stream: 5
//     after map: 22
//     after filter: 22
//     after limit: 22

```

### 2.1.9 collect收集
从上面得代码已经可以看出来，collect是将最终stream中得数据收集起来，最终生成一个list，set，或者map。
```java
public static void main(String[] args) {
    List<Dish> dishList = Dish.getDishList();
    // list
    List<Dish> collect = dishList.stream().limit(2).collect(Collectors.toList());
    // set
    Set<Dish> collect1 = dishList.stream().limit(2).collect(Collectors.toSet());
    // map
    Map<String, Dish.Type> collect2 = dishList.stream().limit(2).collect(Collectors.toMap(Dish::getName, Dish::getType));
}

```

## 2.2 终止操作
- 循环 forEach
- 计算 min、max、count、average
- 匹配 anyMatch、allMatch、noneMatch、findFirst、findAny
- 汇聚 reduce
- 收集器 collect

## 2.3 查找和匹配
常见的数据处理套路是看看数据集中的某些元素是否匹配一个给定的属性。Stream API通过allMatch，anyMatch，noneMatch，findFirst和findAny方法提供了这样的工具。
查找和匹配都是终端操作。

### 2.3.1 anyMatch
anyMatch方法可以回答“流中是否有一个元素能匹配到给定的谓词”。会返回一个boolean值。
```java
public class AnyMatch {
    public static void main(String[] args) {
        List<Dish> dish = Dish.getDish();
        boolean b = dish.stream().anyMatch(Dish::isVegetarian);
        System.out.println(b);
    }
}
```

### 2.3.2 allMatch
allMatch方法和anyMatch类似，校验流中是否都能匹配到给定的谓词。
```java
class AllMatch{
    public static void main(String[] args) {
        List<Dish> dish = Dish.getDish();
        //是否所有菜的热量都小于1000
        boolean b = dish.stream().allMatch(d -> d.getCalories() < 1000);
        System.out.println(b);
    }
}
```

### 2.3.3 noneMatch
noneMatch方法可以确保流中没有任何元素与给定的谓词匹配。
```java
class NoneMatch{
    public static void main(String[] args) {
        List<Dish> dish = Dish.getDish();
        //没有任何菜的热量大于等于1000
        boolean b = dish.stream().allMatch(d -> d.getCalories() >= 1000);
        System.out.println(b);
    }
}
```
`anyMatch`，`noneMatch`，`allMatch`这三个操作都用到了所谓的短路。

### 2.3.4 findAny
findAny方法将返回当前流中的符合过滤条件的任意元素。
```java
class FindAny{
    public static void main(String[] args) {
        List<Dish> dish = Dish.getDish();
        Optional<Dish> any = dish.stream().filter(Dish::isVegetarian).findAny();
        System.out.println("any = " + any);
    }
}
```

### 2.3.5 findFirst
findFirst方法能找到你想要的第一个元素。
```java
class FindFirst{
    public static void main(String[] args) {
        List<Dish> dish = Dish.getDish();
        Optional<Dish> any = dish.stream().filter(Dish::isVegetarian).findFirst();
        System.out.println("any = " + any);
    }
}
```
## 2.4 reduce 归约
此类查询需要将流中所有元素反复结合起来，得到一个值，比如一个 Integer 。这样的查询可以被归类为归约操作（将流归约成一个值）。用函数式编程语言的术语来说，这称为折叠（fold），因为你可以将这个操
作看成把一张长长的纸（你的流）反复折叠成一个小方块，而这就是折叠操作的结果。
### 2.4.1 元素求和
```java
public static void main(String[] args) {
    List<Integer> integers = Arrays.asList(1, 2, 3, 6, 8);
    //求list中的和，以0为基数
    Integer reduce = integers.stream().reduce(0, (a, b) -> a + b);
    //Integer的静态方法
    int sum = integers.stream().reduce(0, Integer::sum);
    System.out.println("reduce = " + reduce);
}
```

### 2.4.2 最大值和最小值
```java
public static void main(String[] args) {
    List<Integer> integers = Arrays.asList(1, 2, 3, 6, 8);
    Optional<Integer> min = integers.stream().reduce(Integer::min);
    System.out.println("min = " + min);
    Optional<Integer> max = integers.stream().reduce(Integer::max);
    System.out.println("max = " + max);
}
```

## 2.5 Collectors 收集器
### 2.5.1 查找流中的最大值和最小值 minBy maxBy
```java
public static void main(String[] args) {
    List<Dish> dish = Dish.getDish();
    //创建一个Comparator来进行比较  比较菜的卡路里
    Comparator<Dish> dishComparator = Comparator.comparingInt(Dish::getCalories);
    //maxBy选出最大值
    Optional<Dish> collect = dish.stream().collect(Collectors.maxBy(dishComparator));
    System.out.println("collect = " + collect);
    //选出最小值
    Optional<Dish> collect1 = dish.stream().collect(Collectors.minBy(dishComparator));
    System.out.println("collect1 = " + collect1);
}
```

### 2.5.2 汇总 summingInt
```java
 public static void main(String[] args) {
    List<Dish> dish = Dish.getDish();
    //计算总和
    int collect = dish.stream().collect(Collectors.summingInt(Dish::getCalories));
    System.out.println("collect = " + collect);
}
```

### 2.5.3 平均数 averagingInt
```java
public static void main(String[] args) {
    List<Dish> dish = Dish.getDish();
    //计算平均数
    Double collect = dish.stream().collect(Collectors.averagingInt(Dish::getCalories));
    System.out.println("collect = " + collect);
}
```

### 2.5.4 连接字符串 joining
```java
public static void main(String[] args) {
    List<Dish> dish = Dish.getDish();
    String collect = dish.stream().map(Dish::getName).collect(Collectors.joining());
    System.out.println("collect = " + collect);
}
```
joining 工厂方法有一个重载版本可以接受元素之间的分界符，这样你就可以得到一个逗号分隔的菜肴名称列表。
```java
String collect = dish.stream().map(Dish::getName).collect(Collectors.joining(", "));
```

### 2.5.5 得到流中的总数 counting
```java
long howManyDishes = dish.stream().collect(Collectors.counting());
```

## 2.6 分组
### 2.6.1 分组 groupingBy
```java
public static void main(String[] args) {
    List<Dish> dish = Dish.getDish();
    //groupingBy接受一个function作为参数
    Map<Dish.Type, List<Dish>> collect = dish.stream().collect(Collectors.groupingBy(Dish::getType));
    System.out.println("collect = " + collect);
}
```
如果想用以分类的条件可能比简单的属性访问器要复杂。例如，你可能想把热量不到400卡路里的菜划分为“低热量”（diet），热量400到700卡路里的菜划为“普通”（normal），高于700卡路里的划为“高热量”（fat）。由于Dish类的作者没有把这个操作写成一个方法，你无法使用方法引用，但你可以把这个逻辑写成Lambda表达式。
```java
public static void main(String[] args) {
    List<Dish> dishList = Dish.getDish();
    Map<String, List<Dish>> collect = dishList.stream().collect(Collectors.groupingBy(dish->{
        if (dish.getCalories() <= 400) {
            return "DIET";
        } else if (dish.getCalories() <= 700) {
            return "NORMAL";
        } else {
            return "FAT";
        }
    }));
    System.out.println("collect = " + collect);
}
```
### 2.6.2 多级分组
要实现多级分组，我们可以使用一个由双参数版本的Collectors.groupingBy工厂方法创建的收集器，它除了普通的分类函数之外，还可以接受collector类型的第二个参数。那么要进行二级分组的话，我们可以把一个内层groupingBy传递给外层groupingBy，并定义一个为流中项目分类的二级标准。
```java
public static void main(String[] args) {
    List<Dish> dish = Dish.getDish();
    Map<Dish.Type, Map<String, List<Dish>>> collect = dish.stream().collect(Collectors.groupingBy(Dish::getType, Collectors.groupingBy(d -> {
        if (d.getCalories() <= 400) {
            return "DIET";
        } else if (d.getCalories() <= 700) {
            return "NORMAL";
        } else {
            return "FAT";
        }
    })));
    System.out.println("collect = " + collect);
}
```
### 2.6.3 按子组收集数据
在上一面，我们看到可以把第二个groupingBy收集器传递给外层收集器来实现多级分组。但进一步说，传递给第一个groupingBy的第二个收集器可以是任何类型，而不一定是另一个groupingBy。

例如，要数一数菜单中每类菜有多少个，可以传递counting收集器作为groupingBy收集器的第二个参数。

```java
Map<Dish.Type, Long> typesCount = dish.stream().collect(groupingBy(Dish::getType, counting()));
```
普通的单参数groupingBy(f)（其中`f`是分类函数）实际上是 groupingBy(f,toList()) 的简便写法。

# 3 并行流
并行流就是一个把内容分成多个数据块，并用不同的线程分别处理每个数据块的流。这样一来，你就可以自动把给定操作的工作负荷分配给多核处理器的所有内核，让它们都忙起来。

## 3.1 将顺序流转为并行流
可以把流转换成并行流，从而让前面的函数归约过程（也就是求和）并行运行——对顺序流调用 parallel 方法:
```java
public static long parallelSum(long n) {
    return Stream.iterate(1L, i -> i + 1)
            .limit(n)
            .parallel()
            .reduce(0L, Long::sum);
}
```
Stream在内部分成了几块。因此可以对不同的块独立并行进行归纳操作，最后，同一个归纳操作会将各个子流的部分归纳结果合并起来，得到整个原始流的归纳结果。
![Stream并行流](/image/java8Stream.jpg)
类似地，你只需要对并行流调用 sequential 方法就可以把它变成顺序流。

看看流的parallel方法，你可能会想，并行流用的线程是从哪儿来的？有多少个？怎么自定义这个过程呢？
并行流内部使用了默认的ForkJoinPool，它默认的线程数量就是你的处理器数量，这个值是由Runtime.getRuntime().available-Processors()得到的。

但是你可以通过系统属性java.util.concurrent.ForkJoinPool.common.parallelism来改变线程池大小，如下所示：
```java
System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism","12");
```
这是一个全局设置，因此它将影响代码中所有的并行流。反过来说，目前还无法专为某个并行流指定这个值。一般而言，让ForkJoinPool的大小等于处理器数量是个不错的默认值，除非你有很好的理由，否则我们强烈建议你不要修改它。

## 3.2 分支/合并框架
分支/合并框架的目的是以递归方式将可以并行的任务拆分成更小的任务，然后将每个子任务的结果合并起来生成整体结果。它是ExecutorService接口的一个实现，它把子任务分配给线程池（称为ForkJoinPool）中的工作线程。
### 3.2.1 使用RecursiveTask
要把任务提交到这个池，必须创建RecursiveTask的一个子类，其中R是并行化任务（以
及所有子任务）产生的结果类型，或者如果任务不返回结果，则是RecursiveAction类型（当
然它可能会更新其他非局部机构）。
要定义RecursiveTask，只需实现它唯一的抽象方法compute ：
```java
protected abstract R compute();
```
这个方法同时定义了将任务拆分成子任务的逻辑，以及无法再拆分或不方便再拆分时，生成单个子任务结果的逻辑。

### 3.2.2 使用RecursiveTask求和
```java
public class ForkJoinSumCalculator
        extends java.util.concurrent.RecursiveTask<Long> {
    private final long[] numbers;
    private final int start;
    private final int end;
    public static final long THRESHOLD = 10_000;

    public ForkJoinSumCalculator(long[] numbers) {
        this(numbers, 0, numbers.length);
    }

    private ForkJoinSumCalculator(long[] numbers, int start, int end) {
        this.numbers = numbers;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        int length = end - start;
        if (length <= THRESHOLD) {
            return computeSequentially();
        }
        //创建一个子任务来为数组得前一半求和
        ForkJoinSumCalculator leftTask =
                new ForkJoinSumCalculator(numbers, start, start + length / 2);
        //利 用 另 一 个ForkJoinPool线程异步执行新创建的子任务
        leftTask.fork();
        //创建一个子任务来为数组得后一半求和
        ForkJoinSumCalculator rightTask =
                new ForkJoinSumCalculator(numbers, start + length / 2, end);
        //同步执行第二个子任务，有可能进一步递归
        Long rightResult = rightTask.compute();
        //读取第一个任务得结构，未完成就等待
        Long leftResult = leftTask.join();
        return leftResult + rightResult;
    }

    private long computeSequentially() {
        long sum = 0;
        for (int i = start; i < end; i++) {
            sum += numbers[i];
        }
        return sum;
    }

    public static long forkJoinSum(long n) {
        long[] numbers = LongStream.rangeClosed(1, n).toArray();
        ForkJoinTask<Long> task = new ForkJoinSumCalculator(numbers);
        return new ForkJoinPool().invoke(task);
    }

    public static void main(String[] args) {
        long l = ForkJoinSumCalculator.forkJoinSum(5);
        System.out.println("l = " + l);
    }
}
```
