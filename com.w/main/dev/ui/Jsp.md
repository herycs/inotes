# Jsp

## 1.有关设置

1.1 page

设置页面的各种属性

- import

- session
- errorPage
- isErrorPage

- contextType
- pageEncoding
- isELIgnored

1.2 include

- 静态包含和动态包含
  - 能用静态别用动态

2.JSP 的六个动作

- <jsp:include > 动态包含
- <jsp:forward> 请求转发
- <jsp:param> 设置请求参数 
- <jsp:useBean> 创建一个对象
- <jsp:setProperty> 给指定的对象属性赋值
- <jsp:getProperty> 取出指定对象的属性值

3.9个内置对象

- request
- response
- session
- application
- exception
- page
- config
- out
- pageContext

## 2.四个作用域

- PageContext : pageConext 存放的数据在当前页面有效。开发时使用较少。
- ServletRequest: request  存放的数据在一次请求（转发）内有效。使用非常多。
- HttpSession: session 存放的数据在一次会话中有效。使用的比较多。
- ServletContext: application 存放的数据在整个应用范围内都有效，应尽量少用。

## 3.EL表达式

- 本质：jsp中**获取数据**的一种规范
- 限制：只能获取存在4个作用域中的数据，对于null这样的数据，在页面中表现为空字符串
- 有关运算符
  - empty:判断null，空字符串和 没有元素的集合返回TRUE
  - 支持3元运算符
- 隐式对象：
  - pageContext
    - 通过这个获取其他
  - pageScope
  - requestScope
  - sessionScope
  - applicationScope
  - param
  - paramValues
  - header
  - headerValues
  - initParam
  - cookie

## 4.JSTL

4.1认识：JSP标准标签库

4.2作用：使用JSTL实现JSP页面中逻辑处理

4.3使用：

- JSP页面中添加taglib指令
- 使用JSTL标签
  - 通用标签
    - set | out | remove
  - 条件标签
    - if | choose
  - 迭代标签
    - forech

