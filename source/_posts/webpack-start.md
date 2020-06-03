---
title: Webpack配置-基础配置
date: 2020-05-13 14:20:34
tags: webpack
---

## 创建

新建空的目录webpack-first

初始化package.json文件：进入webpack-first文件夹运行 `npm init -y`命令

在项目内安装webpack：命令行运行 `npm install webpack webpack-cli --save-dev`

新建src目录，在src目录下新建index.js文件

安装完成，打印一下webpack版本：命令行执行`./node_modules/.bin/webpack`命令

![打印webpack版本](/img/webpack/w1.png)

## 尝试运行

webpack的配置文件是在根目录下webpack.config.js文件里面配置

webpack的配置主要配置的是：

1. 基础功能：入口，出口文件配置

2. 转换器loader

3. 插件plugins

在根目录新建webpack.config.js这个文件，简单配置一下

```javascript
// webpack.config.js
const path = require('path');

const webpackConfig = {
  mode: 'production',
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'), //必须是绝对路径
    filename: 'boundle.js',
  },
}
module.exports = webpackConfig;
```

在src新建helloWorld文件

```javascript
// ./src/helloWorld.js
export function helloWorld() {
  return 'hello world';
}
```

index.js引入helloWorld

```javascript
// src/index.js
import { helloWorld } from './helloWorld';

document.write(helloWorld());
```

运行 `./node_modules/.bin/webpack`，不添加任何参数就是不指定配置文件，打包结果会在根目录下面的dist目录生成一个boundle.js文件

### 每次运行`./node_modules/.bin/webpack`一长串太麻烦了

./node_modules/.bin/目录下存放的是该项目所有依赖的软连接，所有需要从./node_modules/.bin/目录启动webpack来打包项目，实际上呢package.json文件可以默认读取到./node_modules/.bin/目录下的命令，在package.json增加一个scripts配置，执行build命令的时候会去./node_modules/.bin/目录下寻找webpack命令，然后执行

```json
{
  "name": "webpack-first",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.11"
  }
}
```

现在可以直接执行`npm run build`打包项目了

## 简单了解webpack核心概念

### entry指定打包的入口

webpack是模块打包器，一切皆模块，不仅是代码模块，图片、字体、文本都会当成一个个模块，模块之间的依赖关系根据入口文件entry寻找，生成一颗依赖树，在打包的过程中如果发现依赖关系，就把文件加入依赖树中，最终生成打包后的资源

单入口的entry是一个字符串

```javascript
entry: './src/index.js',
```

多入口的entry是一个对象

```javascript
entry: {
  app: './src/index.js',
  pc: './src/pc.js'
},
```

### output指定webpack编译打包后的文件如何输出到磁盘

单入口对应单个的output配置

```javascript
output: {
  filename: '',
  path: ''
},
```

多入口对应多个output配置

```javascript
output: {
  filename: '[name].js',
  path: ''
},
```

### 尝试多入口配置

在src目录下新建一个login.js文件作为另外一个入口文件，随便写点什么可执行的代码进去

然后修改webpack.config.js这个文件，修改entry字段和output字段以配置多入口

```javascript
const path = require('path');

const webpackConfig = {
  ...
  entry: {
    index: './src/index.js',
    login: './src/login.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'), // 必须是绝对路径
    filename: '[name].js', // 使用占位符为每个入口打包之后的文件命名
  },
}

module.exports = webpackConfig;
```

运行`rm -rf dist/`删除dist目录下已经生成的打包文件，再次执行npm run build 会发现dist目录下面新生成了两个文件：index.js,login.js

### loaders

对于webpack不能解析的文件（es6,7以及更高版本，less文件，image文件等）通过loaders先转换一下

基本用法：

```js

const webpackConfig = {
  ...
  module: {
    rules: [
      {
        test: /\.txt$/, // 匹配规则
        use: 'css-loader' // 要使用的loader
      }
    ]
  }
}

module.exports = webpackConfig;
```

### plugins

用于优化打包出来的文件，资源管理和环境变量的注入，作用于整个构建过程

### mode  webpack4新提出的概念

指定当前的构建环境：production（默认），development，none

参数|描述|解释
-|-|-
development|设置process.env.NODE_ENV设置为development，开启 NamedChunksPlugin和NamedModulesPlugin|NamedChunksPlugin：把chunk id变为一个字符串标识符。NamedModulesPlugin：当开启 HMR 的时候使用该插件会显示模块的相对路径，建议用于开发环境。
production|设置process.env.NODE_ENV设置为production，开启 FlagDependencyUsagePlugin, FlagIncludedChunksPlugin, ModuleConcatenationPlugin, NoEmitOnErrorsPlugin, OccurrenceOrderPlugin, SideEffectsFlagPlugin 和 UglifyJsPlugin|启用生产环境适用的插件，比如代码压缩，去注释等

## 了解webpack基础用法

### 在浏览器中查看页面

为了查看webpack基础用法配置之后的页面，需要 html 文件，可以使用 html-webpack-plugin 插件来帮助。

首先安装一下：`npm install html-webpack-plugin -D`

新建 public 目录，并在其中新建一个 index.html 文件( 文件内容使用 html:5 快捷生成即可)

修改 webpack.config.js 文件。

```javascript
//首先引入插件
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
    //...
    plugins: [
        //数组 放着所有的webpack插件
        new HtmlWebpackPlugin({
            template: './public/index.html',
            filename: 'index.html', //打包后的文件名
            minify: {
                removeAttributeQuotes: false, //是否删除属性的双引号
                collapseWhitespace: false, //是否折叠空白
            },
            // hash: true //是否加上hash，默认是 false
        })
    ]
}
```

执行npm run build，dist目录下新生成一个index.html的文件，打开dist目录，双击在浏览器打开这个文件，可以看到页面内显示的hello world文本

### Babel-loader 解析es6和jsx

#### babel如何解析es语法

首先安装：`npm i @babel/core @babel/preset-env babel-loader -D`

Babel-loader执行依赖Babel的配置文件.babelrc，在根目录新建.babelrc文件，写入代码：

```javascript
// .babelrc
{
  "presets": ["@babel/preset-env"] //预设一组es6的插件
}
```

然后修改webpack.config.js这个文件，增加babel-sloader的配置，手动设置一下`mode: 'development'`，一会打包的时候可以看到打包出来的代码不会被压缩

```javascript
// webpack.config.js
...
const webpackConfig = {
  ...
  mode: 'development',
  module: {
    rules: [{
      test: /\.js$/,
      use: 'babel-loader'
    }]
  }
}
module.exports = webpackConfig;
```

测试一下：打开之前新建的src/login文件，修改代码：特意使用es6的箭头函数

```javascript
// src/login
const login = () => {
  console.log('login page');
}
```

执行npm run build可以看到在dist目录下构建出的login.js里面的函数被转换成普通的function函数

```javascript
// dist/login.js代码片段
/***/ "./src/login.js":
/*!**********************!*\
  !*** ./src/login.js ***!
  \**********************/
/*! no static exports found */
/***/ (function(module, exports) {

eval("var login = function login() {\n  console.log('login page');\n};\n\n//# sourceURL=webpack:///./src/login.js?");

/***/ })

/******/ });
```

#### Babel如何转换jsx或者vuex语法

为了测试babel解析jsx语法，首先安装一下react以及要解析jsx所需要的的babel/preset-react：`npm i react react-dom @babel/preset-react -D`

安装完成在.babelrc文件增加preset-react配置

```javascript
{
  "presets": [..., "@babel/preset-react"]
}
```

为了测试效果，修改src/index.js文件的代码：

```javascript
import react from 'react';
import reactdom from 'react-dom';

class Index extends react.Component {
  render () {
    return <p>hello world react</p>
  }
}

reactdom.render(<Index />, document.getElementById('root'))
```

在public/index.html新增target节点 id="root"

```html
<!-- public/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <!-- 新增 -->
  <div id="root"></div>
</body>
</html>
```

运行npm run build，在浏览器刷新index.html页面，发现网页显示的文本变成hello world react

### 解析css

#### css-loader用于加载.css文件，并且转换成commonjs文件，style-loader将样式通过style标签插入到head中

首先安装css-loader，style-loader: `npm i style-loader css-loader -D`

新建src/index.css文件，写入样式

```css
.text {
  color: pink;
}
```

然后在index.js文件中引入这个样式，并且修改react组件使用这个样式:

```javascript
...
import styles from './index.css';

class Index extends react.Component {
  render () {
    return <p className="text">hello world react</p>
  }
}
...
```

最后不要忘了在webpack.config.js文件配置css-loader

```javascript
// webpack.config.js
...
const webpackConfig = {
  ...
  mode: 'development',
  module: {
    rules: [
      ...
      {
        test: /\.css$/,
        use: ['style-loader','css-loader']
      }
    ]
  },
  ...
}
module.exports = webpackConfig;
```

注意loader的解析是链式解析，总右往左，所以上面的loader的执行顺序是先使用css-loader解析css，然后将解析好的文件传递给style-loader再进行下一步的工作

运行npm run build，在浏览器刷新index.html，可以看到文字变成了粉色

#### 如何使用less-loader和sass-loader

less-loader和sass-loader的使用和css-loader的使用方法类似，以下以less为例，less-loader就是将less.css转换成css，

首先安装less 和 less-loader：`npm i less less-loader -D`

修改刚才的index.css文件后缀名为index.less，并写点简单的样式：

```less
// index.less
@base: #f938ab;
.text {
  color: @base;
}
```

index.js中引入index.less

在webpack.config.js文件配置less-loader

```javascript
// webpack.config.js
...
const webpackConfig = {
  ...
  mode: 'development',
  module: {
    rules: [
      ...
      {
        test: /\.less$/,
        use: ['style-loader', 'css-loader', 'less-loader']
      }
    ]
  },
  ...
}
module.exports = webpackConfig;
```

运行npm run build，在浏览器刷新index.html，可以看到文字变成新设置的颜色

### 解析图片和字体资源

#### file-loader解析图片文件

使用`file-loader`解析文件资源，首先安装一下:`npm i file-loader -D`，安装完成之后，在src下新建一个img文件夹，用来存放图片资源，照一张漂亮的图片放进img文件夹

在index.js中引用这张图片：

```javascript
import xiaozhan from './img/zhanzhan.jpg'

class Index extends React.Component {
  render () {
    return (<>
      <img src={xiaozhan} />
    </>)
  }
}
```

然后在webpack.config.js文件配置file-loader，匹配一些常用的图片文件后缀

```javascript
// webpack.config.js
...
const webpackConfig = {
  ...
  mode: 'development',
  module: {
    rules: [
      ...
      {
        test: /\.(png|jpg|gif|jpeg)$/,
        use: ['file-loader']
      }
    ]
  },
  ...
}
module.exports = webpackConfig;
```

运行npm run build，在浏览器刷新index.html，可以看到页面增加了一个图片文件

![图片](/img/webpack/w3.png)

#### file-loader解析字体

字体文件的解析与图片文件一样可以使用file-loader

```javascript
// webpack.config.js
...
const webpackConfig = {
  ...
  mode: 'development',
  module: {
    rules: [
      ...
      {
        test: /\.(woff|woff2|eot|otf|ttf)$/,
        use: ['file-loader']
      }
    ]
  },
  ...
}
module.exports = webpackConfig;
```

#### 解析图片和字体文件也可以使用url-loader

url-loader可以较小资源自动base64转换，看一下使用的效果

首先安装：`npm i url-loader -D`

然后修改webpack.config.js的配置，把file-loader替换成url-loader

```javascript
// webpack.config.js
...
const webpackConfig = {
  ...
  mode: 'development',
  module: {
    rules: [
      ...
      {
        test: /\.(png|jpg|gif|jpeg)$/,
        use: {
          loader: 'url-loader',
          options: {
            limit: 10240, // 图片小于10k，webpack在打包的时候会自动用base64转换
          }
        }
      }
    ]
  },
  ...
}
module.exports = webpackConfig;
```

找一个小于10k的图片测试一下，首先看一下使用file-loader打包的文件，dist目录下打包出来了图片文件，index.js是848k，图片文件6.9k

![图片](/img/webpack/w4.png)

然后删除掉dist目录，使用url-loader的配置再打包一次，

![图片](/img/webpack/w5.png)

可以看到，dist目录下打包出来的文件并没有图片文件，而index.js文件变成了857k，因为图片使用base64转行进去了，刷新页面，看到的页面和刚才一样的

### webpack中文件的监听

刚才一系列操作，每次做点什么代码上的改动都要手动build，手动刷新页面，极其不方便

文件的监听就是在发现源代码发生变化的时候，自动重新构建出新的输出文件

webpack中开启监听有两种方式：

+ 启动webpack命令时带上 --watch参数，在package.json增加一个scripts配置

```json
{
  ...
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "watch": "webpack --watch"
  },
  ...
}
```

修改了之后，在命令行输入 npm run watch，命令行提示 ：webpack is watching the files…

![图片](/img/webpack/w6.png)

然后随便做点修改，保存，webpack就会监听到文件发生了变化，就会重新构建出新的输出文件，并在命令行生成一个记录

![图片](/img/webpack/w7.png)

这个时候在手动刷新一些浏览器，可以看到浏览器显示了修改之后的效果，这个方法的缺陷就是还是需要手动刷新浏览器

+ webpack.config.js中配置watch: true

```javascript
// webpack.config.js
...
const webpackConfig = {
  ...
  watch: true, //默认false
  watchOptions: {
    ignored: /node_modules/, //不监听的文件夹，支持正则匹配
    aggregateTimeout: 300, //监听到发生变化后等待300秒再去执行
    poll: 1000 //每秒询问1000次文件是否发生变化
  },
  ...
}
module.exports = webpackConfig;
```

watch: true原理：轮询判断文件的最后编辑时间是否发生变化，设置了aggregate: 300,，检测到某个文件发生了变化，不会立即执行，而是先缓存起来，等待300秒之后再去执行，如果在此期间其他的文件也发生了变化，会一起打包

在命令行运行npm run build，wabpack会进入监听模式。这个方法的缺陷也是需要手动刷新浏览器，所以就需要热更新

### 热更新

#### webpack-dev-server

webpack-dev-server配合HotModuleReplacementPlugin插件一起使用，以达到热更新的目的

webpack-dev-server：不刷新浏览器，不输入文件，而是放在内存中，构建速度有一定的优势

安装： `npm install webpack-dev-server `

在package.json文件增加一个scripts配置：

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "watch": "webpack --watch",
    "dev": "webpack-dev-server --open"
  },
```

webpack-dev-server在开发过程中是不需要使用的，所以修改webpack.config.js文件中的mode配置为development，然后引入webpack内置插件：HotModuleReplacementPlugin插件，同时设置devServer

```javascript
// webpack.config.js
const webpack = require('webpack');
...
const webpackConfig = {
  ...
  mode: 'development',
  plugins: [
    ...
    new webpack.HotModuleReplacementPlugin()
  ],
  devServer: {
    contentBase: './dist', // 服务基础目录
    hot: true // 开启热更新
  }
}
module.exports = webpackConfig;
```

设置好了之后在命令行运行npm run dev，webpack会自动在浏览器打开页面，尝试修改，浏览器也会自动刷新修改后的内容

另外一种热更新的方式webpack-dev-middleWare

#### webpack-dev-middleWare

webpack-dev-middleWare将输出的文件传送给服务器

#### 热更新原理

![图片](/img/webpack/w8.png)

热更新的过程

1. 启动阶段，在文件系统里把文件用webpack compile进行编译，把编译好的文件传输给bundle server,bundle server让打包出来的文件以server的方式浏览器能够访问的到，

2. 如果源代码文件发生了修改，会再次用webpack compile进行编译，编译完成后将代码发送给HMR Server，HMR Server就会知道哪些模块发生了改变，然后通知HMR Runtime哪些文件发生了变化，HMR Runtime负责更新代码，以达到不刷新浏览器改变代码的目的

### 文件指纹

打包后输出文件的后缀，做文件的版本的管理，在发布版本的时候，可以只更新有改动的文件，对于没有修改的文件，可以继续使用浏览器的缓存

![图片](/img/webpack/w9.png)

由于在开发换进和生产环境可能需要不同的webpack配置，例如热更新只需要在开发环境使用，而文件指纹hash是在开发环境支持的，所有需要区分一下生产环境和开发环境的webpack配置，做法就是首先把之前的webpack.config.js改成webpack.dev.js作为开发环境的配置使用，然后新建名为webpack.prod.js文件，作为开发环境的配置使用

```javascript
// webpack.prod.js
const path = require('path');

const HtmlWebpackPlugin = require('html-webpack-plugin');

const webpackConfig = {
  mode: 'production',
  entry: {
    index: './src/index.js',
    login: './src/login.js'
  },
  output: {
    path: path.resolve(__dirname, 'dist'), //必须是绝对路径
    filename: '[name].js',
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader'
      },
      {
        test: /\.css$/,
        use: ['style-loader','css-loader']
      },
      {
        test: /\.less$/,
        use: ['style-loader','css-loader', 'less-loader']
      },
      {
        test: /\.(png|jpg|gif|jpeg)$/,
        use: {
          loader: 'url-loader',
          options: {
            limit: 10240, // 图片小于10k，webpack在打包的时候会自动用base64转换
          }
        }
        // use: 'file-loader'
      },
      {
        test: /\.(woff|woff2|eot|otf|ttf)$/,
        use: ['file-loader']
      }
    ]
  },
  plugins: [
    //数组 放着所有的webpack插件
    new HtmlWebpackPlugin({
        template: './public/index.html',
        filename: 'index.html', //打包后的文件名
        minify: {
            removeAttributeQuotes: false, //是否删除属性的双引号
            collapseWhitespace: false, //是否折叠空白
        },
        // hash: true //是否加上hash，默认是 false
    }),
  ],
}
module.exports = webpackConfig;
```

接着修改package.json，指定开发环境使用webpack.dev.js，生产环境使用webpack.prod.js

```json
// package.json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack --config webpack.prod.js",
    "watch": "webpack --watch",
    "dev": "webpack-dev-server --config webpack.dev.js --open"
  },
```

#### JS文件指纹的设置

使用ChunkHash，通过设置output的filename哈希值，因为文件指纹是在生产环境使用的，所有在webpack.prod.js配置里面修改

```javascript
// webpack.prod.js
const path = require('path');

const webpackConfig = {
  // ...
  output: {
    path: path.resolve(__dirname, 'dist'), //必须是绝对路径
    filename: '[name]_[chunkhash:8].js',
  },
  // ...
}
module.exports = webpackConfig;
```

运行npm run build 可以看到dist目录下构建出的js文件带有后缀名hash

![图片](/img/webpack/w10.png)

在src/index.js中修改一点文字，再次运行npm run build ，查看dist目录，文件夹里有两个构建出来的index.js文件带有不同的hash后缀，其中一个就是第二次构建的新文件

![图片](/img/webpack/w11.png)

#### 图片、其他文件指纹设置

使用hash，图片使用file-loader的option配置添加指纹

```javascript
// webpack.prod.js

const webpackConfig = {
  // ...
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif|jpeg)$/,
        use: {
          loader: 'file-loader',
          options: {
            name: '[name]_[hash:8].[ext]'
          }
        }
      },
    ]
  },
  // ...
}
module.exports = webpackConfig;
```

添加好之后，运行npm run build，查看dist目录，可以看到构建出的图片文件添加了hash后缀

#### css文件指纹，使用contentHash

前面提到解析css文件使用的style-loader的作用是将样式通过style标签插入到head中，css文件并没有被提构建成一个独立的文件的，所以目前是不能为css文件添加指纹

那么就需要借助MiniCssExtractPlugin这个插件来帮助在构建的时候把css文件提取成一个文件，并添加指纹

首先安装一下： `npm i mini-css-extract-plugin -D`

然后把插件添加到plugin的数组里去，删除style-loader，使用MiniCssExtractPlugin.loader：

```javascript
// webpack.prod.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

const webpackConfig = {
  // ...
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader,'css-loader']
      },
      {
        test: /\.less$/,
        use: [MiniCssExtractPlugin.loader,'css-loader', 'less-loader']
      },
    ]
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: '[name]_[contenthash:8].css'
    })
  ],
  // ...
}
module.exports = webpackConfig;
```

运行npm run build查看效果

### 代码压缩

#### js代码压缩

使用uglifyjs-webpack-plugin，插件uglifyjs-webpack-plugin是webpack4内置的插件，所以正常build构建出的代码就是默认压缩的

#### css压缩

使用optimize-css-assets-webpack-plugin插件进行压缩，同时需要安装css预处理器cssnano

首先安装下：`npm i optimize-css-assets-webpack-plugin -D`

安装css预处理器：`npm i cssnano -D`

在webpack.prod.js中配置使用

```javascript
// webpack.prod.js
const OpimizeCssAssetsPlugin = require('optimize-css-assets-plugin');

const webpackConfig = {
  // ...
  plugins: [
    // ...
    new OpimizeCssAssetsPlugin({
      AccetNameRegExp: /\.css$/g,
      cssProcessor: require('cssnano')
    })
  ],
  // ...
}
module.exports = webpackConfig;
```

测试效果，删除dist目录，执行npm run build，查看dist目录下构建的css文件

#### html文件压缩

前面用到的HtmlWebpackPlugin，可以设置压缩html的参数，压缩空格、换行符号、注释，一个页面对应一个HtmlWebpackPlugin，设置`collapseWhitespace: true`压缩空格和换行，removeComments: true删除注释

```javascript
// webpack.prod.js
const HtmlWebpackPlugin = require('html-webpack-plugin');

const webpackConfig = {
  // ...
  plugins: [
    // ...
    new HtmlWebpackPlugin({
        template: './public/index.html',
        filename: 'index.html', //打包后的文件名
        minify: {
            removeAttributeQuotes: false, //是否删除属性的双引号
            collapseWhitespace: true, //是否折叠空白
            removeComments: true, //删除注释
        },
        // hash: true //是否加上hash，默认是 false
    }),
  ],
  // ...
}
module.exports = webpackConfig;
```

运行npm run build，在dist目录下找到对应的index.html文件检测效果
