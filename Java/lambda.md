#
## Lambda 
- function包，提供lambda接口
   ```
       public interface Function<T, R> {
           /**
            * Applies this function to the given argument.
            *
            * @param t the function argument
            * @return the function result
            */
           R apply(T t);
       
           default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
               Objects.requireNonNull(before);
               return (V v) -> apply(before.apply(v));
           }
           ...
       }
   ```
- 除了default修饰的方法外，接口中只能有一个方法apply，这也是使用lambda接口的必要条件。
  表示给定一个输入T，返回一个输出R，使用lamdba接口时，我们使用表达式来表示实现接口的唯一方法apply()
  ```
      Function<Integer,Integer> function = (x)->{
        System.out.println("x : "+ String.valueof(x));
        return x+10;
      };
  ```
## Lambd 两种表现形式
- 定义源
  ```
    List<String> list = Arrays.asList ("a", "c", "A", "C");
  ```
- lambda表达式的另一种表现形式为 lambda方法引用
   ```   
       1）
       list.sort (String::compareToIgnoreCase);
       2）
       Predicate<String> p = String::isEmpty;
       3)
       Function<String, Integer> f1 = (String a) -> {return Integer.valueOf (a);};
       Integer result = f1.apply ("2");

   ```
-  函数编程,会被编译成一个函数式接口。
   ```
       1）
       list.sort ((s1, s2) -> s1.compareToIgnoreCase (s2));
       2）
       Predicate<String> q = (String a) -> {
           return a.isEmpty ();
       };
       3)
       Function<String, Integer> f2 = Integer::valueOf;
       // Function中的泛型 String代表返回类型，Integer代表输入类型，在lambda引用中会根据泛型来进行类型推断。
       Integer result = f2.apply ("2");
   ```
## Lambda 用途
- 只有一个抽象方法的函数式接口
    ```
     new Thread(new Runnable() {
       @Override
       public void run(){
          System.out.println("t1");
       } 
     }).start();
     
     new Thread( () -> run("t2") ).start();
     new Thread( () -> {
       Integer a = 2;
       System.out.println(a);
     }).start();
    ```
- 集合批量操作
    ```
        public class listLambda {
            public static void main(String[] args) {
                List<String> list = Arrays.asList("a","b","b");
                //foreach
                for (String  s : list
                     ) {
                    System.out.println(s);
                }
                System.out.println("-----------------");
                //lambda  用Iterable.forEach()取代foreach loop
                list.forEach((e) -> System.out.println(e));
            }
        }
    ```
- 流操作
    ```
        public class listLambda {
            public static void main(String[] args) {
                List<String> list = Arrays.asList("a","b","b");
                System.out.println(list.stream().filter((e) -> "b".equals(e)).count());
            }
        }
    ``` 
## 参考例子
```
 //在map集合下的使用
      Map<String,Object> map =new HashMap<String, Object>();
      map.put("A",1);
      map.put("B",2);
      //需要添加判断条件的
      map.keySet().forEach(name ->{
          if(name.startsWith("A")){
             System.out.println(map.get(name));
          }
      });
      System.out.println("-----------------------------------------");
      //不需要判断条件
      map.keySet().forEach(name ->{
            System.out.println(map.get(name));
      });
      //方法引用由::双冒号操作符标示
      System.out.println("-----------------------------------------");
      map.keySet().forEach( System.out::println);

      // list
      List future = Arrays.asList("Great works are accomplished not by strength but by perseverance.","Keep its original intention and remain unchanged");
      //lambda
      future.forEach(n -> System.out.println(n));
      //lambda 结合 方法引用
      future.forEach(System.out::println);
```
## 总结
1. lambda 表达式在java中也称为闭包或匿名函数；
2. lambda表达式 在编译器内部会被编译成私有方法；
3. lambda 外部变量，不能修改，只能访问。
![image](https://yqfile.alicdn.com/b5321b390e4e2178937e3d331f61d4a057c4f353.png)
![image](https://yqfile.alicdn.com/0c5f7fffab246cbc212f1bd98472065f0b23302c.png)
![image](https://yqfile.alicdn.com/d7eea5d9711e8cf292a1130703aa33d6e62ff386.png)

## Lambda结合forEach, stream()等新特性使代码更加简洁！
## github博客列表地址
[github](https://github.com/florarose/biji)
欢迎关注公众号，查看更多内容 ： 
![XG54_9_WXMH_5X_HB_H_7V](https://yqfile.alicdn.com/17479bd1026b3d93f5718893256adf7d6d164e5d.png)
