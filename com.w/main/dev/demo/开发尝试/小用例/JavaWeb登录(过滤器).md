# Servlet 登录

> (过滤器简单示例）

# 1.目标功能

- 用户登录后分别授予不同目录资源的访问权限

# 2.设计

![image-20191211194715247](C:\Users\ANGLE0\AppData\Roaming\Typora\typora-user-images\image-20191211194715247.png)

# 3.架构

- 项目类型：Java Web

- 目录结构：
  - ![image-20191211194913586](C:\Users\ANGLE0\AppData\Roaming\Typora\typora-user-images\image-20191211194913586.png)

# 4.编码

## 4.1 后台

### 4.1.1 模块代码

- loginServlet.java

  ```java
  package com.z.servlet;
  
  import com.z.domain.User;
  
  import javax.servlet.ServletException;
  import javax.servlet.annotation.WebServlet;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  
  /**
   * @ClassNameloginServlet
   * @Description
   * @Author ANGLE0
   * @Date2019/12/11 18:54
   * @Version V1.0
   **/
  
  @WebServlet("/login")
  public class loginServlet extends HttpServlet {
  
      @Override
      protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
          String name = req.getParameter("username");
          String passwd = req.getParameter("pass");
  
          System.out.println(name+ passwd);
  
          User u = new User(name, passwd);
          String to  = null;//前往的地址
  
          if (u.getName() != null && u.getPassword() != null) {
              if(u.getName().equals("aaa") && u.getPassword().equals("aaa")){
                  u.setPermission("A");
                  to = "/list.jsp";
                  req.getSession().setAttribute("user", u.getName());
                  req.getSession().setAttribute("role", u.getPermission());
              }else if(u.getName().equals("bbb") && u.getPassword().equals("bbb")){
                  u.setPermission("B");
                  to = "/list.jsp";
                  req.getSession().setAttribute("user", u.getName());
                  req.getSession().setAttribute("role", u.getPermission());
              }else{
                  to = "/login.jsp";
              }
          }
          System.out.println("登录失败,回登录页");
          resp.sendRedirect(to);
      }
  
      @Override
      protected void doGet(HttpServletRequest req, HttpServletResponse resp)  throws IOException, ServletException {
          this.doPost(req, resp);
      }
  }
  
  ```

  ```java
  //上例中，以下两句解释
  //用户登录后，user的值不为空，以此判断用户是否以登录，role判断是否有权限
  req.getSession().setAttribute("user", u.getName());
  req.getSession().setAttribute("role", u.getPermission());
  ```

- Filter_A.java

  ```java
  package com.z.Filter;
  
  import com.z.domain.User;
  
  import javax.servlet.*;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  /**
   * @ClassNameFilter_A
   * @Description
   * @Author ANGLE0
   * @Date2019/12/11 18:48
   * @Version V1.0
   **/
  public class Filter_A implements Filter {
      @Override
      public void init(FilterConfig filterConfig) throws ServletException {
      }
  
      @Override
      public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
          HttpServletRequest request = (HttpServletRequest) servletRequest;
          HttpServletResponse response = (HttpServletResponse) servletResponse;
  
          String user = (String) request.getSession().getAttribute("user");
  
          if(user != null){
              String per = (String)request.getSession().getAttribute("role");
              if(per.equals("A")){
                  filterChain.doFilter(request, response);
              }else{
                  response.sendRedirect("/list.jsp");
              }
              return;
  
          }
          response.sendRedirect("/login.jsp");
          return;
      }
  
      @Override
      public void destroy() {
      }
  }
  ```

- Filter_B.java

  ```java
  package com.z.Filter;
  
  import com.z.domain.User;
  import org.omg.CosNaming.NamingContextExtPackage.StringNameHelper;
  
  import javax.servlet.*;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.IOException;
  
  /**
   * @ClassNameFilter_B
   * @Description
   * @Author ANGLE0
   * @Date2019/12/11 18:49
   * @Version V1.0
   **/
  public class Filter_B implements Filter {
      @Override
      public void init(FilterConfig filterConfig) throws ServletException {
      }
  
      @Override
      public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
          HttpServletRequest request = (HttpServletRequest) servletRequest;
          HttpServletResponse response = (HttpServletResponse) servletResponse;
  
          String user = (String) request.getSession().getAttribute("user");
  
          if(user != null){
              //用户已登录，判断是否有权限
              String per = (String)request.getSession().getAttribute("role");
              if(per.equals("B")){
                  filterChain.doFilter(request, response);
              }else{
                  response.sendRedirect("/list.jsp");
              }
              return;
          }
          response.sendRedirect("/login.jsp");
          return;
      }
  
      @Override
      public void destroy() {
      }
  }
  ```

- User.java

  ```java
  package com.z.domain;
  
  /**
   * @ClassNameUser
   * @Description
   * @Author ANGLE0
   * @Date2019/12/11 18:48
   * @Version V1.0
   **/
  public class User {
  
      private String name;
      private String password;
      private String permission;
  
      public User(String name, String password) {
          this.name = name;
          this.password = password;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getPassword() {
          return password;
      }
  
      public void setPassword(String password) {
          this.password = password;
      }
  
      public String getPermission() {
          return permission;
      }
  
      public void setPermission(String permission) {
          this.permission = permission;
      }
  }
  ```

### 4.1.2 配置文件

- web.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
  
      <filter>
          <filter-name>a</filter-name>
          <filter-class>com.z.Filter.Filter_A</filter-class>
      </filter>
      <filter-mapping>
          <filter-name>a</filter-name>
          <url-pattern>/pages/a/*</url-pattern>
      </filter-mapping>
  
      <filter>
          <filter-name>b</filter-name>
          <filter-class>com.z.Filter.Filter_B</filter-class>
      </filter>
      <filter-mapping>
          <filter-name>b</filter-name>
          <url-pattern>/pages/b/*</url-pattern>
      </filter-mapping>
  
      <welcome-file-list>
          <welcome-file>login.jsp</welcome-file>
      </welcome-file-list>
  </web-app>
  ```

## 4.2 前端

- login.jsp

  ```jsp
  <%--
    Created by IntelliJ IDEA.
    User: ANGLE0
    Date: 2019/12/11
    Time: 19:04
    To change this template use File | Settings | File Templates.
  --%>
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>login</title>
  </head>
  <body>
  <form action="login" method="post">
      姓名：<input type="text" name="username">
      密码：<input type="password" name="pass">
      <br>
      <input type="submit" value="登录">
  </form>
  </body>
  </html>
  ```

- list.jsp

  ```jsp
  <%--
    Created by IntelliJ IDEA.
    User: ANGLE0
    Date: 2019/12/11
    Time: 19:05
    To change this template use File | Settings | File Templates.
  --%>
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>list</title>
  </head>
  <body>
      <a href="/pages/a/a1.jsp">a1</a>
      <a href="/pages/a/a2.jsp">a2</a>
      <a href="/pages/b/b1.jsp">b1</a>
      <a href="/pages/b/b1.jsp">b2</a>
  </body>
  </html>
  ```

- 目标页代码（webapp目录下创建pages，并在此文件夹下创建a, b文件夹）

  a中创建：a1.jsp, a2.jsp

  b中创建：b1.jsp, b2.jsp

  示例页面：

  ```jsp
  <%--
    Created by IntelliJ IDEA.
    User: ANGLE0
    Date: 2019/12/11
    Time: 19:08
    To change this template use File | Settings | File Templates.
  --%>
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
      <title>Title</title>
  </head>
  <body>
  a--1
  </body>
  </html>
  ```

  