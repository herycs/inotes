# 登陆注册小Demo实现

## 1.基础部件

- 数据库
- 服务器
- 程序

## 2.详细配置

### 2.1创建项目并搭建服务器，使它们可以运行

### 2.2 配置数据库

字段要求：除了u_id 和 u_tel外，其他均是 Varchar类型

### 2.3 整体认知

- 项目结构

  以下均为架包

  > dao:数据库操作接口定义
  >
  > ​	daoimpl：接口的实现
  >
  > daomain:类的定义
  >
  > service:调用dao方法的接口定义
  >
  > ​	serviceimpl：接口的实现
  >
  > servlet:和jsp交互
  >
  > exception：定义异常
  >
  > util:数据库工具
  >
  > 

- 数据流

  jsp<-->servlet<--->serviceimpl<-->daoimpl<--->DB

### 2.4 具体代码

- 类代码

  - 和数据库对应的类的代码

    ```java
    package com.itt.w.domain;
    
    public class User {
        private int u_id;
        private String u_name;
        private String u_password;
        private int u_age;
        private String u_eamil;
        private String u_tel;
    
        public void setU_id(int u_id) {
            this.u_id = u_id;
        }
    
        public void setU_name(String u_name) {
            this.u_name = u_name;
        }
    
        public void setU_password(String u_password) {
            this.u_password = u_password;
        }
    
        public void setU_age(int u_age) {
            this.u_age = u_age;
        }
    
        public void setU_eamil(String u_eamil) {
            this.u_eamil = u_eamil;
        }
    
        public void setU_tel(String u_tel) {
            this.u_tel = u_tel;
        }
    
        public int getU_id() {
            return u_id;
        }
    
        public String getU_name() {
            return u_name;
        }
    
        public String getU_password() {
            return u_password;
        }
    
        public int getU_age() {
            return u_age;
        }
    
        public String getU_eamil() {
            return u_eamil;
        }
    
        public String getU_tel() {
            return u_tel;
        }
    
    }
    
    ```

    

  - 用于用户注册时对输入信息检验的类的代码

    ```java
    package com.itt.w.domain;
    
    import java.util.HashMap;
    import java.util.Map;
    
    public class UserConfig {
        private int u_id;
        private String u_name;
        private String u_password;
        private String u_repassword;
        private int u_age;
        private String u_eamil;
        private String u_tel;
    
        //验证错误消息集
        private Map<String,String> msg = new HashMap<String,String>();
    
        public boolean validate(){
    
            //检验用户名合法性
            if ("".equals(u_name)) {
                msg.put("name","用户名不能为空！");
            }else if (!u_name.matches("\\w{2,12}")){
                msg.put("name","用户名必须为2~12位的字母数字组成！");
            }
    
            //检验密码合法性
            if ("".equals(u_password)) {
                msg.put("password","密码不能为空！");
            }else if (!u_password.matches("\\d{6,12}")){
                msg.put("password","密码必须为6~12位的数字组成！");
            }
            //检验二次密码的正确性
            if (!u_repassword.equals(u_password)) {
                msg.put("repassword","两次密码不一致！");
            }
    
            //验证邮箱合法性
            if ("".equals(u_eamil)) {
                msg.put("eamil","密码不能为空！");
            }else if (!u_eamil.matches("\\b^['_a-z0-9-\\+]+(\\.['_a-z0-9-\\+]+)*@[a-z0-9-]+(\\.[a-z0-9-]+)*\\.([a-z]{2}|aero|arpa|asia|biz|com|coop|edu|gov|info|int|jobs|mil|mobi|museum|name|nato|net|org|pro|tel|travel|xxx)$\\b")){
                msg.put("eamil","邮箱格式不正确！");
            }
    
            //验证电话合法性
            if ("".equals(u_tel)) {
                msg.put("tel","电话不能为空!");
            }else if (!u_tel.matches("\\d{11}")){
                msg.put("tel","电话格式不正确");
            }
    //        //验证日期
    //        if("".equals(birthday)){
    //            msg.put("birthday", "生日不能为空！");
    //        }else {
    //            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    //            try {
    //                sdf.parse(birthday);
    //            } catch (ParseException e) {
    //                msg.put("birthday", "生日格式不正确！");
    //            }
    //        }
    //        for(Map.Entry<String, String> m: msg.entrySet()){
    //            System.out.println(m);
    //        }
    
            //错误会放到这个集合中，若集合为空，返回True,表示用户的注册信息合法
            return msg.isEmpty();
        }
    
        public void setU_id(int u_id) {
            this.u_id = u_id;
        }
    
        public void setU_name(String u_name) {
            this.u_name = u_name;
        }
    
        public void setU_password(String u_password) {
            this.u_password = u_password;
        }
    
        public void setU_repassword(String u_repassword) {
            this.u_repassword = u_repassword;
        }
    
        public void setU_age(int u_age) {
            this.u_age = u_age;
        }
    
        public void setU_eamil(String u_eamil) {
            this.u_eamil = u_eamil;
        }
    
        public void setU_tel(String u_tel) {
            this.u_tel = u_tel;
        }
    
        public int getU_id() {
            return u_id;
        }
    
        public String getU_name() {
            return u_name;
        }
    
        public String getU_password() {
            return u_password;
        }
    
        public String getU_repassword() {
            return u_repassword;
        }
    
        public int getU_age() {
            return u_age;
        }
    
        public String getU_eamil() {
            return u_eamil;
        }
    
        public String getU_tel() {
            return u_tel;
        }
    
        public Map<String, String> getMsg() {
            return msg;
        }
    }
    
    ```

- 数据库操作的工具类代码（此处的C3P0只是个名称，内部使用DBUtil实现操作）

  ```java
  package com.itt.w.util;
  
  import com.mchange.v2.c3p0.ComboPooledDataSource;
  
  import java.sql.*;
  
  public class C3P0Util {
  
      private static ComboPooledDataSource dataSource = new ComboPooledDataSource();
      public static Connection getConnection(){
          String driver = "com.mysql.cj.jdbc.Driver";
          String url = "jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf-8";
          String username = "root";
          String password = "123456";
          Connection conn = null;
          
          try {
  
              try {
                  Class.forName(driver); //classLoader,加载对应驱动
                  conn = DriverManager.getConnection(url, username, password);
                  return conn;
              } catch (ClassNotFoundException e) {
                  e.printStackTrace();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          } catch (Exception e) {
              throw new RuntimeException("服务器忙");
          }finally {
              return conn;
          }
      }
      public static void release(Connection con, Statement st, ResultSet set){
          if (set != null) {
              try {
                  set.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
              set = null;
          }
          if (st != null) {
              try {
                  st.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
              st = null;
          }
          if (con != null) {
              try {
                  con.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
              con = null;
          }
      }
  }
  
  ```

- dao

  - dao代码

    ```java
    package com.itt.w.dao;
    
    import com.itt.w.domain.User;
    
    public interface UserDao {
        /*
        * 添加用户
        * */
        public void addUser(User user) throws Exception;
        /*
        * 查找用户
        * */
        public User findUser(User user) throws Exception;
        /*
        * 根据用户名查询用户是否存在
        * */
        public boolean findUserByName(String name);
    }
    
    ```

  - daoimpl代码

    ```java
    package com.itt.w.dao.impl;
    
    import com.itt.w.dao.UserDao;
    import com.itt.w.domain.User;
    import com.itt.w.exception.U_ExitException;
    import com.itt.w.util.C3P0Util;
    
    import java.sql.Connection;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    
    public class UserDaoImpl implements UserDao {
        @Override
        public void addUser(User user) throws Exception {
    
            Connection con = null;
            PreparedStatement ps = null;
    
            try {
                con = C3P0Util.getConnection();
    
                ps = con.prepareStatement("insert into user (u_name, u_password, u_eamil, u_tel) values(?, ?, ?, ?)");
                ps.setString(1, user.getU_name());
                ps.setString(2, user.getU_password());
                ps.setString(3, user.getU_eamil());
                ps.setString(4, user.getU_tel());
    
                ps.executeUpdate();
            } catch (Exception e) {
                e.printStackTrace();
            }finally {
                C3P0Util.release(con, ps, null);
            }
        }
    
        @Override
        public User findUser(User user) throws Exception {
            Connection con = null;
            PreparedStatement ps = null;
            ResultSet set = null;
            User u = null;
            try {
                con = C3P0Util.getConnection();
                ps = con.prepareStatement("select * from user where u_name = ? and u_password = ? ");
                ps.setString(1, user.getU_name());
                ps.setString(2, user.getU_password());
                set = ps.executeQuery();
                if (set.next()){
                    u = new User();
                    u.setU_id(set.getInt(1));
                    u.setU_name(set.getString(2));
                    u.setU_password(set.getString(3));
                    u.setU_age(set.getInt(4));
                    u.setU_eamil(set.getString(5));
                    u.setU_tel(set.getString(6));
                }
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                C3P0Util.release(con, ps, set);
            }
            return u;
        }
    
        @Override
        public boolean findUserByName(String name){
            Connection con = null;
            PreparedStatement ps = null;
            ResultSet set = null;
            try {
                con = C3P0Util.getConnection();
                ps = con.prepareStatement("select * from user where u_name = ?");
                ps.setString(1, name);
                set = ps.executeQuery();
                if (set.next()){
                   return true;
                }
            } catch (Exception e){
                e.printStackTrace();
            }finally {
                C3P0Util.release(con, ps, set);
            }
            return false;
        }
    }
    
    ```

- service

  - service代码

    ```java
    package com.itt.w.service;
    
    import com.itt.w.domain.User;
    import com.itt.w.exception.U_Exception;
    import com.itt.w.exception.U_ExitException;
    
    public interface Userservice {
        /*
        *
        * 注册
        * */
        public void register (User user) throws Exception;
        /*
        *
        * 登陆
        * */
        public User login(User user) throws U_Exception;
        /*
        * 根据用户名查询用户是否存在
        * */
        public boolean findUserByName(String name) throws U_ExitException;
    }
    
    ```

  - serviceimpl代码

    ```java
    package com.itt.w.service.impl;
    
    import com.itt.w.dao.UserDao;
    import com.itt.w.dao.impl.UserDaoImpl;
    import com.itt.w.domain.User;
    import com.itt.w.exception.U_Exception;
    import com.itt.w.exception.U_ExitException;
    import com.itt.w.service.Userservice;
    
    public class UserServiceImpl implements Userservice {
        UserDao userDao = new UserDaoImpl();
        @Override
        public void register(User user) throws Exception {
            userDao.addUser(user);
        }
    
        @Override
        public User login(User user) throws U_Exception {
            User u = null;
            try {
                u = userDao.findUser(user);
                if (u == null) {
                    throw new U_Exception("用户名和密码错误");
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
            return u;
        }
    
        @Override
        public boolean findUserByName(String name) throws U_ExitException{
            boolean b = userDao.findUserByName(name);
            if (b) {
                throw new U_ExitException("用户名已存在");
            }
            return b;
        }
    }
    
    ```

- servlet代码

  - regServlet（注册）

    ```java
    package com.itt.w.web.servlet;
    
    import com.itt.w.domain.User;
    import com.itt.w.domain.UserConfig;
    import com.itt.w.exception.U_ExitException;
    import com.itt.w.service.Userservice;
    import com.itt.w.service.impl.UserServiceImpl;
    import org.apache.commons.beanutils.BeanUtils;
    
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    
    @WebServlet("/reg")
    public class regServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            req.setCharacterEncoding("UTF-8");
            resp.setContentType("text/html;charset=UTF-8");
    
            UserConfig uf = new UserConfig();
            try {
                BeanUtils.populate(uf, req.getParameterMap());
            } catch (Exception e) {
                e.printStackTrace();
            }
            if (!uf.validate()) {
                //非空证明有错误
                req.setAttribute("uf",uf);
                req.getRequestDispatcher("reg.jsp").forward(req,resp);
                return;
            }
    
            User u = new User();
            try {
                BeanUtils.populate(u, req.getParameterMap());
                Userservice userservice = new UserServiceImpl();
                //查找用户名是否被注册，已注册会抛出异常
                System.out.println(u);
                boolean s = userservice.findUserByName(u.getU_name());
                userservice.register(u);
            }catch (U_ExitException e) {
                e.printStackTrace();
                req.setAttribute("user_Exit","用户名已存在");
                req.getRequestDispatcher("reg.jsp").forward(req, resp);
            } catch (Exception e) {
                e.printStackTrace();
            }
            resp.getWriter().write("注册成功，2秒后跳转至登陆界面！");
            resp.setHeader("refresh","2;login.jsp");
        }
    
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.doPost(req, resp);
        }
    }
    
    ```

  - loginServlet代码（登陆）

    ```java
    package com.itt.w.web.servlet;
    
    import com.itt.w.domain.User;
    import com.itt.w.exception.U_Exception;
    import com.itt.w.service.Userservice;
    import com.itt.w.service.impl.UserServiceImpl;
    import org.apache.commons.beanutils.BeanUtils;
    
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    
    @WebServlet("/login_test")
    public class loginServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException{
            req.setCharacterEncoding("UTF-8");
    
            User user = new User();
    
            try {
                BeanUtils.populate(user, req.getParameterMap());
            } catch (Exception e) {
                e.printStackTrace();
            }
            Userservice userservice = new UserServiceImpl();
            User u;
            try {
                u = userservice.login(user);
    
                if (u != null) {
                    //登陆成功，user对象放入Session中
                    req.getSession().setAttribute("u", user);
                    req.getRequestDispatcher("index.jsp").forward(req, resp);
                }
            } catch (U_Exception e) {
                e.printStackTrace();
                //错误，跳转回登陆界面
                req.setAttribute("msg", e.getMessage());
                resp.sendRedirect(req.getContextPath()+"/login.jsp");
    //            req.getRequestDispatcher("login.jsp").forward(req, resp);
            }
    
        }
    
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            doGet(req, resp);
        }
    }
    
    ```

  - ​	logoutServlet（注销代码）

    ```java
    package com.itt.w.web.servlet;
    
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    
    @WebServlet("/logout")
    public class logoutSerlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            //销毁Session
            req.getSession().invalidate();
            resp.sendRedirect("index.jsp");
        }
    
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.doPost(req, resp);
        }
    }
    
    ```

- 前端代码

  - login.jsp

    ```jsp
    <%@ page import="java.util.HashMap" %>
    <%@ page import="java.util.Map" %>
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <%
    
        String name = "";
        Cookie[] cookies = request.getCookies();
        for (int i = 0; cookies != null && i < cookies.length; i++) {
            if (name.equals(cookies[i].getName())) {
                name = cookies[i].getValue();
            }
        }
    %>
    
    
    <html>
    <head>
        <title>Title</title>
    </head>
    <body>
    
    <div id="form" class="form_table">
        <%--action="ss"--%>
        <form action="login_test" method="get">
            <%
                String msg = (String) request.getSession().getAttribute("msg");
                if (msg != null) {
                    out.write(msg);
                }
            %>
            <div id="form_username">
                <span>用户名：</span><input type="text" name="u_name" class="input_text" value=<%=name%> >
            </div>
            <br>
            <div id="form_password">
                <span>密&nbsp;&nbsp;&nbsp;&nbsp;码: </span><input type="password" name="u_password" class="input_text" >
            </div>
            <br>
            <br>
            <input type="submit" value="登陆" onclick="login()">
        </form>
    </div>
    </body>
    </html>
    
    ```

    

  - reg.jsp

    ```jsp
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>register</title>
    </head>
    <body>
    <form action="reg" method="get">
    
        <%
            String msg = (String) request.getSession().getAttribute("user_Exit");
        if (msg != null) {
            out.write(msg);
        }
        %>
    
        <div id="form_username">
            <span>用户名：</span><input type="text" name="u_name" class="input_text" value=${uf.u_name}>${uf.msg.name}${requestScope.get("user_Exit")}
        </div>
        <br>
        <div id="reg_password">
            <span>密&nbsp;&nbsp;码: </span><input type="password" name="u_password" class="input_text" >${uf.msg.password}
        </div>
        <br>
        <div id="reg_repassword">
            <span>再次输入密码: </span><input type="password" name="u_repassword" class="input_text" >${uf.msg.repassword}
        </div>
        <br>
        <div id="reg_age">
            <span>年&nbsp;&nbsp;&nbsp;&nbsp;龄: </span><input type="text" name="u_age" class="input_text" value="${uf.u_age}" >
        </div>
        <br>
        <div id="reg_eamil">
            <span>eamil: </span><input type="text" name="u_eamil" class="input_text" value="${uf.u_eamil}">${uf.msg.eamil}
        </div>
        <br>
        <div id="reg_tel">
            <span>电&nbsp;&nbsp;&nbsp;&nbsp;话: </span><input type="text" name="u_tel" class="input_text" value="${uf.u_tel}">${uf.msg.tel}
        </div>
        <br>
        <br>
        <input type="submit" value="注册">
    </form>
    </body>
    </html>
    
    ```

    

  - Start.html

    ```jsp
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta http-equiv="Content-Type" content="text/html" charset="UTF-8">
        <title>Start</title>
        <link rel="stylesheet" type="text/css" href="css/myStyle.css">
        <!--<script type="text/javascript" src="js/myjs.js"></script>-->
    </head>
    <body>
        <div style="align:center;">
            <a href="/login.jsp">登陆</a>
            <a href="/reg.jsp">注册</a>
        </div>
    </body>
    </html>
    ```

  - index.jsp(登陆成功界面)

    ```java
    <%@ page import="javax.naming.InitialContext" %>
    <%@ page import="javax.naming.Context" %>
    <%@ page import="javax.sql.DataSource" %>
    <%@ page import="java.sql.Connection" %>
    
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
    <html>
      <head>
        <title>My Web Test</title>
      </head>
      <body>
            <c:if test="${empty u}">
                <a href="login.jsp">登陆</a>
            </c:if>
            <c:if test="${not empty u}">
                欢迎您：${u.u_name}<a href="logout">注销</a>
            </c:if>
      </body>
    </html>
    
    ```

    