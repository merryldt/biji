#@JsonView 的使用
## 一、返回的结果直接使用实体类
- 代码如下
 ```
   @RequestMapping("api/user")
   @RestController
   public class UserApi {
   
       /**
        * 如果直接返回UserAdminView ，不需要在ResponseModel 中设置以下内容
        *   @JsonView(value = View.Base.class )
        *   private T data;
        * @return
        */
   
       @RequestMapping(value = "/listUser2",method = RequestMethod.POST)
       public UserAdminView listUser2(){
           UserAdminView userAdminView = new UserAdminView();
           return userAdminView;
       }
   }

 ```
 - 实体设置如下：
 ```
  
@Data
public class UserAdminView {

	public interface UserSimpView{};
	
	private Integer id;
	@JsonView(value = View.Base.class )
	private String username;
	@JsonView(value = View.Base.class )
	private String password;

	**@JsonView(value = View.Base.class )**
	private String note;
	@JsonView(value = View.Base.class )
	private Map<String,String> map;
	@JsonView(value = View.Base.class )
	private String []  ss;
	@JsonView(value = View.Base.class )
	private int [] intDemo;
	@JsonView(value = View.Base.class )
	private Integer b =null;
	@JsonView(value = View.Base.class )
	private boolean bbbb ;
	@JsonView(value = View.Base.class )
	private List<String> dd;
}

 ``` 
 - 实现控制
  ```
    public class View {
      public interface Base{};
    }

  ```
## 二、 使用自己的对象
 - 代码如下
   ```
     @RequestMapping("api/user")
     @RestController
     public class UserApi {
     
         /**
          * 自己设定了返回值的用这个
          * @return
          */
         @RequestMapping(value = "/listUser",method = RequestMethod.POST)
         public ResponseModel listUser(){
             UserAdminView userAdminView = new UserAdminView();
             return new ResponseModel(ResponseCode.OK,userAdminView);
         }
     }
   ``` 
 - ResponseModel 设置
   ```
    package com.json.demo.common;
    import com.fasterxml.jackson.annotation.JsonView;
    import java.io.Serializable;
    
   
    public class ResponseModel<T> implements Serializable {

      private static final long serialVersionUID = 1L;
      private int code;
      private String message;
    
      //重点是这里
      **@JsonView(value = View.Base.class )
      private T data;**
    
      public ResponseModel(int code, String message, T responseData) {
        this.code = code;
        this.message= message;
        this.data = responseData;
      }
       
    }

   ```  
  - 实体、view 和第一种一样。
