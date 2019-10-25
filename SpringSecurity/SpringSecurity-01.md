## 一、获取用户信息的两种方式
![image](https://yqfile.alicdn.com/3135bf9913fdf858817e568f26239a22f3252590.png)
## 二、重点讲解使用Java方式获取用户信息，讲解其核心组件
#核心组件：
## 1 定义
1. 称为shared的组件，是指它在框架中占有很重要的位置，框架离开它无法运行。
2. 内置的一系列的过滤器中都用到了这些共享组件。
3. 这些Java类表达了维持系统的构建代码块。

## 1 SecurityContextHolder
1. 最基础的对象，当前应用程序的当前安全环境的细节存储在其中。
2. 默认情况下，SecurityContextHolder使用ThreadLocal存储这些信息，如此意味着安全环境在同一个线程执行的方法一直是有效的。
3. 我们把安全主体和系统交互的信息都保存在SecurityContextHolder中。
   ```
          private static SecurityContextHolderStrategy strategy;
          public static SecurityContext getContext() {
              return strategy.getContext();
          }

   ```  
##  2 SecurityContext
1. 上下文细节，以接口的形势表示。
   ```
       public interface SecurityContext extends Serializable {
           Authentication getAuthentication();
           void setAuthentication(Authentication var1);
       }
   ```
2. 其实现类为SecurityContextImpl
##  3 Authentication
1. 在认证请求时用到是一个接口。我们把安全主体和系统交互的信息都保存在SecurityContextHolder 中了。 Spring Security 使用一个 Authentication 对应来表现这些 信 息 。 虽 然 你 通 常 不 需 要 自 己 创 建 一 个 Authentication 对 象 ， 直 接 通 过SecurityContextHolder 获取上下文对象，然后通过上下文对象（SecurityContext）获取即可.引用《Pro Spring Security》中的一段话，An Authentication object is used both when an authentication request is created (when a user logs in),to carry around the different layers and classes of the framework the requesting data, and then when it is validated,containing the authenticated entity and storing it in SecurityContext.The most common behavior is that when you log in to the application a new Authentication object will be created storing your user name, password, and permissions—most of which are technically known as Principal, Credentials, and Authorities, respectively.翻译过来就是说，在创建身份验证请求（用户登录时）时，将使用一个Authentication对象，以携带请求数据的框架的不同层和类，然后在验证数据时将其包含验证的实体并将其存储在SecurityContext中。最常见的行为是，当您登录到应用程序时，将创建一个新的身份验证对象（Authentication）存储您的用户名，密码和权限-在技术上大多数依次称为Principal(主体，即用户)，Credentials（凭证）和 Authorities(权限)
##  4 UserDetails
1.  UserDetails 是一个 Spring Security 的核心接口。 它代表一个主体（包含于用户相关的信息）。
     在 Authentication 接口中有一个方法 Object getPrincipal(); 这个方法返回的是一个
     安全主题，大多数情况下，这个对象可以强制转换成 UserDetails 对象，获取到
     UerDetails 对象之后，就可以通过这个对象的 getUserName()方法获取当前用户名。
2.  我们可以定义类实现这个接口，保存我们想要的信息，并把它保存在Authentication中。
## 三、 流程
  SecurityContextHolder通过SecurityContextHolderStrategy的三个实现类中的一个来保存SecurityContext（即上下文），也就是Authentication（认证信息），
  用户认证信息包括但不限于：
  1. 用户权限集合
  2. 用户名、密码
  3. 是否已认证成功
  ...等
  ,通过getContext()获取。
  
## 四、案例地址，简单的demo
[Spring.security案例地址](https://github.com/florarose/spring.security)