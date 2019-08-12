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
## Lambda结合FunctionalInterface Lib, forEach, stream()，method reference等新特性可以使代码变的更加简洁！