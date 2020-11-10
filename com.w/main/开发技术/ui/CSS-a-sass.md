# Sass

> 官网： www.sass-lang.com

## 概述

### 文件

.sass文件中 空格缩进为格式表示层级关系

.scss文件中 采用括号表示层级关系

### 编译过程

sass -> css

### 解决痛点

#### 嵌套规则

括号方式解决复杂CSS父子嵌套

#### 变量规则

减少冗余，通过变量将公共CSS样式抽离，减少冗余CSS代码

#### 条件逻辑

像写高级语言一样编写逻辑性的CSS代码

## 基础语法

### 变量

格式

```scss
定义
$width: 300px
    $width: 300px ! default
$url: '/mine/logo.jpg'
$str: abs
引用
div {
	width: $width;
    background-image: url('./img/' + $url); // url 引用
    background-image: url('./img/ + $str'); // url 引用
    background-image: url('./img/#{$url}'); // 差值表达式
}
```

类型

num

String

Array

- 下标从1开始

- 获取数据采用其内置的方法

    例如：nth($list, 2)

作用域

import

符合4种格式的CSS，SASS会在编译时保持原样