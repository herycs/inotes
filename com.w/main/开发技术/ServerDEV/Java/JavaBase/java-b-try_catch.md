# Try_Catch

## try中有return是否还会执行，finally呢？

JVM规范中说明如下：

如果try语句里有return，那么代码的行为如下：
1.如果有返回值，就把返回值保存到局部变量中
2.执行jsr指令跳到finally语句里执行
3.执行完finally语句后，返回之前保存在局部变量表里的值

