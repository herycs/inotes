# vue——开发项目总结

# 1.开发资源

- 开发工具：VsCode
- nodejs
- 工具：webpack
- 语言：vue

# 2.项目基础目录结构

- 以下是项目一级目录及个文件

- 项目根目录下文件及其结构

# 3.开发结构

- src目录下

  - App.vue:最基础的页面组件结构

    示例文件：

    ```vue
    <template>
      <div class="app-container">
        <!-- 顶部区域 -->
        <mt-header fixed title="顶部"></mt-header>
    
        <!-- 中间路由 -->
        <transition>
          <router-view></router-view>
        </transition>
    
        <!-- 底部区域 -->
        <nav class="mui-bar mui-bar-tab">
          <router-link class="mui-tab-item" to="/home">
            <span class="mui-icon mui-icon-home"></span>
            <span class="mui-tab-label">首页</span>
          </router-link>
          <router-link class="mui-tab-item" to="/member">
            <span class="mui-icon mui-icon-contact"></span>
            <span class="mui-tab-label">会员</span>
          </router-link>
          <router-link class="mui-tab-item" to="/shopcar">
            <span class="mui-icon mui-icon-extra mui-icon-extra-cart">
              <span class="mui-badge">0</span>
            </span>
            <span class="mui-tab-label">购物车</span>
          </router-link>
          <router-link class="mui-tab-item" to="/search">
            <span class="mui-icon mui-icon mui-icon-search"></span>
            <span class="mui-tab-label">搜索</span>
          </router-link>
        </nav>
      </div>
    </template>
    
    
    <script>
    </script>
    
    
    <style lang="scss" scoped>
    .app-container {
      padding-top: 40px;
      padding-bottom: 50px;
      overflow-x: hidden;
    }
    //中间块切换动画
    .v-enter {
      opacity: 0;
      transform: translateX(100%);
    }
    //与进入时相反
    .v-leave-to {
      opacity: 0;
      transform: translateX(-100%);
    }
    .v-enter-active,
    .v-leacve-active {
      transition: all 0.5s ease;
    }
    </style>
    ```

  - index.html基础展示，这里基本只写一个展示框架，详细的文件等信息通过加载对应的组件

    示例：

    ```js
  <html>
    	<head>
            <meta charset="UTF-8">
            <meta 
    			name="viewport content="width=device-width, 
                    initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0, user-scalable=no">
            <title>主页</title>
    	</head>
    	<body style="background-color:white;">
            	<div id="app"/></div>
    	</body>
    </html>
    ```
    
  - main.js：这里写项目主js, 同时配置并加载vue，将第三方组件及注册进veu
  
    将页面的div交给此vue

    示例文件：

    ```js
  //入口文件
    import Vue from 'vue'
  
    //路由
  //导入路由
    import VueRouter from 'vue-router'
    //安装路由
    Vue.use(VueRouter)
    //导入路由模块
    import router from './router.js'
    
    //vue-resource
    //导入vue-resource
    import VueResource from 'vue-resource'
    //安装
    Vue.use(VueResource)
    
    //全局过滤器
    //导入格式化插件
    import moment from 'moment'
    //定义全局时间过滤器
    Vue.filter('dataFormat', function (dataStr, pattern = "YYYY-MM-DD HH:mm:ss"){
            return moment(dataStr).format(pattern)
    })
    
    //导入mint-ui中的组件
    import './lib/mui/css/mui.min.css'
    import 'mint-ui/lib/style.css'
    import './lib/mui/css/icons-extra.css'
    
    //Mint-ui组件
    //引入组件
    import {Header, Swipe, SwipeItem } from 'mint-ui'
    //注册组件
    Vue.component(Header.name, Header)
    Vue.component(Swipe.name, Swipe);
    Vue.component(SwipeItem.name, SwipeItem);
    
    import app from './App.vue'
    
    var vm = new Vue({
            el:'#app',
            render: c=>c(app),
            router
    })
    ```
  
    
  
  - router.js：文件中写路由配置信息
  
    示例：

    ```js
    import VueRouter from 'vue-router'
    
    ```
  
    ```js
    //导入路由组件
    import HomeContainer from './components/tabbar/HomeContainer.vue'
    import SearchContainer from './components/tabbar/SearchContainer.vue'
    ```
  
    ```js
    var router = new VueRouter({
        routes:[
            {path:'/', redirect:'/home'},
            {path:'/home', component:HomeContainer},
            {path:'/search', component:SearchContainer}
        ],
        //覆盖默认高亮类
        linkActiveClass:'mui-active'
    })
    
    //向外部提供路由接口
    export default router
    ```

# 4.其它范例文件

- webpack.config.js

  ```js
  const path = require('path')
  
  const htmlWebpackPlugin = require('html-webpack-plugin')
  
  //向外暴露配置对象，webpack运行的时候，加载指定的配置
  module.exports = {
    entry: path.join(__dirname, './src/main.js'), 
    output: {
      path: path.join(__dirname, './dist'),
      filename: 'bundle.js'
    },
    plugins: [
      new htmlWebpackPlugin({
        template: path.join(__dirname, './src/index.html'),
        filename: 'index.html'
      })
    ],
    module: {
      rules: [
        { test: /\.css$/, use: ['style-loader', 'css-loader'] }, 
        { test: /\.less$/, use: ['style-loader', 'css-loader', 'less-loader'] },
        { test: /\.scss$/, use: ['style-loader', 'css-loader', 'sass-loader'] }, 
        { test: /\.jpg|png|gif|bmp$/, use: 'url-loader?limit=7631&name=[hash:8]-[name].[ext]' }, 
        { test: /\.js$/, use: 'babel-loader', exclude: /node_modules/ }, 
        { test: /\.vue$/, use: 'vue-loader' },
        { test: /\.ttf|woff|woff2|eot|svg$/, use: 'url-loader' },
      ]
    },
    resolve: {
      /* alias: { // 别名
        'vue$': 'vue/dist/vue.js'
      } */
    }
  }
  ```

- package.json（此文件在使用npm init初始化项目后就会有，后续信息会在使用npm下载组件时自动编译进来）

  ```json
  {
    "name": "vue_demo02",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
      "dev": "webpack-dev-server --open --port 3000 --host 192.168.1.244 --hot",
      "pub": "webpack --config webpack.publish.config.js",
      "build": "webpack"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "babel-core": "^6.26.0",
      "babel-loader": "^7.1.2",
      "babel-plugin-transform-remove-strict-mode": "^0.0.2",
      "babel-plugin-transform-runtime": "^6.23.0",
      "babel-preset-env": "^1.6.1",
      "babel-preset-stage-0": "^6.24.1",
      "clean-webpack-plugin": "^0.1.17",
      "css-loader": "^0.28.7",
      "extract-text-webpack-plugin": "^3.0.2",
      "file-loader": "^1.1.6",
      "html-webpack-plugin": "^2.30.1",
      "less": "^2.7.3",
      "less-loader": "^4.0.5",
      "node-sass": "^4.7.2",
      "optimize-css-assets-webpack-plugin": "^3.2.0",
      "sass-loader": "^6.0.6",
      "style-loader": "^0.19.1",
      "url-loader": "^0.6.2",
      "vue-loader": "^13.6.0",
      "vue-template-compiler": "^2.5.13",
      "webpack": "^3.10.0",
      "webpack-dev-server": "^2.9.7"
    },
    "dependencies": {
      "axios": "^0.17.1",
      "css-loader": "^0.28.11",
      "mint-ui": "^2.2.13",
      "moment": "^2.24.0",
      "style-loader": "^0.19.1",
      "stylus": "^0.54.7",
      "stylus-loader": "^3.0.2",
      "vue": "^2.5.13",
      "vue-resource": "^1.5.1",
      "vue-router": "^3.0.1",
      "vue2-preview": "^1.0.1",
      "vuex": "^3.0.1"
    }
  }
  ```


# 5.开发结束

- 打包：使用webpack

- 命令：npm run build

- 配置：

  package.json中script添加

  ​    "build": "webpack"

  命令行运行：npm run build

  dist下的文件一起打包即可发布