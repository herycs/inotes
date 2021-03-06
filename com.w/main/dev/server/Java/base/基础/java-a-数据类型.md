# 数据类型

# 基本数据类型

### 8种基本数据类型

- char
- boolean
- byte, short, int, long
- float, double

#### 整型中byte、short、int、long的取值范围

-　byte：byte用1个字节来存储，范围为-128(-2^7)到127(2^7-1)，
    -　默认值为0。
-　short：short用2个字节存储，范围为-32,768 (-2^15)到32,767 (2^15-1)
    -　默认值为0，一般情况下，因为Java本身转型的原因，可以直接写为0。
-　int：int用4个字节存储，范围为-2,147,483,648 (-2^31)到2,147,483,647 (2^31-1)
    -　默认值为0。
-　long：long用8个字节存储，范围为-9,223,372,036,854,775,808 (-2^63)到9,223,372,036, 854,775,807 (2^63-1)
    -　long类型的默认值为0L或0l，也可直接写为0。

#### char取值范围

> 16位Unicoode存储

- 最小值是 **\u0000**（即为0）；
- 最大值是 **\uffff**（即为65,535）；

#### 什么是浮点型？

- 1的二进制是01，1.0/2=0.5，那么，0.5的二进制表示应该为(0.1)，以此类推，0.25的二进制表示为0.01

#### 什么是单精度和双精度？

- 单精度浮点数在计算机存储器中占用4个字节（32 bits）
- 双精度浮点数(double)使用 8字节（64 bits） 来存储一个浮点数

#### 为什么不能用浮点型表示金额？

- 计算机中保存的小数其实是十进制的小数的近似值，并不是准确值
