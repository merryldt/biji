# 使用静态工厂方法替代构造方法
## 一、 类获取实例的方法
  1. 提供一个公共构造方法（传统）
  2. 提供一个公共静态工厂方法
## 二、 使用静态工厂方法的优劣
### 1 优点
####  1. 有自己的名字，构造方法没有;      
  - 如果构造方法的参数本身并不描述被返回的对象，有名称的静态工厂更易于使用，易于阅读;
  - 一个类只能有一个给定签名的构造方法。如果需要多个同样签名的构造方法，只能通过修改参数类型的顺序，造成识别的困难，此时可以使用静态工厂方法。
####  2. 与构造方法不同，不需要每次调用都创建一个新的对象;
  - 表现： 允许不可变的类使用预先构建的实例或者在构造时缓存实例，提高对象的使用率，从而提高性能；
  - 静态工厂方法 重复调用返回相同对象的能力 可以使 类在任何时候存在的实例都可以严格控制。这样的类称为实例控制。   
    实例是专门类的对象（实例即对象，实例是从类的角度考虑，对象是从程序的角度考虑）   
  - 实例控制的作用:    
    1. 允许一个类来保证它是一个单例项或不可实例化的;
       - 使一个类是单例有三种方法，静态工厂方法是其中一种，如下：   
       1 公共属性方法
         ```
              public class Elvis{
                 public static final Elvis INSTANCE = new Elvis();
                 private Elvis(){...}
                 public void leaveTheBuilding(){...}
              }
         ```    
         私有构造方法只调用一次，初始化公共静态final Elvis.INSTANCE 属性。没有公共的或受保护的构造方法，保证了全局的唯一性;   
         一旦Elvis被初始化，一个Elvis的实例就会存在，仅此一个。客户端做任何事情都无法改变。但是，如果特殊存在调用AccessibleObject.setAccessible()方法，   
         以反射方式调用私有构造方法，就需要额外处理，在请求创建第二个实例时抛出异常。      
        2  静态工厂方法 
         ```
              public class Elvis{
                 private static final Elvis INSTANCE = new Elvis();
                 private Elvis(){...}
                 public static Elvis getInstance(){return INSTANCE;}
                 public void leaveTheBuilding(){...}
              }
         ```   
         所有对Elvis.getInstance 的调用都返回相同的对象引用，并且不会创建其他的Elvis实例。   
        3 以上两种方法都是基于保持构造方法私有和导出公共静态成员以提供对唯一实例的访问。   
         第一种()的优点: 1 API明确表示该类是一个单例:公共静态属性是final的，故包含相同的对象引用；2 更简单。
         第二种: 1 灵活，无论该类是否为单例而不必更改其API;返回唯一的实例，但可修改。 2 如果程序需要，可以编写一个泛型单例工厂。3 方法引用可以用   
                 supplier,例如 Elvis :: instance 等同于 Supplier<Evlist> 。除非符合这三个优点，否则选公共属性方法。
        4 值得注意的是，第三种实现方法是使用枚举类
          ```
          public enum Elvis{
             INSTANCE;
             public void leaveTheBuilding(){...}
          }
          ```
         使用枚举实现的好处是: 是实现单例的最佳方式。提供了免费的序列化机制，并提供了针对多个实例化的坚固保证，即使是在复杂的序列化或反射攻击的情况下。   
         局限性: 如果单例必须继承Enum以外的父类，就不能使用。   
            
       -  使用私有构造方法实现类的非实例化   
          一般工具类，以此实现；比如 Math 或 Arrays ;
          ```
           public class UtilityClass{
              private UtilityClass(){
                throw new AssertionError();
              }
           }
          ```
          构造方法是私有的，在类外不可访问。AssertionError异常不是严格要求的，但是它防止了客户端在类中意外地调用构造方法。保证类在任何情况下都不会被实例化。   
          缺点是： 阻止了类的子类化。所有的构造方法都必须显式或隐式地调用父类构造方法，而子类则没有可访问的父类构造方法来调用。   
          值得注意的是: 通过创建抽象类来实现类的非实例化是不行的。抽象类是可以被子类化，子类可以被实例化。而且，误导用户改类的设计是为了继承。
    > 2  允许一个不可变的值类保证不存在两个相同的实例
#### 3、 与构造方法不同，可以返回其返回类型的任何子类型的对象
  1. 在选择返回对象的类时提供了很大的灵活性。
  - 应用： Api可以返回对象而不需要公开它的类。
  - 适用于： 基于接口的框架，接口为静态工厂方法提供自然返回类型。
  - 版本不同：  
    1. 1.8 之前   
       接口不能有静态方法。   
       通过静态工厂方法在一个非实例类中实现，返回对象的类都是非公开的。使用这种静态工厂方法需要客户端通过接口而不是实现类来返回引用的对象。
       一般来说，也应该优先使用接口而不是类来引用对象，唯一真正需要应用对象的类的时候是使用构造函数创建它的时候。   
    2. 1.8 及之后   
       接口可以包含静态方法，很多公开的静态成员应该放在这个接口本身。但是大部分实现代码还是应该放在单独的私有类中，因为   
       java8要求所有的静态成员都是公共的。java9 允许私有静态方法，但是静态字段和成员仍然需要公开。
       
#### 4、 返回对象的类可以根据输入参数的不同而不同
  1. 声明的返回类型的任何子类都是允许的，返回对象的类也是可以变化。   
   - 根据传入参数的不同，返回不同的值。比如：   
        ```
            public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
                Enum<?>[] universe = getUniverse(elementType);
                if (universe == null)
                    throw new ClassCastException(elementType + " not an enum");
        
                if (universe.length <= 64)
                    return new RegularEnumSet<>(elementType, universe);
                else
                    return new JumboEnumSet<>(elementType, universe);
            }
        ```   
     
   - 这个方法的调用，根据枚举类型的元素数量，返回不同的实例。
   - 实现类RegularEnumSet和JumboEnumSet 的存在对于客户端来说是不可见的，或者说，客户端也是不关心的。只需要知道是EnumSet的子类即可。
    
#### 5、 在编写包含该方法的类时，返回的对象的类不需要存在
   1. 静态工厂方法构成了服务提供者框架的基础，比如Java数据库连接Api(JDBC)。服务提供者框架是提供者实现服务的系统，并且系统使得实现对   
      客户端可用，从而将客户端从实现中分离出来。   
      - 服务提供者框架有三个基本组：
        1. 服务接口，表示实现；
        2. 提供者注册Api,提供者用来实现注册；
        3. 服务访问Api,客户端使用该Api获取服务的实例。
        - 值得注意的是 服务访问 API 允许客户指定选择实现的标准。在缺少这样的标准的情况下，API 返回一个默认实现的实例，或者允许客户通过所有可用的实现进行遍历。服务访问 API 是灵活的静态工厂，
        它构成了服务提供者框架的基础。
      - 服务提供者框架可以提供一个可选的组件做为第四个组件是：服务提供者接口；它描述了一个生成服务接口实例的工厂对象。在没有服务提供者接口的情况下，   
        必须对实现进行反射实例化。例子：   
        - 在jdbc的情况下，Connection 扮演服务接口的一部分，DriverManager.registerDriver 提供程序注册Api、DriverManager.getConnection 是服务访问Api,
        Driver是服务提供者接口。
        - 在Java6开始，平台提供一个通过的服务提供者框架 ServiceLoader，jdbc不用，因为jdbc出来的更早。
        - 依赖注入框架是很强大的服务提供者。
        - 服务访问 API 可以向客户端返回比提供者提供的更丰富的服务接口。称为桥接模式。
### 2  缺点
#### 1、没有公共或受保护的构造方法的类不能被子类化
   - 比如，在Collentions框架中，不能将实现类子类化。
#### 1、程序员很难找到，不突出
   - 解决，可以通过使用通用的命名减少这个问题。
## 三、总结   
   - 静态工厂方法和公共构造方法都有它们的用途，并且了解它们的相对优点是值得的。通常，静态工厂更可
取，因此避免在没有考虑静态工厂的情况下提供公共构造方法。