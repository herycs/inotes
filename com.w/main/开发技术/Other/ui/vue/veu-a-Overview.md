# Overview

## MVVM模式

model view  viewModel

- 文件中temple 就是view
- new Vue 就是ViewModel
- Model就是\<javascript>的数据

## 认识

目录结构

```mysql
index.html # 应用挂载点
main.js # 主js文件，负责引入依赖以及接入其他子组件
route.js # 主路由文件，负责处理逻辑转发
App.vue # 应用主结构
src # 相关资源目录
	components # 组件目录
	images # 图片目录
	lib # 资源文件目录
```

运行流程

用户触发事件-》调用相关方法-》发起请求-》收响应-》修改data展示

## 使用

基本vue文件模板

```vue
<template>
</template> # 页面骨架

<script>
</script> # 数据，操作绑定

<style lang="scss" scoped>
</style> # 样式文件
```

main.js绑定文件

```js
//入口文件
import Vue from 'vue'

//路由
import VueRouter from 'vue-router' //导入路由
Vue.use(VueRouter) //安装路由
import router from './router.js' //导入路由模块

//vue-resource
import VueResource from 'vue-resource' //导入vue-resource
Vue.use(VueResource) //安装

//全局过滤器
import moment from 'moment' //导入格式化插件
Vue.filter('dataFormat', function (dataStr, pattern = "YYYY-MM-DD HH:mm:ss"){ //定义全局时间过滤器
        return moment(dataStr).format(pattern)
})

//导入mint-ui中的组件
import './lib/mui/css/mui.min.css'
import 'mint-ui/lib/style.css'
import './lib/mui/css/icons-extra.css'

//Mint-ui组件
import {Header, Swipe, SwipeItem } from 'mint-ui' //引入组件

Vue.component(Header.name, Header) //注册组件
Vue.component(Swipe.name, Swipe);
Vue.component(SwipeItem.name, SwipeItem);

import app from './App.vue'
var vm = new Vue({
        el:'#app',
        render: c=>c(app),
        router
})
```

route.js文件

```js
import VueRouter from 'vue-router'

//导入路由组件
import HomeContainer from './components/tabbar/HomeContainer.vue'
import MemberContainer from './components/tabbar/MemberContainer.vue'
import ShopcarContainer from './components/tabbar/ShopcarContainer.vue'
import SearchContainer from './components/tabbar/SearchContainer.vue'
//home九宫格路由
import NewsList from './components/news/NewsList.vue'
import NewsInfo from './components/news/NewsInfo.vue'
import PhotoList from './components/photos/photolist.vue'
import PhotoInfo from './components/photos/photoinfo.vue'
import GoodsList from './components/goods/goodslist.vue'
import GoodsInfo from './components/goods/goodsinfo.vue'

var router = new VueRouter({
        routes:[
                {path:'/', redirect:'/home'},
                {path:'/home', component:HomeContainer},
                {path:'/member', component:MemberContainer},
                {path:'/shopcar', component:ShopcarContainer},
                {path:'/search', component:SearchContainer},
                {path:'/home/newslist', component:NewsList},
                {path:'/home/newsinfo/:id', component:NewsInfo},
                {path:'/home/photolist', component:PhotoList},
                {path:'/home/photoinfo/:id', component:PhotoInfo},
                {path:'/home/goodslist', component:GoodsList},
                {path:'/home/goodsinfo/:id', component:GoodsInfo}

        ],
        //覆盖默认高亮类
        linkActiveClass:'mui-active'
})

//向外部提供路由接口
export default router
```

Api文件

```
封装对API的请求，并提供调用方式，使得API抽离出逻辑代码编写中，同时提高了扩展性
```

Route文件

```
1.处理路由跳转
2.处理页面导航展示
3.权限拦截
4.异常页面跳转
```

## 请求处理

1. route.js-》刷新页面布局以及相关联组件连接
2. vue.index-》发起传参，请求调用
3. api-》提供请求接口
4. util-》封装了请求公用部分，例如：全局请求域名