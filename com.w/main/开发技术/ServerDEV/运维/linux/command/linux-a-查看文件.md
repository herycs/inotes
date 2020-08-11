# Linux查看文件内容的5种方式

**目录**

[1. more指令 —— 分页显示文件内容](https://blog.csdn.net/pro_fan/article/details/84348793#more指令 —— 分页显示文件内容)

[2. less指令 —— 可以向前或向后查看文件内容](https://blog.csdn.net/pro_fan/article/details/84348793#2、less指令)

[3. head指令 —— 查看文件开头的内容](https://blog.csdn.net/pro_fan/article/details/84348793# 3、head指令)

[4. tail指令 —— 显示文件尾部的内容](https://blog.csdn.net/pro_fan/article/details/84348793#4、tail指令)

[5. cat指令 —— 显示文件内容](https://blog.csdn.net/pro_fan/article/details/84348793#5、cat指令)

------

## 1. more指令 —— 分页显示文件内容

***more指令会以一页一页的形式显示文件内容\***，按**空白键（space）**显示下一页内容，按**Enter键**会显示下一行内容，按 **b 键**就会往回（back）一页显示，其基本用法如下：

**more  file1**       查看文件file1的文件内容；

**more  -num  file2**  查看文件file2的内容，一次显示num行；

**more  +num  file3**  查看文件file3的内容，从第num行开始显示；

------

## 2. less指令 —— 可以向前或向后查看文件内容

less指令查看文件内容时可以向前或向后随意查看内容；

less指令的基本用法为：

**less  file1**  查看文件file1的内容；

**less  -m  file2**   查看文件file2的内容，并在屏幕底部显示已显示内容的百分比；

按空格键显示下一屏的内容，按回车键显示下一行的内容； 

**按 U 向前滚动半页，按 Y  向前滚动一行；**

**按[PageDown]向下翻动一页，按[PageUp]向上翻动一页；**

**按  Q  退出less命令；**

------

## 3. head指令 —— 查看文件开头的内容

head指令用于**显示文件开头的内容**，默认情况下，只显示文件的头10行内容；

head指令的基本用法：

**head -n <行数>  filename   显示文件内容的前n行；**

例如：head  -n  5  file1   显示文件file1的前5行内容

**head  -c <字节>  filename   显示文件内容的前n个字节；**

例如：head -c 20 file2   显示文件file2的前20个字节内容

------

## 4. tail指令 —— 显示文件尾部的内容

 tail指令用于**显示文件尾部的内容**，默认情况下只显示指定文件的末尾10行；

tail指令的基本用法：

**tail  file1**   显示文件file1的尾部10行内容；

**tail -n <行数> filename**  显示文件尾部的n行内容；

例如：tail -n 5  file1  显示文件file1的末尾5行内容

**tail -c <字节数>  filename**   显示文件尾部的n个字节内容；

例如：tail -c 20  file2  显示文件file2的末尾20个字节

------

## 5. cat指令 —— 显示文件内容

***使用cat命令时，如果文件内容过多，则只会显示最后一屏的内容；***

cat指令的基本用法：

**cat  file1**    用于查看文件名为file1的文件内容；

**cat  -n  file2**    查看文件名为file2的文件内容，并**从1开始对所有输出的行数（包括空行）进行编号**；

**cat  -b  file3**   查看文件名为file3的文件内容，并从1开始对所有的**非空行进行编号**；