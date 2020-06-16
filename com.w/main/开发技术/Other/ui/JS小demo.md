# JS小页面跳转

## 1.页面跳转

### 1.1页面跳转(请求转发)

- 按钮的点击跳转

- 代码

- 1.下面界面跳转到servelt_refush

  ```Java
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta http-equiv="Content-Type" content="text/html" charset="UTF-8">
      <title>Start</title>
      <link rel="stylesheet" type="text/css" href="css/myStyle.css">
      <style type="text/css">
          .hello{
              text-align: center;
              font-size: 23px;
              font-family: Cambria;
          }
          .hello p{
              text-align: center;
              font-size: 20px;
              color: #ffc0cb;
          }
          .form_table table{
              margin:0px auto;
          }
      </style>
      <script type="text/javascript">
          function login() {
              // var but = document.getElementById("button");
              window.location.href="st.fo";
          }
      </script>
  </head>
  <body>
      <div id="form" class="form_table">
          <table>
              <tr>
                  <td>用户名：<input type="text" name="userName" value=""></td>
              </tr>
              <tr>
                  <td>密&nbsp;&nbsp;&nbsp;码：<input type="password" name="pwd"></td>
              </tr>
              <tr>
              <tr>
                  <td>记住用户名和密码：<input type="checkbox" name="remember"></td>
              </tr>
              <tr>
                  <td><input id="button" type="button" name="Submit" value="登陆" onclick=login()></td>
              </tr>
          </table>
      </div>
  </body>
  </html>
  ```

- servelt_refush代码（定时跳转至下一界面）

  ```java
  package com.it.w.test;
  
  import javax.servlet.ServletContext;
  import javax.servlet.ServletException;
  import javax.servlet.annotation.WebServlet;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  @WebServlet("/st.fo")
  public class servelt_refush extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          //-------------------------------------------
          //设置浏览器刷新
          //一秒一次
  //        resp.setIntHeader("refresh", 1);
  //        -----------------------------------
          resp.setContentType("text/html;charset=utf-8");
  
          resp.getWriter().write("注册成功，3秒后跳转登陆界面");
          resp.setHeader("refresh", "3;sel.to");
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          super.doPost(req, resp);
      }
  }
  
  ```

- 3.代码

  ```java
  package com.it.w.test;
  
  import javax.servlet.ServletException;
  import javax.servlet.annotation.WebServlet;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  @WebServlet("/sel.to")
  public class ht extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          resp.setContentType("text/html;charset=utf-8");
          resp.getWriter().write("请核对信息");
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          super.doPost(req, resp);
      }
  }
  
  ```

### 1.2.页面跳转（重定向）

- 跳转初始网页

  ```java
  package com.it.w.test;
  
  import javax.servlet.ServletException;
  import javax.servlet.annotation.WebServlet;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  @WebServlet("/te")
  public class servlet_reRetPoint extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          System.out.println("办理业务》》》");
          System.out.println("移送相关部门，任务交接中。。。");
          
  //        //告诉客户端重定向新的资源
  //        resp.setStatus(302);
  //        //告诉客户端应该去访问的资源
  //        resp.setHeader("location","task2");//响应给客户端，交由客户端处理
  
          resp.sendRedirect("task2");
          System.out.println("业务结束");
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req, resp);
      }
  }
  
  ```

- 目标跳转网页

  ```java
  package com.it.w.test;
  
  import javax.servlet.ServletException;
  import javax.servlet.annotation.WebServlet;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  @WebServlet("/task2")
  public class servlet_reRetPointDemo extends HttpServlet {
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          System.out.println("业务办理结束，反馈给上级部门");
      }
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
          doGet(req, resp);
      }
  }
  
  ```

  