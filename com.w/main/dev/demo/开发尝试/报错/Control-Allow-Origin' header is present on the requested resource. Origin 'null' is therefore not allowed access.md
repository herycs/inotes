# Control-Allow-Origin' header is present on the requested resource. Origin 'null' is therefore not allowed access

# 1.问题背景

> 练手过程中，尝试使用前后端分离的技术，但不幸测试界面使用时便出错

# 2.问题原因

- 跨域请求，后端不允许

  > 可能原因，你的后台代码中没有配置相关设置，导致其默认阻止其他域的访问

- 我的后台使用的是springMVC模式

# 3.解决办法

- 方法一，给后端加过滤器，设置相关属性

- 方法二，我的处理办法是在控制器中添加注解，@CrossOrigin

  ```java
  @CrossOrigin
  @org.springframework.stereotype.Controller
  public class UserController implements Controller {  ...  }
  ```