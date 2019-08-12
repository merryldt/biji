#
##  Stream 允许我们以声明的方式处理数据。
## 简介
- 在 Java 中，集合和数组是两种常见的数据结构
- 类似于 SQL 语句从数据库查询数据的形式，Stream 提供了对 Java 集合操作和表示的高度抽象。
- 要处理的元素集合被视为流，在流水线中进行传输。并可在流水线各节点处理这些元素，例如过滤，排序和聚合。
## 特点
- 不占用空间。Stream 只是数据源的视图，表现形式可以是数组、容器或者I/O通道。
- 流操作数据源，不会改变数据源。 例如： 过滤Stream后不会删除过滤的元素，而是生成一个新的不包含过滤元素的Stream
- 懒加载。 对Stream的操作只有在需要的时候才会执行。
- 不可重复消费。 在流的生命周期中，元素仅能被访问一次。
## 操作
### &emsp;流的创建
- 集合创建
    ```
        List<String> list = Arrays.asList("Hello", "Word", "!");
        Stream<String> stream = list.stream();
    ``` 
- Stream自带的方法
    ```
       Stream<String> streams = Stream.of("Hello", "Word", "!");
    ```
### &emsp;中间操作
- 多个中间操作组合形成流水线，中间操作返回的是一个新的Stream.
- filter
    ```
       System.out.println(streams.stream().filter((e) -> "Hello".equals(e)));
       strings.stream().filter(e -> !e.isEmpty()).forEach(System.out::println);
    ```
- 映射 
  + map(Function<T, R> f)：接收一个函数作为参数，该函数会被应用到流中的每个元素上，并将其映射成一个新的元素。
  + flatMap(Function<T, Stream<R>> mapper)：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流
    ```
       List<String> numbers = Arrays.asList("a","b");
       numbers.stream().map(i -> i + i).forEach(System.out::println);
       Stream<List<String>> stream2 = Stream.of(Arrays.asList("H","E"), Arrays.asList("L", "L", "O"));
       stream2.flatMap(list -> list.stream()).forEach(System.out::println);
    ```
- limit/skip
  + limit 返回 Stream 中的前 N 个元素。skip 舍弃 Stream 中的前 N 个元素。
  ```
       List<String> numbers = Arrays.asList("a","b");
       numbers.stream().limit(1).forEach(System.out::println);
       numbers.stream().skip(1).forEach(System.out::println);
  ```
- 排序
  + sorted()：自然排序使用Comparable<T>的int compareTo(T o)方法
  + sorted(Comparator<T> com)：定制排序使用Comparator的int compare(T o1, T o2)方法
  ```
      List<String> numbers = Arrays.asList("b","a");
      numbers.stream().sorted().forEach(System.out::println);
      numbers.stream().sorted((x,y) -> y.compareTo(x)).forEach(System.out::println);
  ```
- distinct
  + 去重
  ```
      List<String> numbers = Arrays.asList("b","a","b");
      numbers.stream().distinct().forEach(System.out::println);
  ```
- reduce 
  + 累加
  ```
      System.out.println("=====reduce:将流中元素反复结合起来，得到一个值==========");
      Stream<Integer> stream = Stream.iterate(1, x -> x+1).limit(200);
      //stream.forEach(System.out::println);
      Integer sum = stream.reduce(10,(x,y)-> x+y);
      System.out.println(sum);
  ```
### &emsp;终端操作
- 返回值不为Stream 的为终端操作(立即求值)，终端操作不支持链式调用，会触发实际计算
- 执行过末端操作以后，Stream 不可再次使用，且不允许执行任何中间操作。
- forEach
  + 遍历
  ```
    List<String> numbers = Arrays.asList("b","a","b");
    numbers.stream().forEach(System.out::println);
  ```
- count
  + 统计个数
  ```
    List<String> numbers = Arrays.asList("b","a");
    numbers.stream().count();
  ```
- collect
  + 组合，返回对应的类型,包括list,set,treeset ,map 等
  ```
    List<String> numbers = Arrays.asList("b","a");
    numbers = numbers.stream().sorted().collect(Collectors.toList());
   List<lambdaDemo> lambdaDemos = Arrays.asList(new lambdaDemo("zhang","bing",1),
            new lambdaDemo("li","hua",2));
    Map<Integer,String> maps = lambdaDemos.stream().collect(Collectors.toMap(lambdaDemo::getAge,lambdaDemo::getName));
    System.out.println(maps);
    //以年龄为唯一值，
    public static  Map<Integer,lambdaDemo> check(List<lambdaDemo> lambdaDemos){
        return   lambdaDemos.stream().collect(Collectors.toMap(lambdaDemo::getAge, Function.identity(),
                (existing, replacement) -> existing));
    }
  ```
- 分组和分区
  + Collectors.groupingBy()对元素做group操作。
  + Collectors.partitioningBy()对元素进行二分区操作。
  ```
      public static  void test9() {
          List<lambdaDemo> lambdaDemos = Arrays.asList(new lambdaDemo("zhang","黑人",65),
                  new lambdaDemo("li","黑人",40),
                  new lambdaDemo("liu","白人",40));
          System.out.println("=======根据人的肤色进行分组==========================");
          Map<String, List<lambdaDemo>> map = lambdaDemos.stream().collect(Collectors.groupingBy(lambdaDemo::getName));
          System.out.println(map);
          System.out.println("=======根据人的年龄范围多级分组==========================");
          Map<Integer, Map<String, List<lambdaDemo>>> map2 = lambdaDemos.stream().collect(Collectors.groupingBy(lambdaDemo::getAge,
                  Collectors.groupingBy(
                     ( p ) -> {
                          if ( p.getAge() > 60 ) {
                              return "老年人";
                          } else {
                              return "年轻人";
                          }
                     }
                  )
          ));
          System.out.println(map2);
      }
      public static void test10()  {
          List<lambdaDemo> lambdaDemos = Arrays.asList(new lambdaDemo("zhang","黑人",60),
                  new lambdaDemo("li","黑人",2),
                  new lambdaDemo("liu","白人",3));
          System.out.println("========根据人的年龄是否大于1000进行分区========================");
          Map<Boolean, List<lambdaDemo>> map = lambdaDemos.stream().collect(Collectors.partitioningBy(p -> p.getAge() > 40));
          System.out.println(map);
      }
  ```
