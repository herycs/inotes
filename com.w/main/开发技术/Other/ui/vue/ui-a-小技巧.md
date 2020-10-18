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