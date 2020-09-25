# SSM整合报错

# 1.配置文件路径报错：

- 错误明细： Could not open ServletContext resource [/WEB-INF/dispatcherServlet-servlet.xml]

- ```xml
  <servlet>
      <servlet-name>dispatcherServlet</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
      </init-param>
      <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
      <servlet-name>dispatcherServlet</servlet-name>
      <url-pattern>/</url-pattern>
    </servlet-mapping>
  ```

  其中的：contextConfigLocation书写错误，导致Tomcat直接从根目录下找文件加载失败

  - 其它同类别错误：不一定是此处配置文件写错，但tomcat查找配置文件时是从根路径下找的，所以如果没在根下面，则声明其位置

    web.xml中的前端解析器如上图配置（Maven的目录结构，文件在：resources下）

    其它的项目目录结构则需要写上： 

    ```xml
    <context:property-placeholder location="classpath:具体路径" /> 
    ```

# 2.服务器不愿公开...

- 这个问题是在你的服务器中没有找到资源，或者你没给权限

- 解决方法：

  - 针对没有资源：检查你的路径，WEB-INF下的文件是不允许直接访问的，所以如何需要设为<kbd>welcome-file</kbd>建议放到和WEB-INF同级别下

  - 针对没给权限：如何是SSM整合,则需要查看前端控制器处<kbd>url-pattern</kbd>,是写成<kbd>/</kbd>还是<kbd>/*</kbd>,其它项目类似

    ```xml
    <!--配置前端控制器-->
      <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!--加载springmvc.xml配置文件-->
        <init-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <!--启动服务器，创建该servlet-->
        <load-on-startup>1</load-on-startup>
      </servlet>
      <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
      </servlet-mapping>
    ```

  - 其它解决方法：如果是其它类型项目，类似，找到你的界面控制配置处查看路径，存在某些点击后页面跳转却不存在，这时看下你的跳转路径的配置或者路径名是否有写错。

# 3.mysql连接时报错

## 3.1错误：Cannot load connection class because of underlying exception: com.mysql.cj.exceptions.WrongArgumentException:Malformed database URL, failed to parse the main URL sections.

- 解决方法：mysql8版本需要注意，驱动及连接和之前版本的有点区别
  - driver:com.mysql.cj.jdbc.Driver
  - url:jdbc:mysql://localhost:3306///travel??useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=GMT

## 3.2报错：serverTimezone。。。

- 解决方法：url后缀中加上<kbd>？serverTimezone=UTC</kbd>，此时url=jdbc:mysql://localhost:3306///db_name?serverTimezone=UTC

## 3.3报错：username@localhost     user password(yes)

- 解决方法：可能由于你的配置文件书写有误，认证核查一遍properties以及xml引用（如何有的话）
- 其它解决方法：启动mysql，修改用户名及密码，并尝试使用修改后的密码登录（测试其可用性），结束后将配置文件数据修改为最新的可用账户数据

## 3.4报错：有关MySql 8

- 一个报错：页面报错： 500 Request processing failed; nested exception is  org.mybatis.spring.MyBatisSystemException: nested exception is  org.apache.ibatis.exceptions.PersistenceException:  

- 下面贴出我的部分jdbc配置

  ```properties
  driver=com.mysql.cj.jdbc.Driver
  url=jdbc:mysql:///travel?serverTimezone=PRC
  user=root
  password=123456
  ```

- [mysql8报错处理](https://blog.csdn.net/weixin_40845165/article/details/84140606)

