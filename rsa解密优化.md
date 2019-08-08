## 一、 原来代码
 
   - 示例代码:
   
      ```
         private static org.bouncycastle.jce.provider.BouncyCastleProvider bouncyCastleProvider = null;
         public static  org.bouncycastle.jce.provider.BouncyCastleProvider getBouncyCastleProviderInstance() {
             if (bouncyCastleProvider == null) {
                 bouncyCastleProvider = new org.bouncycastle.jce.provider.BouncyCastleProvider();
             }
             return bouncyCastleProvider;
         }
     ```   
 ## 二、 优化后代码
 
   - 示例代码:
   
     ```
       private static  volatile org.bouncycastle.jce.provider.BouncyCastleProvider bouncyCastleProvider ;
       public static  org.bouncycastle.jce.provider.BouncyCastleProvider getBouncyCastleProviderInstance() {
           if (bouncyCastleProvider == null) {
               synchronized(org.bouncycastle.jce.provider.BouncyCastleProvider.class){
                   if(bouncyCastleProvider == null){
                       bouncyCastleProvider = new org.bouncycastle.jce.provider.BouncyCastleProvider();
                   }
               }
           }
           return bouncyCastleProvider;
       }
     ```  
## 三、实现原理
   
   - 添加synchronized关键字。添加到同步代码块中，而不是如下：
          
       ```     
       public static synchronized org.bouncycastle.jce.provider.BouncyCastleProvider getBouncyCastleProviderInstance() {
           if (bouncyCastleProvider == null) {
               bouncyCastleProvider = new org.bouncycastle.jce.provider.BouncyCastleProvider();
           }
           return bouncyCastleProvider;
        }
      ```  

   - 目的：即：保证在同一时刻最多只有一个线程执行该段代码，达到保证并发安全的效果。
## 四 、 synchronized
   
   - #### 定义
       -  synchronized锁是重量级锁，重量级体现在活跃性差一点。synchronized锁是内置锁，意味着JVM能基于synchronized锁做一些优化：比如增加锁的粒度(锁粗化)、锁消除。
       -  synchronized锁释放是自动的，当线程执行退出synchronized锁保护的同步代码块时，会自动释放synchronized锁。
       -  线程在竞争synchronized锁时是非公平的。
   - #### synchronized Java中的关键字，是一种同步锁。它修饰的对象有以下几种： 
       1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象； 
       2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象； 
       3. 修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象； 
       4. 修饰一个类，其作用的范围是synchronized后面括号括起来的部分，作用主的对象是这个类的所有对象。
   - #### 使用
   
      -  1、 普通同步方法，锁的是当前实例的对象
          
          - 同步方法锁住的是调用方法的对象；新创建的对象调用方法，和之前的方法不同步。
             ```
             class mYread implements Runnable {
                private static int count;
                public mYread() {
                   count = 0;
                }
                public synchronized  void method() {
                   for (int i = 0; i < 5; i ++) {
                      try {
                         System.out.println(Thread.currentThread().getName() + ":" + (count++));
                         Thread.sleep(100);
                      } catch (InterruptedException e) {
                         e.printStackTrace();
                      }
                   }
                }
                public synchronized void run() {
                   method();
                }
             }
             //执行调用
              mYread read1 = new mYread();
              mYread read2 = new mYread();
              Thread thread1 = new Thread(read1, "mYread1");
              Thread thread2 = new Thread(read2, "mYread2");
              thread1.start();
              thread2.start();
              
              read1 和 read2 是两个对象，thread1和thread2执行不会保持线程同步。
             ```  
      -  2、 静态同步方法，锁的是静态方法所在的类对象
          
          - 静态同步方法锁的是类的对象，新创建的对象也是被锁的对象。
          ```
          class mYread implements Runnable {
             private static int count;
             public mYread() {
                count = 0;
             }
             public synchronized static void method() {
                for (int i = 0; i < 5; i ++) {
                   try {
                      System.out.println(Thread.currentThread().getName() + ":" + (count++));
                      Thread.sleep(100);
                   } catch (InterruptedException e) {
                      e.printStackTrace();
                   }
                }
             }
             public synchronized void run() {
                method();
             }
          }
          //执行调用
           mYread read1 = new mYread();
           mYread read2 = new mYread();
           Thread thread1 = new Thread(read1, "mYread1");
           Thread thread2 = new Thread(read2, "mYread2");
           thread1.start();
           thread2.start();
           
           read1 和 read2 是两个对象，但在thread1和thread2并发执行时却保持了线程同步。
          ```  
               
      -   3、 同步代码块，锁的是括号里的对象。（此处的可以是实例对象，也可以是类的class对象。）
      
       1. 指定给某个对象加锁
            > 如果有一个明确的对象:
            
            ```
              public abstract class RSACoderUtils extends CoderUtils {
                 if (bouncyCastleProvider == null) { 
                     synchronized(org.bouncycastle.jce.provider.BouncyCastleProvider.class){
                         if(bouncyCastleProvider == null){
                             bouncyCastleProvider = new org.bouncycastle.jce.provider.BouncyCastleProvider();
                         }
                     }
                 }
              }
             ```
            > 如果没有一个明确的对象:
    
             ```   
                public abstract class RSACoderUtils extends CoderUtils {
                   if (bouncyCastleProvider == null) { 
                       private byte[] lock = new byte[0];  // 特殊的instance变量         
                       synchronized(lock){
                              if(bouncyCastleProvider == null){
                                  bouncyCastleProvider = new org.bouncycastle.jce.provider.BouncyCastleProvider();
                              }
                        }
                   }
                }            
             ```
             -. 比较两种创建对象的消耗：
              
             ``` 
                  Object lock = new Object()；
                  private byte[] lock = new byte[0]；
             ``` 
              
             通过查看编译后的字节码：生成零长度的byte[]的lock需要3行操作码，Object 的lock 则需要7行操作码,故零长度的byte数组对象创建性能更好。
        
       2. 给某个类加锁
         
             ```
                  public abstract class RSACoderUtils extends CoderUtils {
                     public void test(){
                         synchronized(RSACoderUtils.class){
                             if(bouncyCastleProvider == null){
                                 bouncyCastleProvider = new org.bouncycastle.jce.provider.BouncyCastleProvider();
                             }
                         }
                     }
                  }
             ```
## 五、volatile
   - 禁止指令重排序优化
   - 保证此变量对所有线程的可 见性，这里的“可见性”是指当一条线程修改了这个变量的值，新值对于其他线程来说是可以 立即得知的。
## 六、 单例模式
   - 保证一个类仅有一个实例，并提供一个访问它的全局访问点。
   - 懒汉模式 ：
   ```
   	public class Monitor {
   		   private static Monitor monitor = null;
   		   private static volatile Monitor monitor = null;   //
   		       private Monitor() {}
   			public static Monitor getMonitor() {
   				   if (monitor == null) {
   					       synchronized (Monitor.class) {
   					           if (monitor == null) {
   					               monitor = new Monitor();
   					           } 
   					       }
   				   }
   				   return monitor;
   			}
   }
   ```
   - 饿汉模式：
   ```
    public class Monitor {
           private static Monitor monitor = new Monitor ();
           private  Monitor () {}
           public static Monitor getMonitor() {
               return monitor;
           }
    }

   ```
