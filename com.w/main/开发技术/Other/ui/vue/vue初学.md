# vue初学

# 1.基础

> 基础结构：
>
> ```html
> <!DOCTYPE html>
> <html>
> <head>
>      <meta charset="UTF-8">
>      <title>test1</title>
>      <script src="../js/vue.js"></script>
> </head>
> <body>
>      <div id="run">
>  	<input type="button" @click="run" :value="msg">
>  </div>
>      <script>
>               var vue = new Vue({
>                 	el:'#run',
>                 	data:{},
>                         methods:{},
>                         filters:{},
>                         directives:{},
>                         component:{},
>                         
>                         beforeCreate(){},
>                         created(){},
>                         beforeMount(){},
>                         monuted(){},
>                         beforeUpdate(){},
>                         updated(){},
>                         beforeDestory(){},
>                         destoryed(){}
>                 })
>         </script>
>      </script>
> </body>
> </html>
> ```
>
> 



- v-cloak：设置属性后可解决闪烁问题，且命名可自定义，注意若是下划线则会出问题

  如：v_cloak

  测试代码：

- v-text：默认没有闪烁，但解析后会将属性值填写到标签中

  例如：<p v-text="message">123</p>显示时123会被覆盖

- 使用插入值表达式：只替换表达式部分格式为<kbd>{{data中的属性名}}</kbd>

- 绑定事件：v-on 简写 @

- 绑定属性：v-bind 简写<kbd> ：</kbd>

# 2. v-on修饰符

- .stop 阻止事件冒泡
- .prevent 阻止默认事件
- .capture 添加事件侦听器时使用捕获机制
- .self 只有事件在该元素本身触发事触发回调
- .once 只触发一次

# 3. v-model双向绑定数据

- model绑定元素仅限于表单元素

# 4.vue样式

## 4.1使用方式

- 标签中直接使用<kbd>:style="写样式"</kbd>
- 标签中引用<kbd>style="样式写data中，此处调用即可"</kbd>
- 标签中使用数组<kbd>style="[css1, css2, ...]"</kbd>引用多个对象

# 5.v-for

## 5.1迭代数据

- 迭代数组
- 迭代数据
- 迭代对象

## 5.2v-for问题

- 迭代数据时，会遍历对象数组，包括其ID，但遍历结束时会失去ID，dom中若对数据项进行操作很可能会出现问题

  解决方法：遍历时指定唯一可识别的KEY

# 6. v-if 和 v-show

- 特点：

  v-if每次回重新删除创建元素

  v-show改变其显示方式

- 消耗：

  v-if较高切换性能的消耗

  v-show较高的原始渲染的性能消耗

- 应用：设计频繁切换，建议v-if； 元素对用户始终不可见，建议v-show

# 7.列表遍历动画渲染

- 列表渲染：

  ```html
  <!-- 添加appear会使得列表加载时产生入场效果 -->
  <transition-group appear><kbd>此处添加要遍历的数据</kbd></transition-group>
  
  <!-- 添加tag,渲染时会渲染为目标属性，默认是span -->
  <transition-group appear tah="ul"><kbd>此处添加要遍历的数据</kbd></transition-group>
  ```

- 列表渲染样式

  ```css
  /*增加添加动画效果*/
  .v-enter,
  .v-leave-to{
      opacity: 0;
      transform: translateY(80px);
  }
  .v-enter-active,
  .v-leave-active{
      transition: all 0.6s ease;
  }
  ```

- 优化数据项的移动效果

  ```css
  /*元素位移时的渐变,与下面的 v-leave-actice配合使用*/
  .v-move{
  	transition: all 0.6s ease;
  }
  .v-leave-active{
  	position: absolute;
  }
  ```

# 8.动画显示

## 8.1自定义动画展示样式

> 基本属性：
>
>  v-enter：动画开始前
>  v-leave-to：动画结束后
>
>  v-enter-active：入场动画
>  v-leave-active：离场动画



- 自定义显示样式

  ```css
  /*移入*/
  .v-enter,
  .v-leave-to{
      opacity: 0;
      transform: translateX(150px);
  }
  /*v-enter-active：入场动画
  v-leave-active：离场动画*/
  /*淡出*/
  .v-enter-active,
  .v-leave-active{
      transition:all 0.4s ease;
  }
  ```

- 自定义绑定前缀（就是代码样式中的v）

  ```html
  /*自定义前缀实现不同元素不同动画加载样式的绑定*/
  <style>
  	.my-enter,
      .my-leave-to{
      opacity: 0;
          transform: translateY(150px);
      }
  
      .my-enter-active,
      .my-leave-active{
          transition:all 0.4s ease;
      }
  </style>
  /*表单中锁定前缀*/
  <transition name="my">
  	<h1 v-if="flag1">123</h1>
  </transition>
  ```

## 8.2调用第三方库

- animate.css

- 使用：

  ```html
  <transition enter-active-class="animated bounceInDown" leave-active-class="animated bounceOutRight" :duration="800">
  	<h1 v-if="flag">123</h1>
  </transition>
  ```

## 8.3钩子函数使用

- 页面使用

  ```html
  <transition
      @before-enter="beforeEnter"
      @enter="enter"
      @after-enter="afterEnter">
      <div v-show="flag" class="ball"></div>
  </transition>
  //对应函数写自己的动画效果
  ```

# 9.组件

> 抽离代码，提高复用，与模块化不同的是组件化不倾向于代码逻辑而是页面部件
>
> 》》》template中有且仅有唯一的一个根元素
>
> - 示例：template:'<h1>第一个根元素</h1><div>第二个根元素</div>'是不合法的

## 9.1组件的创建

- 方式1：vue.extend

  ```js
  //定义组件：
  var com = Vue.extend({
  	template:'<h1>这是创建的组</h1>'
  })
  //绑定组件
  Vue.component('my-com', com)
  
  //简写以上方式
   Vue.component("my-com",Vue.extend({
       template:'<h1>这是创建的组</h1>'
   }))
  ```

- 使用vue.component

  ```js
  Vue.component('com2',{
  	template:'<h2>组件创建方式2</h2>'
  })
  ```

- 外置模板引入

  ```js
  //模板位置，vue绑定的元素块外部
  <template id="tem">
      <div>
           <h1>外部模板</h1>
      </div>
  </template>
  
  //模板绑定
  Vue.component('com3',{
  	template:'#tem'
  })
  ```

## 9.2组件使用

```xml
html代码的vue绑定区中直接使用你的组件名
例如：上面组件中的，my-com, com1, com3作为标签名即可
```

```xml
组件中可以有data,但data必须为function且返回对象
>>>好处,避免同名组件间元素相互影响
```

## 9.3组件间的切换

- v-if和v-else实现不同组件的显示与否（两个组件）：简单应用，登录注册点击后切换公共区域

- 使用component占位（多个组件）

  ```html
  <component :is="comName"></component>
  ```

  切换方式

  ```html
  <a href="" @click.prevent="comName='login'">登录</a>
  <a href="" @click.prevent="comName='register'">注册</a>
  //点击更改组件ID值为对应的组件名
  <script>
  	Vue.component('login',{
           template:'<h1>填写登录信息</h1>'
      })
      Vue.component('register',{
          template:'<h1>填写注册信息</h1>'
      })
  </script>
  ```

  - ```html
    //使用transition包裹component即可
    <transition>
    	<component :is="comName"></component>
    </transition>
    ```

    ```html
    //切换顺序
    <transition mode="out-in"></transition>
    ```

## 9.5父子组件传值

### 9.5.1父组件---》子组件

> 子组件默认无法访问父组件的数据

- 引用时传值：v-bind属性绑定给子组件，需要注意的是，父组件传递值时使用的变量需在子组件中props数组中定义

  ```html
  //vue绑定域中传递值给子组件,同时变量名为parentmsg，值是msg
  <div id="app">
  	<com v-bind:parentmsg="msg"></com>
  </div>
  ```

  ```js
  //子组件中获取值,定义与父组件传值相同的变量名
  //注意：props数组中定义的值一般不建议修改，可将变动数据写到data中
  components:{
      com:{
          template:'<h1>子组件的模型，父组件传递的数据解析结果是：{{parentmsg}}</h1>',
          props:['parentmsg']
  	}
  }
  ```

## 9.6父子组件传递方法

- 使用事件绑定机制， v-on，向子组件传递一个父组件的方法引用

- 子组件中使用$emit触发父组件的方法执行

  ```html
  //vue绑定的区域中，传递引用，方法引用名为：fun, 被引用方法是：show
  <com1 v-on:fun="show"></com1>
  ```

  ```js
  //vue操作区
  var vue = new Vue({
      el: '#app',
      data: { },
      methods: {
          show(){
              alert('父组件方法执行了')
          }
      },
      components:{
          com1:{
              template:'<h1>子组件的模型1，等待父组件方法<input type="button" @click="click_func" value="点击触发方法"/></h1>',
              methods:{
                  click_func(){
                      //调用click_func时会通过以下语句触发引用函数
                      //this代表当前组件，$emit触发操作，()中写触发的方法引用名
                      this.$emit('fun')
                  }
              }
          }
      }
  })
  ```

  传递方法同时传参

  ```js
  //上面方法修改部分如下
  //父组件方法
  show(data){
      alert('父组件方法执行了'+data)
  }
  //子组件方法修改
  click_func(){
      //调用click_func时会通过以下语句触发引用函数
      //this代表当前组件，$emit触发操作，()中写触发的方法引用名
      this.$emit('fun',999)
  }
  ```

# 10获取组件元素

- 标签设置属性ref="XXX"，便可在vue中使用<kbd>this.$refs.XXX.innerText</kbd>获取其值
- 组件设置ref="XXX"，便可在vue中通过这个组件的引用操作组件data，调用组件方法

# 11.路由

- 前端路由而言，单页面应用程序主要通过URL中的Hash实现不同页面的跳转，而HTTP请求中并不会包含Hash相关内容，这种通过Hash改变页面跳转的方式叫前端路由

## 11.1 v-router

- 使用基础：配置对应的router.js包

- 使用方式：

  1.创建routerObj对象

  ```js
  // 登录组件模板对象
  var login = ({
      template:'<h1>登录</h1>'
  });
  //注册模块组件
  var register = ({
      template:'<h1>注册</h1>'
  });
  
  var routerObj = new VueRouter({
      //路由匹配规则
      //下面的route对象数组中，每个对象必有path和component属性
      //path:监听路径，component:路径匹配时显示的组件（代码中写组件模板对象，非组件引用）
      route:[
          {path:'/login',component:login},
          {path:'/register', component:register}
      ]
  });
  ```

  2.将路由组件与Vue实例关联

  ```js
  var vue = new Vue({
      el:'#app',
      data:{},
      methods:{
  
      },
      //注册vue组件
      router:routerObj
  })
  ```

  3.实现组件切换

  ```html
  //在vue接管区域
  //传统跳转连接
  <a href="#/login">登录</a>
  <a href="#/register">注册</a>
  //新跳转方式,router-link默认渲染a标签
  <router-link to="/login">登录</router-link>
  <router-link to="/register">注册</router-link>
  <router-view></router-view>
  ```

  4.设置页面默认组件

  ```js
  {path:'/', redirect:'/login'},
  ```

## 11.2 高亮

- 法1：

  ```css
  //给router-link-active添加样式
  <style>
      .router-link-active{
      }
  </style>
  ```

- 法2：

  ```js
  //router构造方法中设置属性值
  linkActiveClass:'mycss'
  ```

## 11.3 查询规则

- 路由中传递参数作为查询规则，则不需要修改路由规则

  ```js
  //下面的都匹配/login
  <router-link to="/login?id=13">登录</router-link>
  <router-link to="/login?id=10&name=lll">登录</router-link>
  ```

  ```js
  //组件读出路径参数
  //在解析路径时会将参数放置到route的query对象中，故此，我们调用其值的方法是:this.$route.query.id
  template:'<h1>登录组件,组件中传递的参数是：{{$route.query.id}}'
  ```

- 路由传参，路由模式匹配

  ```js
  //模板对象
  template: '<h1>登录组件2,组件中传递的参数是：{{$route.params.id}},组件本身的msg={{msg}}</h1>',
  ```

  ```html
  <div id="app2">
      <router-link to="/login2/14">登录</router-link>
      <router-link to="/register2">注册</router-link>
      <transition mode="out-in">
          <router-view></router-view>
      </transition>
  </div>
  ```

  ```js
  routes: [
      { path: '/login2/:id', component: login2 },
      { path: '/register2', component: register2 }
  ],
  //此处需要注意：我们的路由若是：/login/:id/:name,而实际路径是：/login/12则会匹配不到
  ```

## 11.4 路由嵌套

- 方法：路由嵌套使用

  ```js
  routes: [
      { 
          path: '/account',
          component: account,
          children:[
              {path:'login', component: login},
              {path:'register', component: register}
          ]
      }
  ],
  ```

- 子组件嵌套

  ```html
  //页面接管区域
  <div id="app">
      <router-link to="/account">account</router-link>
      <router-view></router-view>
  </div>
  //组件模板
  <template id="temp">
      <div>
          <h1>account组件</h1>
        	//路径从/开始  
          <router-link to="/account/login">登录组件</router-link>
          <router-link to="/account/register">注册组件</router-link>
          <router-view></router-view>
      </div>
  </template>
  <template id="login">
      <div>
          <h1>登录组件</h1>
      </div>
  </template>
  <template id="register">
      <div>
          <h1>注册组件</h1>
      </div>
  </template>
  ```

  ```js
  //组件模板绑定到组件模板对象
  //组件模板对象
  var account = {
      template: '#temp'
  }
  var login = ({
      template: '#login'
  })
  var register = ({
      template: '#register'
  })
  //路由使用
  var router = new VueRouter({
      routes: [
          {
              path: '/account',
              component: account,
              children: [
                  //路径前不加 /
                  { path: 'login', component: login },
                  { path: 'register', component: register }
              ]
          }
      ],
      linkActiveClass: 'myactive'
  })
  //vue绑定路由
  var vue = new Vue({
      el: '#app',
      data: {},
      methods: {},
      router: router
  })
  ```

# 12 布局

- 多个组件同时布局到一个页面分割整个区域

- 引用：

  ```html
  <div id="app">
      <router-view></router-view>
      <div class="container">
          <router-view name="left"></router-view>
          <router-view name="main"></router-view>
      </div>
  </div>
  ```

  

- 路由:

  ```js
  var router = new VueRouter({
      routes: [
          {
              path: '/',
              components: {
                  'default': header,
                  'left': left,
                  'main': body
  
              }
          },
      ]
  })
  ```

# 13 watch

## 13.1监听数据域的变化

>  可在其中编写数据变动时触发的方法

- 代码示例：

  界面数据区域

  ```html
  <div id="app">
      姓：
      <input type="text" v-model="firstName">
      名：
      <input type="text" v-model="lastName">
      <input type="text" v-model="fullName">
  </div>
  
  ```

  watch监听区

  ```js
  var vue = new Vue({
      el: '#app',
      data: {
          firstName:'',
          lastName:'',
          fullName:'',
      },
      methods: {
      },
      watch:{
          'firstName':function(newVal, oldVal){
              this.fullName = newVal + '-' + this.lastName
          },
          'lastName':function(newVal){
              this.fullName = this.firstName + '-' + newVal
          }
      }
  })
  ```

## 13.2 监听路由变化

- 代码示例：

  ```js
  //veu中
  watch:{
      '$route.path':function(newVal, oldVal){
          if(newVal === '/login'){
              console.log('欢迎来到登录界面！');
          }else if(newVal === '/register'){
              console.log('欢迎来到注册界面，请注册')
          }else{
              console.log('你的请求去找诗和远方了')
          }
      }
  }
  ```

# 14 使用计算属性

> 计算属性本质是一个方法，但将方法名当属性用，使用属性时并不会作为调用方法

- 代码示例：

  ```js
  computed:{
      //计算属性本质是一个方法，但将方法名当属性用，使用属性时并不会作为调用方法
      'fullName':function(){
          return this.firstName + '-' + this.lastName
      }
  }
  ```

- 注意：

  - 方法属性引用时当普通属性即可
  - 计算属性的方法内部任何数据变化都会导致重新计算
  - 属性求值方法存在缓存中，数据不变化，计算不重置

# 15 render

> 可将绑定元素替换为设置的模板

- reder中写函数，返回模板对象,这个方法会将vue控制区域的所有东西替换掉

  ```js
  render:function(createElements){
      // createElements方法会将指定的模板对象渲染为html
      var html = createElements(login)
      return html
  }
  ```