# 数据库工具类使用

> - 1.自己编写工具
> - 2.使用连接池
>   - 自己编写实现
>   - 使用jar

## 1.JDBCUtil

- 基础操作：

  > - 注册驱动
  > - 获取连接
  > - 编写语句
  > - 执行，并接受返回结果

## 2.DataSource

### 2.1 CRUD

- 代码：

  ```java
  package com.W.jdbc.util;
  
  import com.W.jdbc.domain.Person;
  import com.W.jdbc.hader.IResulltSet;
  
  import java.sql.*;
  import java.util.ArrayList;
  import java.util.List;
  
  public class CRUDTemplate {
  
      public static int excuteUpdate(String sql, Object...params){
          Connection con = null;
          PreparedStatement ps = null;
          try {
              //获取连接
              con = JdbcUtil.getConnection();
              //执行SQL
              ps = con.prepareStatement(sql);
              //设置变量
              for (int i = 0; i < params.length; i++) {
                  ps.setObject(i+1,params[i]);
              }
              //返回受影响的行
              return ps.executeUpdate();
          } catch (SQLException e) {
              e.printStackTrace();
          }finally {
              JdbcUtil.Close(con,ps,null);
          }
          return -1;
      }
  
      public static <T>T excuteQuery(String sql,IResulltSet<T> iResulltSet, Object...params){
          Connection con = null;
          PreparedStatement ps = null;
          ResultSet set = null;
          List<Person> list = new ArrayList<Person>();
          try {
              //连接数据库
              con = JdbcUtil.getConnection();
              //预处理
              ps = con.prepareStatement(sql);
              //遍历设置参数
              for (int i = 0; i < params.length; i++) {
                  ps.setObject(i+1, params[i]);
              }
              //执行SQl语句
              set = ps.executeQuery();
              //遍历结果集，并反回创建的对象
              return iResulltSet.handle(set);
          } catch (Exception e) {
              e.printStackTrace();
          }finally {
              JdbcUtil.Close(con,ps,set);
          }
          return null;
      }
  }
  
  ```

  

- Util工具类

  > - 导包：jar包来源alibaba
  >
  > - 代码：

  ```java
  package com.W.jdbc.util;
  
  import com.alibaba.druid.pool.DruidDataSourceFactory;
  
  import javax.sql.DataSource;
  import java.io.FileInputStream;
  import java.io.FileNotFoundException;
  import java.io.IOException;
  import java.io.InputStream;
  import java.sql.*;
  import java.util.Properties;
  
  
  public class JdbcUtil {
  
      public static DataSource ds = null;
  
      static {
  
          try {
              //加载配置文件
              Properties p = new Properties();
              InputStream in = new FileInputStream("src/Source/db.properties");
              p.load(in);
              ds = DruidDataSourceFactory.createDataSource(p);
          } catch (FileNotFoundException e) {
              e.printStackTrace();
          } catch (IOException e) {
              e.printStackTrace();
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  
      //连接
      public static Connection getConnection(){
          try {
              return ds.getConnection();
          } catch (SQLException e) {
              e.printStackTrace();
          }
          return null;
      }
      //关闭
      public static void Close(Connection connection, Statement statement, ResultSet resultSet){
          if (resultSet != null) {
              try {
                  resultSet.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
          if (statement != null) {
              try {
                  statement.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
          if (connection != null) {
              try {
                  connection.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
      }
  
  }
  ```

### 2.2 C3P0Util

- 导包：

  > c3p0-0.9.1.2.jar
  >
  > standard.jar

- 代码：

  ```java
  package com.itt.w.util;
  
  import com.mchange.v2.c3p0.ComboPooledDataSource;
  
  import java.sql.Connection;
  import java.sql.ResultSet;
  import java.sql.SQLException;
  import java.sql.Statement;
  
  public class C3P0Util {
      private static ComboPooledDataSource dataSource = new ComboPooledDataSource();
      public static Connection getConnection(){
          try{
              return dataSource.getConnection();
          }catch (SQLException e){
              throw new RuntimeException("服务器忙");
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

- 配置文件：

  > 查看官网的配置教程
  >
  > 此处以xml配置为例

  

  - 1.文件名：c3p0-config.xml

  - 2.文件:

  ```xml
  <c3p0-config>
      <default-config>
  
          <property name="DriverClass">com.mysql.jdbc.Driver</property>
          <property name="JdbcUrl">jdbc:mysql://localhost:3306/jwgl</property>
          <property name="User">root</property>
          <property name="Password">123</property>
  
  
          <property name="automaticTestTable">con_test</property>
          <property name="checkoutTimeout">30000</property>
          <property name="idleConnectionTestPeriod">30</property>
          <property name="initialPoolSize">10</property>
          <property name="maxIdleTime">30</property>
          <property name="maxPoolSize">100</property>
          <property name="minPoolSize">10</property>
          <property name="maxStatements">200</property>
  
          <user-overrides user="test-user">
              <property name="maxPoolSize">10</property>
              <property name="minPoolSize">1</property>
              <property name="maxStatem   ents">0</property>
          </user-overrides>
  
      </default-config>
  </c3p0-config>
  ```

  