

# 小技巧

## 文本溢出省略号

单行：

```css
text-overflow: ellipsis;
white-space: nowrap;
overflow: hidden;
```

多行：

```css
word-break: break-word;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 4;
overflow: hidden;
```

## 点击事件绑定

```vue
模板：
@click="method_1"
脚本：
methods: {
	method_1(){
		dosomething();
	}
}
```

## 路由跳转

### 基础

1. 建立主页和跳转页

2. 配置路由，将跳转页配置到路由工单中（router.js)

3. 建立跳转事件，绑定触发方法

    方法内容：this.$router.push('page2')

### 带参路由跳转

1. 配置router.js中路由路径为 （/page2/:id）
2. 跳转页获取参（created() { this.iid = this.$route.params.id}）

## 父子组件传参

```vue
<span @click="handleClick">
    {{item}}
</span>
```

### 父-》子

```vue
props: {
    title:{
        type: String,
        	default:''
        },
        list: {
            type: Array,
            default: () => []
        }
}
```

### 子-》父

```vue
methods:{
    handleClick(){
        console.log('child')
        this.$emit('aaa')
    }
}
```

## 顶部导航条

使用css3调整其动态的样式适应

```css
.{
    position: relative;
    
    a-left {
        margin-left: 10px;

        # 后面找到子元素为right则，使用下面的操作
        ~ .left {
            position: static; # 设置其定位为静态定位（为了去除浮动定位效果）
        }
    }

    a-center {
        flex: 1;
        margin: 0 10px;
         ~ .left {
            position: static; # 设置其定位为静态定位（为了去除浮动定位效果）
        }
    }

    a-right {
        position: absolute; # 为了排除left center都没有的时候导致显示到左侧去
        right: 0;
        @include flex-center() #自己设计的实现垂直水平居中
        height: 100%;
        mergin-right: 10px;
    }
}
```

## 幻灯片

问题，图片切换，1-2-3-4-5

当从5 ——》 1时，会逐次从5-》4...-》1

需要直接替换为1

## 懒加载

图标 + 文字

需要在样式显示中确保两点

1. 图片与文字设置开关样式：不一定每个地方同时要图片和文字存在
2. 图片与文字位置关系：图片和文字的位置显示需要设置好定位问题，尤其是不同时存在时

## 滑动条

元素显示百分百问题可能导致显示不正常

更新问题，数据异步更新（可用方式：传参更新 | 滚动条暴露方法）

下拉刷新

顶部导航条虹吸效果

## 父子页面跳转以及显示

不同的数据显示不同的信息