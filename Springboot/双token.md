# 双Token登录
## 研究背景
   项目使用验证码登录，token的有效期只有两个小时，会出现让用户反复登录的问题，为了解决，找到了双token的问题
## 一、使用背景，首先要说说token
   1. token的由来      
      token是在服务端产生，前端请求后端，请求成功，返回token给前端。前端之后的请求带上token,证明自己是合法请求.
   2. token 解决的问题   
      1. 完全由应用管理，可以避免开源策略；
      2. 无状态的，可以在多个服务间共享；
      3. 可以避免CSRF攻击
   3. token设置有效期的目的   
      - 目的
        从安全的角度考虑，避免token被非法截获后一直可用。
   4. 设置有效期后，带来的失效问题，解决方案
      1. 在服务端保存Token状态，前端每次请求更新Token的过期时间; 但是在请求频繁的情况下，代价很大 ；
      2. 使用Refresh_token（过期时间更长）， 避免频繁的读写操作。（参考微信网页认证token的写法），也就是双token认证方法。
## 二、双Token 登录方法
   1. 目的
      为了让用户在使用过程中感觉不到token失效的问题。
   2. 设计
      ![image](https://yqfile.alicdn.com/401cfaf1b7110ff1b7564fc067743ef0f2068527.png)
   3. 优点
      ![image](https://yqfile.alicdn.com/7d9dde77aa2c7ce85eae90a085200776df62cb74.png)

## 三、双Token登录的时序图
   ![image](https://yqfile.alicdn.com/c09771117aa29f17e24fa71dddcf3a8bcc5e69e9.png)
## 四、文字的逻辑图
  ![image](https://yqfile.alicdn.com/3c7aba19cf7bb6b0c48a3ea6c69a2142d4241958.png)
## 博客都整理在github上，欢迎访问，还有一些常用的工具和学习源码
![image](https://yqfile.alicdn.com/2b964f58919279c8d4bcaafed1941905f9cecce4.png)
![image](https://yqfile.alicdn.com/e2dc5163b3f14c12ac5a79e196cd21cba0d6988d.png)
[github](https://github.com/florarose/biji)

####  如有不对，多多指教