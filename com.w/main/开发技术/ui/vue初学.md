# webpack，npm

> npm, nodejs, webpack

# 1.初识

## 1.1 npm

- 打包管理工具
- 可对前端项目进行管理，文件进行相应处理

## 1.2 webpack

- 项目文件处理工具
- 可对项目文件的js引用进行处理，代码进行压缩，混淆等等，以提高项目的运行性能。
  - 处理js依赖关系
  - 处理js兼容问题，高级别语法-->低级别语法

# 2.基础安装

## 2.1 [nodejs下载](http://nodejs.cn/)，[安装教程](https://www.cnblogs.com/goldlong/p/8027997.html)

> 安装完nodejs，配置好node_global和node_cache,以及命令后基本可以到此为止了，若有其他需要安装的，后面再操作

## 2.2 安装完nodejs后,[创建项目教程](https://www.jianshu.com/p/dc83181ff598)

> 注意创建项目前建议创建一个包，或者cd到你得项目文件处，而后使用上面的教程，对于模块的其他操作不再缀述，可[参考这里](https://www.jianshu.com/nb/19700126)。

# 3.使用webpack

> 安装配置或者操作完上面的操作后基本就可以使用webpack了，建议我们使用webpack前，使用npm命令将webpack及webpack-cli安装到全局仓库
>
> ```js
> //命令模板
> npm install 文件名 -g
> //示例
> npm install webpack-cli -g
> ```

## 3.1主动控制打包命令

- 打包文件：

  - 方式1

    ```js
    //命令行打包指定文件
    webpack 打包文件 目标文件
    ```

    

  - 方式2

    webpack.config.js编写如下

    ```js
    const path = require('path')
    
    module.exports = {
            //打包文件，下面的__dirname是双下划线
            entry:path.join(__dirname, './src/main.js'),
            //目标文件
            output:{
                    path:path.join(__dirname, './dist'),
                    filename:'bundle.js'
            }
    }
    ```

    修改js文件后

    ```
    命令行键入webpack即可
    ```

    页面引用

    ```html
    //我们引用JS时
    <script src="./dist/bundle.js"></script>
    ```

  - 可能的警告：

    - 警告：WARNING in configuration The 'mode' option has  not been set. Set 'mode' option to 'development' or 'production' to  enable defaults for this environment.

    - 处理：

      ```
      配置文件中指定为发行或者研发：
      mode: 'development' |mode: 'production'
      命令行
      webpack --mode development|webpack --mode production  
      ```

## 3.2 使用 workpack-dev-server

### 3.2.1  package.json配置带参数

- 安装：

  ```xml
  //》》安装到本地仓库《《
  npm install workpack-dev-server
  ```

- 使用

  package.json中配置

  ```js
  "scripts": {
  "test": "npm vue01",
  "dev":  "webpack-dev-server"
  },
  //若上述文件中"dev": "webpack-dev-server --open --port 3000 --contentBase src --hot"，
  	//其中添加open 则是表明打包完后自动打开浏览器
      //port：运行端口3000(3000-7200)
      //contentBase：打开根路径为src
      //--hot：热重载，不刷新重载
      //以上webpack-dev-server为基础，后续命令为可有可无的参数
  ```

  配置文件见3.1中的webpack.config.js

  页面引用

  ```xml
  <script src="/bundle.js"></script>
  ```

- 认知

  我们的webpack编译文件时实际是将其托管到内存中的，项目根目录中是找不到的故在webpack托管的时候使用的是/fileName.js，例如说上面的页面引用就是不再是src="./dist/bundle.js"

### 3.2.2 webpack.config.js中配置参数

- package.json

  ```js
  "scripts": {
      "test": "npm vue01",
      "dev":  "webpack-dev-server"
    },
  ```

- webpack.config.js

  ```js
  //指定为开发本
  mode: 'development' 
  //指定为发行版本
  // mode: 'production'
  
  const path = require('path')
  
  //热部署步骤2
  const webpack = require('webpack')
  
  module.exports = {
          //打包文件
          entry:path.join(__dirname, './src/main.js'),
          //目标文件
          output:{
                  path:path.join(__dirname, './dist'),
                  filename:'bundle.js'
          },
          devServer:{
          //配置项,针对package.json中的dev-server
          open:true,
          port:3000,
          contentBase:'src',
          //热部署步骤1
          hot:true
          },
          plugins:[
          //热部署步骤3
                  new webpack.HotModuleReplacementPlugin()
          ]
  }
  ```

## 3.3 插件

> 自动将内存中的js引入到页面中

### 3.3.1 html-webpack-plugin

webpack.config.js

```js
//引用html-webpack-plugin，插件
const htmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
    //...
    plugins:[
        new htmlWebpackPlugin({
            template:path.join(__dirname, './src/index.html'),
            filename:'index.html'
        })
        }
```

### 3.3 加载器

#### 3.3.1样式加载器

- css, scss, less

- 使用：

  webpack加载loader

  webpack.config.js

  ```js
  const path = require('path')
  
  module.exports = {
      //...
      module:{
          //配置第三方加载器
          rules:[
              //处理样式的loader, 样式都得有['style-loader', 'css-loader']
              //加载器执行顺序顺序是右到左
              {test:/\.css$/, use:['style-loader', 'css-loader']},
              {test:/\.less$/, use:['style-loader', 'css-loader', 'less-loader']},
              {test:/\.scss$/, use:['style-loader', 'css-loader', 'sass-loader']},
          ]
      }
  }
  ```

  main.js中引入

  ```js
  import './css/index.css'
  
  import './css/index.less'
  
  import './css/index.scss'
  ```

  页面引用

  ```js
  //和普通css使用基本没有分别
  ```

#### 3.3.2文件加载器

- webpack对url和图片是不支持加载的，故需要第三方加载器支持

- 加载器：url-loader, file-loader

- 使用：

  webpack.config.js

  ```js
  const path = require('path')
  
  module.exports = {
     //...
      module:{
          //配置第三方加载器
          rules:[
              //处理url的loader
              {test:/\.(jpg|png|gif|bmp|jpeg)$/, use:'url-loader'},
              
              //若给use添加参数limit，则此处的参数表示为一种限制
              	//图片>=限制，则会转为base64的字符串
              	//图片<限制，不会被转为base64的字符串
              {test:/\.(jpg|png|gif|bmp|jpeg)$/, use:'url-loader?limit=6000'},
              //给图片指定名称
              	//给use添加参数name,直接指定图片处理后的名称
              	//ext,后缀名
              	//缺点：若有不同图片命名为同一个名称他们的文件内容会统一，即就是只会保存一个
               {test:/\.(jpg|png|gif|bmp|jpeg)$/, use:'url-loader?limit=6000?name=[name].[ext]'}
              	//图片同名处理办法,添加标识符，例如：添加hash值拼接图片名
              {test:/\.(jpg|png|gif|bmp|jpeg)$/, use:'url-loader?limit=6000?[hash:8]-name=[name].[ext]'}
          ]
      }
  }
  
  ```

  可在css中引用

  ```js
  //引用方式，url,举例如下
  background-image: url('../image/p01.jpg');
  ```

#### 3.3.3 引入外部图标

- 使用bootstrop
- 

++ 插件安装指令：

```sql
--jquery
npm i -D jquery

--webpack
npm i webpack -D
    --webpack-cli
    npm i webpack-cli -D
    --webpack-dev-server
    npm i webpack-dev-server

--html-webpack-plugin

--style
npm i style -D
    --style-loader
    npm i style-loader -D

--css
npm i css -D
    --css-loader
    npm i css-loader -D

--less(less-loader的内部依赖)
cnpm i less -D
    --less-loader
    cnpm i less-loader -D

--sass
npm i sass -D
    --sass-loader
    cnpm i sass-loader -D
    --node-sass(一般使用npm是无法下载这个的，建议使用cnpm)
    cnpm i node-sass -D 

--url-loader file-loader(file-loader是url-loader的内部依赖)
cnpm i url-loader file-loader -D

--bootstrap
cnpm i bootstrap -S
    --popper
    cnpm i popper -S
    --由于4.x版本icon文件分离出去,若使用的是4.X版本的
    npm i https://github.com/iconic/open-iconic.git -D 
```

- 下载报错

  ```sql
  -- KIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: want...
  这是warning错误，是因为mac下需要 fsevents，在windows或linux环境下，请忽略这个错误
  -- peerDependencies WARNING sass-loader@^8.0.0 requires a peer of node-sass@^4.0.0 but none was installed
  类似的错误都是缺少对应的解析插件的原因，可能是安装有问题，也可能是没引入或者没安装，针对于没安装，一般使用<kbd>cnpm i {此处替换为@符号前的部分} -D,或者参考上面的安装命令
  ```

# 4.babel

> 此组件提供一些语法的识别，处理

- 需要两个组件

  ```js
  cnpm i babel-core babel=-loader babel-plugin-transform -D
  cnpm i babel-preset-env babel-preset-stage-0 -D
  ```

  - 配置：webpack.config.js

    ```js
    //配置babel加载器的同时，设定其忽略node_moudles下的包，以避免将第三方包再次打包，浪费资源，另外项目也会受影响
    {test:/\.js$/, use:'babel-loader', exclude:/node_moudles/}
    ```

  - 项目根目录新建：.babelrc（babel的配置文件，同时内容要求符合json格式）

    ```json
    {
    	"presets":["env", "stage-0"],
    	"plugins":["transfom-routime"]
    }
    ```

    