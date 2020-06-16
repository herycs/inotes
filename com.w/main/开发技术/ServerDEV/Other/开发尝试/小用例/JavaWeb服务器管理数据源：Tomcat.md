## JavaWeb服务器管理数据源：Tomcat

#### 1配置数据源（DBCP）

- 步骤：

  - 1.拷贝数据库的连接的jar到tomcat\lib目录下

  - 配置数据源的XML文件

    - 使得本数据源所有应用可用，将配置信息写入tomcat的conf目录下的context.xml中

      示例：（下列代码中的userName，myPassword，driver自己配置，另外name=<kbd>name="jdbc/TestDB"</kbd>为标识名用于给服务器或应用提供检索标志，获取到正确的连接）

      ```java
      <Context>
          <Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
                maxTotal="100" maxIdle="30" maxWaitMillis="10000"
                username="myName" password="myPassword" driverClassName="com.mysql.jdbc.Driver"
                url="jdbc:mysql://localhost:3306/javatest"/>
      </Context>
      ```

      

    - 使得本数据源仅对当前应用可用，在当前应用的MATE-INF中创建并写入配置信息文件context.xml

      - 文件内容参考tomcat的运行主页，（默认的本地URL为localhost:8080）公布的配置文件示例

  - index.jsp测试页面

    - ```Java
      <%@ page contentType="text/html;charset=UTF-8" language="java" %>
      <html>
        <head>
          <title>My Web Test</title>
        </head>
        <body>
              <%
                  Context initContext = new InitialContext();
                  DataSource ds = (DataSource)initContext.lookup("java:/comp/env/jdbc/TestDB");
                  Connection conn = ds.getConnection();
                  out.print(conn);
              %>
        </body>
      </html>
      ```

      