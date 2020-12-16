# The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.

## 1.这是问题：

> 源服务器找不到目标资源的当前表示，或者不愿意公开该表示存在
>

- 我的解决方法是：将路径写成web.xml的映射，或者注解

- 例如：

  > 我的程序是：让do1转发给do2
  >
  > ### do1代码：

  ```java
  package com.it.w.test;
  
  import javax.servlet.ServletContext;
  import javax.servlet.ServletException;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  public class servlet_do1 extends HttpServlet {
  
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          System.out.println("业务不对口，转发中。。。");
          ServletContext application = this.getServletContext();
  
  //        RequestDispatcher requestDispatcher = application.getRequestDispatcher("/test/servlet_do2");
  //        requestDispatcher.forward(req, resp);
          application.getRequestDispatcher("/servlet_do2").forward(req, resp);
  
          System.out.println("业务结束。");
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req,resp);
      }
  }
  ```

  

  > ### do2代码：

  ```java
  package com.it.w.test;
  
  import javax.servlet.ServletException;
  import javax.servlet.annotation.WebServlet;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  @WebServlet("/servlet_do2")
  public class servlet_do2 extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          System.out.println("业务办理结束！   ");
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          super.doPost(req, resp);
      }
  }
  
  ```

  > ### 解决办法：我使用的是注解，将do1中的路径改为do2的映射即可

  



