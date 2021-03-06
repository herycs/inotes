# 动态分派

## 静态分派

依赖静态类型定义方法执行版本的分派动作

## 动态分派

>  重写的本质

invokevirtual指令运行时解析过程

1. 操作数栈顶第一个元素指向的对象记作C

2. 若在C中找到与常量描述和简单名称都相符的方法，进行访问权限校验

    通过->返回直接引用，查找结束

    不通过->抛出java.lang.IllegalAccessError

3. 依照继承关系从上往下对父类进行搜索验证

4. 始终未找到，则->java.lang.AbstractMethodError

## 单分派&多分派

单分派：一个宗量对多个目标方法进行选择

多分派：多于一个宗量对多个方法进行选择

基于两个宗量{静态类型+方法参数}进行选择，故java语言的静态分派属于多分派

## 虚拟机动态分派

实际查找过程非常耗时，故使用虚方法表实现对性能影响较小