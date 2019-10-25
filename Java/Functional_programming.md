# 函数式接口  
## 定义   
1. 函数式接口就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口；
2. 函数式接口可以被隐式转换为Lambda表达式；
## 自定义函数式接口
```
 @FunctionalInterface
 interface GreetingService 
 {
     void sayMessage(String message);
 }   
```
## 函数式接口种类
### 1.8 之前已有的
```
    java.lang.Runnable
    java.util.concurrent.Callable
    java.security.PrivilegedAction
    java.util.Comparator
    java.io.FileFilter
    java.nio.file.PathMatcher
    java.lang.reflect.InvocationHandler
    java.beans.PropertyChangeListener
    java.awt.event.ActionListener
    javax.swing.event.ChangeListener
```
### 1.8 新增的
```
    java.util.function
```
-  function中包含很多方法。例子如Predicate的使用。
```
   public static void eval(List<Integer> list, Predicate<Integer> predicate) {
      // 方法一
      for(Integer n: list) {
         if(predicate.test(n)) {
            System.out.println(n + " ");
         }
      }
      //方法二  结合Stream 的过滤方法
      System.out.println("-----------------------------------------");
      list.stream().filter((name) -> (predicate.test(name))).forEach((name) ->{
         System.out.println(name);
      });
      //方法三  结合Stream 的过滤方法 ，在结合 方法引用
      System.out.println("-----------------------------------------");
      list.stream().filter((name) -> (predicate.test(name))).forEach(System.out::println);
   }
   public static void main(String[] args) {
      List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
      // 传递参数 n
      eval(list, n->true);
      // 传递参数 n
      eval(list, n-> n>3);
       
     //Predicate接口接受多个条件同时进行比较
      List<String> lists = Arrays.asList("Great works are accomplished not by strength but by perseverance.","Keep its original intention and remain unchanged");
      Predicate<String> starts = (name) -> name.startsWith("1");
      Predicate<String> lengths = (name) -> name.length() == 4;
      lists.stream().filter(starts.and(lengths)).forEach(System.out::println);
   }
```
  - 方法一使用的是常用的for循环遍历,方法二和方法三结合了Stream Api的过滤方法，及方法引用。
  ## github博客列表地址
  [github](https://github.com/florarose/biji)
  欢迎关注公众号，查看更多内容 ： 
  ![XG54_9_WXMH_5X_HB_H_7V](https://yqfile.alicdn.com/17479bd1026b3d93f5718893256adf7d6d164e5d.png)
  
