# Security配置

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:security="http://www.springframework.org/schema/security"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/security
       http://www.springframework.org/schema/security/spring-security.xsd">

    <security:global-method-security jsr250-annotations="enabled">

    </security:global-method-security>

    <!-- 配置不过滤的资源（静态资源及登录相关） -->
    <security:http security="none" pattern="/reg1.jsp" />
    <security:http security="none" pattern="/login1.jsp" />

    <security:http security="none" pattern="/reg3.jsp" />
    <security:http security="none" pattern="/login3.jsp" />

    <security:http security="none" pattern="/error.jsp" />
    <security:http security="none" pattern="/index.jsp" />
    <security:http security="none" pattern="/login.jsp" />

    <security:http security="none" pattern="/layui/*" />
    <security:http security="none" pattern="/js/*" />
    <security:http security="none" pattern="/css/*" />
    <security:http security="none" pattern="/themes/*" />

    <security:http auto-config="true" use-expressions="false">
        <!-- 配置拦截路径及通行权限 -->
        <security:intercept-url pattern="/**" access="ROLE_USER" />
        <!--
            login-page 自定义登陆页面
            authentication-failure-url 用户名密码错误
            default-target-url 登陆成功跳转页面
            注：登陆页面用户名固定 username，密码 password，action:login -->
        <security:form-login login-page="/login.jsp" login-processing-url="/login"
                             username-parameter="username"
                             password-parameter="password"
                             authentication-failure-url="/error.jsp"
                             default-target-url="/index.jsp"
                             authentication-success-forward-url="/index.jsp"/>
        <!-- 登出，
            invalidate-session 是否删除session
            logout-url：登出处理链接
            logout-success- url：登出成功页面 -->
        <security:logout invalidate-session="true" logout-url="/logout" logout-success-url="/login.jsp" />
        <!-- 关闭 C S R F,跨服务器请求，默认是开启的,关闭 -->
        <security:csrf disabled="true" />
    </security:http>

    <security:authentication-manager>
        <!--下面的userService中就实现了对应的方法，-->
        <security:authentication-provider user-service-ref="userService">
            <security:password-encoder ref="passwordEncoder"/>
        </security:authentication-provider>
    </security:authentication-manager>

<!--    配置加密类-->
    <bean id="passwordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"></bean>

</beans>
```

