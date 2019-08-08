## 一、 定义
   -  Java虚拟机提供的最轻量级的同步机制。
## 二、通过volatile关键字修饰后，具备两种特性
   -  保证此变量对所有线程的可见性，这里的“可见性”是指当一条线程修改了这个变量的值，新值对于其他线程来说是可以 立即得知的。
      > 普通变量不能做到这一点，普通变量的值在线程间传递均需要通过*主内存*来完成，例如，线程A修改一个普通变量的值，
        然后向主内存进行回写，另外一条线程B在线程A回写完成了之后再从主内存进行读取操作，新变量值才会对线程B可见。
   -  禁止指令重排序优化。
      + 通过一个单例模式来讲解，代码如下:  
      ```
         public class Monitors {
                private static volatile Monitors monitors = null; 
                private Monitors() {}
                 public static Monitors getMonitor() {
                        if (monitor == null) {
                                synchronized (Monitors.class) {
                                    if (monitors == null) {
                                        monitors = new Monitors();
                                    } 
                                }
                        }
                        return monitors;
                 }
         }
      ```    
      +  分析代码   
      
         +  monitors = new Monitors();，在这个操作中，JVM主要做三件事：   
       1、在java 堆空间里分配一部分空间；   
       2、执行 Monitor 的构造方法进行初始化；   
       3、把 monitor 对象指向在堆空间中分配好的空间。      
         三步执行完后，这个 monitor 对象就不是空值。      
         + 普通变量不能保证变量赋值操作的顺序与程序代码中的执行顺序一致，指令重排序的优化很可能改变程序的执行顺序。比如，执行顺序可能为
       1、2、3，也可能为1、3、2。如果是按照1、3、2的顺序执行，恰巧在执行到3的时候（还没执行2），新的线程执行 getMonitor() 方法之后判断 
       monitor 不为空就返回了 monitor 实例。此时 monitor 实例虽有值， 但它没有执行构造方法进行初始化（即没有执行2），
       故该线程如果对那些需要初始化的参数进行操作就会发生错误。   
           但是加volatile 关键字的话，就不会出现这个问题。禁止指令重排序优化。 
            
      +  分析class字节码文件得知  
         + 有volatile修饰的变量，赋值后多执行了一个“lock addl ＄0x0，（%esp）”操作，这个操作相当于一个内存屏障
        （Memory Barrier或Memory Fence，指重排序时不能把后 面的指令重排序到内存屏障之前的位置），
        只有一个CPU访问内存时，并不需要内存屏障；但如果有两个或更多CPU访问同一块内存，且其中有一个在观测另一个，就需要内存屏障来保证一致性了。
       
         + lock前缀，查询IA32手册，它的作用是使得本CPU的Cache写入了内存，该写入动作也会引起别的CPU或者别的内核无效化（Invalidate）其Cache，
       这种操作相当于对Cache中的变量做了一次前面介绍Java内存模式中所说的“store和write”操作。
       所以通过这样一个空操作，可让前面volatile变量的修改对其他CPU立即可见。
       store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存 中，以便随后的write操作使用。 
       write（写入）：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入 主内存的变量中。
      + 那为何说它禁止指令重排序呢？  
      
         + 从硬件架构上讲，指令重排序是指CPU采用了允许将多条指令不按程序规定的顺序分开发送给各相应电路单元处理。但并不是说指令任意重排，
      CPU需要能正确处理指令依赖情况以保障程序能得出正确的执行结果。
         + 譬如指令1把地 址A中的值加10，指令2把地址A中的值乘以2，指令3把地址B中的值减去3，这时指令1和指 令2是有依赖的，它们之间的顺序
      不能重排——（A+10）*2与A*2+10显然不相等，但指令3 可以重排到指令1、2之前或者中间，只要保证CPU执行后面依赖到A、B值的操作时能获取到 正确的A和B值即可。
      所以在本内CPU中，重排序看起来依然是有序的。
         + 因此，lock addl＄0x0，（%esp）指令把修改同步到内存时，意味着所有之前的操作都已经执行完成， 这样便形成了“指令重排序无法越过内存屏障”的效果。
          
  
  
## 三、 使用条件
  
   1. 满足如下两个条件：   

      +  运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。
      +  变量不需要与其他的状态变量共同参与不变约束。
   2. 通过加锁（使用synchronized或java.util.concurrent中的原子类）
      
   -  我的理解就是: 保证操作是原子性操作。
   
   #### 原因,在java虚拟机中解释：
   
   -  volatile变量在各个线程的工作内存中不存在一致性问题（在各个线程 的工作内存中，volatile变量也可以存在不一致的情况，
      但由于每次使用之前都要先刷新，执行引擎看不到不一致的情况，因此可以认为不存在一致性问题），但是Java里面的运算并非原子操作，
      导致volatile变量的运算在并发下一样是不安全的。
      
   1. 通过一段简单的演示，代码清单如下：    
      
   ```
        public class Volatile_test {
            public static volatile int race = 0;
            public static void increase(){
                race ++;
            }
            private static final int THREADS_COUNT = 20;
            public static void main(String [] args ){
        
                Thread [] threads = new Thread[THREADS_COUNT];
                for(int i=0;i<THREADS_COUNT;i++)
                {
                    threads[i] = new Thread( new Runnable(){
        
                        @Override public void run(){
                        for(int i =0 ; i<10000 ; i++)
                        {
                            increase();
                        }}
                    });
                    threads[i].start();
                }
                //等待所有累加线程都结束
                while(Thread.activeCount() > 1)
                 Thread.yield();
                System.out.println(race);
            }
        }
   ```   
   2. 结果分析,这段代码开启20个线程，每个线程对race的自增做10000次操作，理论上输出的race为200000，但是实际运行结果总是小于200000。
   3. 推测问题出现在race ++ 这行代码中，通过反编译这个类得到代码清单，发现只有一行代码的increase（）方法在Class文件中是由4条字节码指令构成的。   
      反编译字节码如下：
      ```
           * public static void increase（）；
           * Code：
           * Stack=2，Locals=0，Args_size=0
           * 0：getstatic # 13； //Field race：I
           * 3：iconst_1
           * 4：iadd
           * 5：putstatic # 13；//Field race：I
           * 8：return LineNumberTable：
           * line 14：0
           * line 15：8
      ```
      通过字节码层次分析：    
      
        + 当getstatic指令把race的值取到操作栈顶时，volatile关键字保证了race的值在此 时是正确的，但是在执行iconst_1、iadd这些指令的时候，
         
      其他线程可能已经把race的值加大，而在操作栈顶的值就变成了过期的数据，所以putstatic指令执行后就可能把较小的race值同步回主内存之中。
## 四、 volatile和锁的比较
   - 在某些情况下，volatile的同步机制的性能确实要优于锁（使用synchronized关键字或java.util.concurrent包里面的锁），
      但是由于虚拟机对锁实行的许多消除和优化，使得我们很难量化地认为volatile就会比synchronized快多少；
   - volatile变量读操作的性能消耗与普通变量几乎没有什么差别，但是写操作则可能会慢一些，因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行。
   - 大多数场景下volatile的总开销要比锁低；
   - 在volatile与锁之中选择的唯一依据仅仅是volatile的语义能否满足使用场景的需求。


      
  
