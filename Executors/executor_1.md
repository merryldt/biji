# 线程池的初步认识(一)
## 一、定义
- 管理一组工作线程。 
## 二、作用
- 可以限制应用程序中同一时刻运行的线程数;
- 可以应用在多线程服务器上。
## 三、好处
1. 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗,比如内存;
2. 提高响应速度。创建线程再到执行任务的额外时间，会延迟处理的请求;
3. 提高线程的客观理性。通过线程池，实现对线程的统一分配，调优和监控;比如可以避免无线创建线程引起的OutOfMemoryError。
## 四、java 5 在 java.util.concurrent 中内置线程池 Excutors,返回对象为ThreadPoolExecutor,包含三种创建的方法： 
   ```
   ExecutorService executorService1 = Executors.newSingleThreadExecutor();
   ExecutorService executorService2 = Executors.newFixedThreadPool(10);
   ExecutorService executorService3 = Executors.newScheduledThreadPool(10);
   ```
## 五、执行
### (一)、 通过execute
    ```
    executorService.execute(new Runnable() {
        @Override
        public void run() {
            System.out.println("我要开始运行了");
        }
    });
    ```
### (二)、 通过submit
#### 1.通过提交Runnable
    ```
    Future future =  executorService.submit(()->{
    });
    ```
#### 2.通过提交Callable
    ```
    Future future =  executorService.submit(() -> {
        return null;
    });
    ```
#### Runnable 和 Callable 的区别
1. 非常相似，这两个接口都表示可以由线程或ExecutorService同时执行的任务。
2. 不同之处是两个接口内部执行的方法不同。   
- Runnable
    ```
    public interface Runnable {
        public void run();
    }
    ```
- Runnable
    ```
    public interface Callable{
        public Object call() throws Exception;
    }
    ```    
- 可以看出,call()方法是有返回值的，而且可以引发异常。run没有返回值，也不能引发异常（除非未经检查的异常-RuntimeException的子类)。   
 
3.如果执行的任务需要返回结果，使用Callable。

### (三)、两种执行方法的区别
1. execute 只能提交Runnable类型的任务；submit 除了可以提交Runnable类型的任务外，还可以提交Callable类型。
2. execute 直接抛出任务执行的异常，submit 会捕获，可以通过返回值Future的get方法将执行任务时的异常重新抛出。
3. submit 的顶级接口是ExecutorService；execute 的顶级接口是 Executor；
## 六、获取返回结果
### （一）、 两种 invokeAny() 和 invokeAll()
### （二）、 invokeAny 
1. 当任意一个任务得到结果后，会调用interrupt方法将其他的任务中断;
2. 部分任务失败，会使用第一个成功的任务返回的结果;
3. 任务全部失败了，抛出Execption,invokeAny 方法将抛出ExecutionException。
### （三）、 invokeAll 返回所有任务的执行结果，该方法的执行效果也是阻塞执行的，要把所有的结果都取回时再继续向下执行。

## 七、关闭 ExectorService 
### shutdown()
1. 停止接收新的任务并且等待已经提交的任务（包含提交正在执行和提交未执行）执行完成。在终止前允许执行以前提交的任务；
### shutdownNow() 
1. 阻止等待任务的启动并试图停止当前正在执行的任务。不允许执行以前提交的任务。
### awaitTermination()
1. 接收timeout和TimeUnit两个参数，用于设定超时时间及单位。当等待超过设定时间时，会监测ExecutorService是否已经
   关闭，若关闭则返回true，否则返回false。一般情况下会和shutdown方法组合使用。
### 在实际使用过程中, 使用shutdown()关闭，回收资源。如果有必要，可以在其后执行shutdownNow(),取消所有遗留的任务。
## 八、例子
```
ExecutorService executorService = Executors.newSingleThreadExecutor();
Set<Callable<String>> callables = new HashSet<Callable<String>>();
callables.add(() -> {
        return "Task 1";
    }
});
callables.add(() -> {
        return "Task 2";
    }
});
List<Future<String>> futures = executorService.invokeAll(callables);
for(Future<String> future : futures){
    System.out.println("future.get = " + future.get());
}
executorService.shutdown();
while (!service.awaitTermination(1, TimeUnit.SECONDS)) {
         System.out.println("线程池没有关闭");
      }
```
## github博客列表地址
  [github](https://github.com/florarose/biji)
## 欢迎关注公众号，查看更多内容 ： 
  ![XG54_9_WXMH_5X_HB_H_7V](https://yqfile.alicdn.com/17479bd1026b3d93f5718893256adf7d6d164e5d.png)

