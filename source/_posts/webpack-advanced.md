---
title: webpack-进阶配置
date: 2020-05-27 16:27:33
tags: webpack
---

## 每次构建之前首先自动删除dist目录

在学习基础配置的时候，给webpack配置添加了文件指纹来做文件的版本管理，如果某个文件有改动，就会生成新的构建文件，故此会在一次次的构建过程中dist目录会留下很多版本的文件，所有每次构建之前不得不手动删除dist目录，很是麻烦

webpack提供了一个插件，可以在每次构建之前首先自动删除dist目录：clean-webpack-plugin插件

首先安装一下：`npm i clean-webpack-plugin -D`

```javascript
// webpack.prod.js
const {CleanWebpackPlugin} = require('clean-webpack-plugin');
const webpackConfig = {
  // ...
  plugins: [
    // ...
    new CleanWebpackPlugin()
  ],
}

module.exports = webpackConfig;
```

之后运行npm run build，webpack都会先删除dist，在构建

loader是有严格的顺序的，plugin的顺序要求并不严格，不放心的可以把CleanWebpackPlugin插件放在plugin数组的第一个

## 自动补全css3前缀

有时候为了使用最新的css样式，需要为不同的浏览器手动的补齐带有各自浏览器内核的前缀，webpack提供的postcss-loader和autoprefixer插件，可以设置为不同的浏览器自动补齐前缀

安装： `npm i postcss-loader autoprefixer -D`

然后在webpack.prod.js中配置：找到.less的loader配置，添加一个postcss-loader

```javascript
// webpack.prod.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const webpackConfig = {
  // ...
  module: {
    rules: [
      // ...
      {
        test: /\.less$/,
        use: [MiniCssExtractPlugin.loader,'css-loader', {
          loader: 'postcss-loader', options: {
            plugins: () => [
              require('autoprefixer')({
                overrideBrowserslist: ['last 2 version', '>1%', 'ios 7']
              })
            ]
          }
        }, 'less-loader']
      },
    ]
  },
}

module.exports = webpackConfig;
```

在src/index.less样式文件中写一个比较新的样式来进行测试

```css
/* src/index.less */
@base: #f938ab;
.text {
  color: @base;
  display: flex;
}
```

运行npm run build，查看dist/index.css-loader自动添加了浏览器前缀：

```css
/* dist/index.less */
.text{color:#f938ab;display:-webkit-box;display:-webkit-flex;display:-ms-flexbox;display:flex}
```

postcss-loader要求在css-loader之前执行，所有在数组中的顺序必须要放在css-loader之后，否则运行会报错

## 自动转换px为rem

为了兼容移动设备不同屏幕的分辨率，适配页面，可以使用rem这个单位来编写样式，webpack提供了px2rem-loader这个loader可以帮助px转换为rem单位

借助手淘的一个lib-flexible库，可以在页面渲染时动态计算出跟元素的font-size值

安装：`npm i px2rem-loader -D`

安装：`npm i lib-flexible -S`

在webpack.prod.js中添加配置：

```javascript
// webpack.prod.js
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const webpackConfig = {
  // ...
  module: {
    rules: [
      // ...
      {
        test: /\.less$/,
        use: [MiniCssExtractPlugin.loader,'css-loader', {
          loader: 'postcss-loader', options: {
            plugins: () => [
              require('autoprefixer')({
                overrideBrowserslist: ['last 2 version', '>1%', 'ios 7']
              })
            ]
          }
        }, 'less-loader', {
          loader: 'px2rem-loader',
          options: {
            remUnit: 75, // 转换率：1rem = 75px
            remPrecision: 8 // 转换之后保留小数点后位数
          }
        }]
      },
    ]
  },
}

module.exports = webpackConfig;
```

在src/index.less样式文件中写一个宽度的样式来进行测试

```css
/* src/index.less */
.text {
  /* ... */
  width: 500px;
}
```

运行npm run build，查看dist/index.less，

```css
/* dist/index.less */
.text{width:6.66666667rem}
```

现在可以看到px已经被转换成rem单位，然后还需要根据不同的屏幕尺寸，设定不同的跟元素字体，这个就需要使用已经安装好的lib-flexible，

lib-flexible是一个比较稳定的库，属于静态资源，所有现在要学习一下如何把静态资源内联进构建目录

## 静态资源内联

![图片](/img/webpack/w12.png)

### html和js内联

使用raw-loader

`npm i raw-loader@0.5.1 -D`

## 多页面应用打包

每次增加一个新的页面入口的时候，都需要手动的修改webpack 的配置文件，修改entry字段，增加入口配置，修改plugin里面的HtmlWebpackPlugin配置，因为每个html入口文件对应一个HtmlWebpackPlugin，这样子及其不方便，

推荐使用glob这个库，通过约定文件目录，glob可以动态找到要配置的入口文件，借助这个可以动态生成entry，和HtmlWebpackPlugin数组，而不需要在每次添加一个entry的时候都去手动的修改webpack配置。

首先来整理一下目录：有两个entry，分别是index.js和search.js，在src下新建一个index文件夹，把之前在src下的index.js和index.less移动到index文件夹下面，在index文件夹下新建一个index.html作为index.js生成html文件的模版文件，可以直接从原来的public/index.html复制过来，新建一个src/search文件夹，在这个文件夹下新建模版文件index.html，入口文件index.js，index.less样式文件，

最后的文件目录：

![图片](/img/webpack/w13.png)

安装glob: `npm i glob -D`

修改webpack.prod.js

```javascript
// webpack.prod.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const {CleanWebpackPlugin} = require('clean-webpack-plugin');

// 动态设置entry 和 htmlWebpackPlugin
const glob = require('glob');
setMPA = () => {
  const entry = {};
  const htmlWebpackPlugin = [];

  const entryFiles = glob.sync(path.join(__dirname, './src/*/index.js'));
  Object.keys(entryFiles).map((index) => {
    const entryFile = entryFiles[index];
    const match = entryFile.match(/src\/(.*)\/index\.js/);
    const pageName = match && match[1];
    entry[pageName] = entryFile;
    htmlWebpackPlugin.push(
      new HtmlWebpackPlugin({
        template: path.join(__dirname, `src/${pageName}/index.html`),
        filename: `${pageName}.html`, //打包后的文件名
        chunks: [pageName],
        inject: true,
        minify: {
          html5: true,
          preserveLineBreaks: false,
          removeAttributeQuotes: false, //是否删除属性的双引号
          collapseWhitespace: true, //是否折叠空白
          removeComments: true,
          minifyCSS: true,
          minifyJS: true
        },
    })
    )
  })

  return {
    entry, htmlWebpackPlugin
  }
}

const {entry, htmlWebpackPlugin} = setMPA();

const webpackConfig = {
  mode: 'production',
  entry: entry,
  output: {
    path: path.resolve(__dirname, 'dist'), //必须是绝对路径
    filename: '[name]_[chunkhash:8].js',
  },
  plugins: [
    //数组 放着所有的webpack插件
    new CleanWebpackPlugin(),
    ...htmlWebpackPlugin,
    // ...
  ],
}

module.exports = webpackConfig;
```

再次运行npm run build，查看dist目录，可以检测效果，假如以后要增加一个entry，只需要按照约定新建目录，即可，无需修改webpack配置文件

同样到配置代码，复制一份到webpack.dev.js中

## source map

[JavaScript Source Map 详解](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)

## 提取页面公共资源

提取页面公共资源可以减少构建文件的体积

### 基础库的分离

使用html-webpack-externals-plugin，react\react-dom通过cdn引入，不打包进bundle中

安装：`npm i html-webpack-externals-plugin -D`，

在webpack.prod.js配置文件中使用：

```javascript
// webpack.prod.js
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');

const webpackConfig = {
  // ...
  plugins: [
    new HtmlWebpackExternalsPlugin({
      externals: [{
        module: 'react',
        entry: 'https://unpkg.com/react@16/umd/react.development.js',
        global: 'React'
      },{
        module: 'react-dom',
        entry: 'https://unpkg.com/react-dom@16/umd/react-dom.development.js',
        global: 'ReactDom'
      }]
    })
  ],
}

module.exports = webpackConfig;
```

没有配置之前的构建文件先看一下，构建出来的js文件129k,

![图片](/img/webpack/w14.png)

配置之后的构建文件，js文件10k不到，差别还是很明显的

![图片](/img/webpack/w15.png)

最后分别在入口文件的模版html中引入react/react-dom：

```html
<!-- src/index/index.html -->
<!-- src/search/index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <script type="text/javascript" src="https://unpkg.com/react@16/umd/react.development.js"></script>
  <script type="text/javascript" src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
</head>
<body>
  <!-- zhengwen -->
  <div id="root"></div>
</body>
</html>
```

### SplitChunksPlugin 基础包的分离

把react和react-dom进行单独的分离提取出来，叫vendors.js

```javascript
// webpack.prod.js
// ...
const glob = require('glob');
setMPA = () => {
  const entry = {};
  const htmlWebpackPlugin = [];

  const entryFiles = glob.sync(path.join(__dirname, './src/*/index.js'));
  Object.keys(entryFiles).map((index) => {
    const entryFile = entryFiles[index];
    const match = entryFile.match(/src\/(.*)\/index\.js/);
    const pageName = match && match[1];
    entry[pageName] = entryFile;
    htmlWebpackPlugin.push(
      new HtmlWebpackPlugin({
        template: path.join(__dirname, `src/${pageName}/index.html`),
        filename: `${pageName}.html`, //打包后的文件名
        chunks: ['vendors', pageName],
        inject: true,
        minify: {
          html5: true,
          preserveLineBreaks: false,
          removeAttributeQuotes: false, //是否删除属性的双引号
          collapseWhitespace: true, //是否折叠空白
          removeComments: true,
          minifyCSS: true,
          minifyJS: true
        },
    })
    )
  });

  return {
    entry, htmlWebpackPlugin
  }
}

const webpackConfig = {
  // ...
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /(react|react-dom)/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
}

module.exports = webpackConfig;
```

运行npm run build 查看dist目录，构建出一个vendors.js文件，就是成功提取出的react和react-dom

### SplitChunksPlugin 分离页面公共的代码

通过配置，把页面使用多次的代码提取出一个公共的文件，叫commons.js

```javascript
// webpack.prod.js

// ...

const webpackConfig = {
  // ...
  optimization: {
    splitChunks: {
      minSize: 0,
      cacheGroups: {
        commons: {
          name: 'commons',
          chunks: 'all',
          minChunks: 2 // 最少使用次数
        }
      }
    }
  }
}

module.exports = webpackConfig;
```

运行npm run build 查看dist目录，构建出一个commons.js文件，就是成功提取出的公共的代码

## tree shaking

摇树优化，一个模块可能有多个方法，只要有其中一个方法被用到了，整个模块就会被打包到bundle里面去，tree shaking就是把用到的方法打包进去，多余的代码在uglify阶段就会被清除掉

删除无用代码的工作原理：

- 不会执行到的代码，不会到达的代码

- 代码执行的结果不会被用到

- 代码只写不读

tree shaking原理：利用es6模块的特点，import 、export都是出现在顶层，并且import进来的代码不可修改，tree shaking对模块的代码静态分析，在编译阶段分析哪些代码没有用到，对这些代码进行标记，在uglify阶段把它清除掉

在webpack config的配置文件中，设置`mode: 'production',`就会自动开始tree shaking

## scope hoisting

对于每一个模块打包出来的文件都会有一个包裹的函数，模块数量越多，导致大量的函数闭包导致bundle的体积很大，而且打包的时候还增加内存的消耗

scope hoisting原理：将所有的模块按照引用的顺序包裹在一个函数里，适当的重命名变量防止变量冲突

在webpack config的配置文件中，设置`mode: 'production',`就会自动开始scope hoisting

## 代码分割

代码分割的适用场景：js懒加载，抽离相同的代码到一个共享模块

代码分割搭配动态import使用，动态import并不被原生的es6支持，需要安装一个babel-plugin-syntax-dynamic-import插件

`npm install --save-dev @babel/plugin-syntax-dynamic-import`

在.babelrc文件中配置，引入这个插件

```json
// .babelrc
{
  // ..
  "plugins": ["@babel/plugin-syntax-dynamic-import"]
}
```

写一个动态加载的js函数，在组建中使用这个函数，动态import返回的是一个promise对象：

```jsx
const loadComponent = () => {
  import('./testSync').then((text) => {
    setText(text.default);
  });
}
```

在动态加载的时候使用jsonp的形式加载动态加载的js文件

## 使用ESLint

安装eslint 一套 `npm i eslint eslint-plugin-jsx-a11y eslint-plugin-react eslint-plugin-import -D`

安装eslint-loader: `npm i eslint-loader -D`

安装babel-eslint:`npm i babel-eslint -D`

安装：`npm i eslint-config-airbnb -D`

在根目录新建.eslintrc.js

```javascript
// .eslintrc.js
module.exports = {
  "parser": "babel-eslint",
  "extends": "airbnb",
  "env": {
    "browser": true,
    "node": true
  }
}
```

运行构建

## webpack打包库和组件

新建项目根目录 large-number，进入项目目录,`npm init -y`初始化package.json

安装webpack，webpack-cli:`npm i webpack webpack-cli -D`

在根目录新建src/index.js，写一个库函数，例如大数相加

```javascript
// src/index.js
// 大整数相加
export default function add(a, b) {
  let i = a.length - 1; // 从个位开始相加
  let j = b.length - 1; // b的个位数

  let ret = '';
  let carry = 0;

    while(i >= 0 || j >= 0) {
      let sum = 0;
      let x = 0;
      let y = 0;
      if(i >= 0) {
        x = a[i] - 0;
        i--;
      }
      if(j >= 0) {
        y = b[j] - 0;
        j--;
      } 
      sum = x + y + carry;
      if(sum >= 10) {
        carry = 1;
        sum = sum - 10;
      }else {
        carry = 0;
      }
      ret = sum + ret;
    }
  
  if(carry) {
    return ret = carry + ret;
  }
  return ret;
}

// add('9999', '666');
```

写好之后，在根目录新建webpack.config.js

```javascript
// webpack.config.js
const TerserPlugin = require('terser-webpack-plugin'); //npm i terser-webpack-plugin -D

module.exports = {
  mode: 'none',
  entry: {
    'large-number': './src/index.js',
    'large-number.min': './src/index.js',
  },
  output: {
    filename: '[name].js',
    library: 'largeNumber',
    libraryTarget: 'umd',
    libraryExport: 'default'
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin(
        {include: /\.min\.js$/}
      )
    ]
  }
}
```

修改packagr.json，增加build命令和prepublish

```json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "webpack",
    "prepublish": "webpack" // 发布的时候使用
  },
```

在根目录新建index.js

```javascript
if(process.env.NODE_ENV = 'production') {
  module.exports = require('./dist/large-number.min.js');
}else {
  module.exports = require('./dist/large-number.js');
}
```

npm login登陆npm账号

npm publish发布组件到npm库

## SSR打包

SSR指的是在服务端渲染页面然后返回给前端浏览器做显示

根目录新建server目录，新建server/index.js，编写服务端渲染的代码逻辑

```javascript
if(typeof window === 'undefined') {
  global.window = {};
}

const express = require('express');
const { renderToString } = require('react-dom/server');
const SSR = require('../dist/search-server.js');

const server = (port) => {
  const app = express();
  app.use(express.static('dist'));
  app.get('/search', (req, res) => {
    const html = renderMarkup(renderToString(SSR));
    res.status(200).send(html);
  });
  app.listen(port, () => {
    console.log('Server is running on port:' + port)
  })
}

server(process.env.PORT || 3000);

const renderMarkup = (str) => {
  return `<!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
  </head>
  <body>
    <div id="root">${str}</div>
  </body>
  </html>`
}
```

在根目录新建webpack.ssr.js文件，编写服务端打包的webpack配置

```javascript
const path = require('path');

const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OpimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const {CleanWebpackPlugin} = require('clean-webpack-plugin');
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');

// 动态设置entry 和 htmlWebpackPlugin
const glob = require('glob');
setMPA = () => {
  const entry = {};
  const htmlWebpackPlugin = [];

  const entryFiles = glob.sync(path.join(__dirname, './src/*/index-server.jsx'));
  Object.keys(entryFiles).map((index) => {
    const entryFile = entryFiles[index];
    const match = entryFile.match(/src\/(.*)\/index-server\.jsx/);
    const pageName = match && match[1];
    console.log(entryFile)
    if(pageName) {
      entry[pageName] = entryFile;
      htmlWebpackPlugin.push(
        new HtmlWebpackPlugin({
          template: path.join(__dirname, `src/${pageName}/index.html`),
          filename: `${pageName}.html`, //打包后的文件名
          chunks: ['vendors', pageName],
          inject: true,
          minify: {
            html5: true,
            preserveLineBreaks: false,
            removeAttributeQuotes: false, //是否删除属性的双引号
            collapseWhitespace: true, //是否折叠空白
            removeComments: true,
            minifyCSS: true,
            minifyJS: true
          },
        })
      )
    }
  });

  return {
    entry, htmlWebpackPlugin
  }
}

const {entry, htmlWebpackPlugin} = setMPA();

const webpackConfig = {
  mode: 'production',
  entry: entry,
  output: {
    path: path.resolve(__dirname, 'dist'), //必须是绝对路径
    filename: '[name]-server.js',
    libraryTarget: 'umd'
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: ['babel-loader', 'eslint-loader']
      },
      {
        test: /\.css$/,
        use: [MiniCssExtractPlugin.loader,'css-loader']
      },
      {
        test: /\.less$/,
        use: [MiniCssExtractPlugin.loader,'css-loader', {
          loader: 'postcss-loader', options: {
            plugins: () => [
              require('autoprefixer')({
                overrideBrowserslist: ['last 2 version', '>1%', 'ios 7']
              })
            ]
          }
        }, 'less-loader', {
          loader: 'px2rem-loader',
          options: {
            remUnit: 75, // 转换率：1rem = 75px
            remPrecision: 8 // 转换之后保留小数点后位数
          }
        }]
      },
      {
        test: /\.(png|jpg|gif|jpeg)$/,
        use: {
          loader: 'file-loader',
          options: {
            name: '[name]_[hash:8].[ext]'
          }
        }
      },
      {
        test: /\.(woff|woff2|eot|otf|ttf)$/,
        use: ['file-loader']
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    ...htmlWebpackPlugin,
    new MiniCssExtractPlugin({
      filename: '[name]_[contenthash:8].css'
    }),
    new OpimizeCssAssetsPlugin({
      AccetNameRegExp: /\.css$/g,
      cssProcessor: require('cssnano')
    }),
  ],
  optimization: {
    splitChunks: {
      cacheGroups: {
        commons: {
          test: /(react|react-dom)/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
}

module.exports = webpackConfig;

```

安装：`npm i express -D`

新建src/search/index-server.js，编写一个服务端渲染页面的页面代码，注意使用方法：

```javascript
const React = require('react');

function Index() {
  return (
    <>
      ssr test
    </>
  );
}

module.exports = <Index />;
```

运行npm run build查看dist目录

### 使用打包出来的浏览器端的html文件作为，模版可以显示样式

修改server/index.js

```javascript
...
const fs = require('fs');
const path = require('path');
...

const template = fs.readFileSync(path.join(__dirname, '../dist/search.html'), 'utf-8');

...

const renderMarkup = (str) => {
  // 打包好的模版里面会有注释的占位符，然后用服务端渲染好的代码替换注释
  return template.replace('<!--HTML_PLACEHOLDER-->', str);
}
```

修改search/index.html模版文件，添加一个占位符

```html
<body>
  <div id="root"><!--HTML_PLACEHOLDER--></div>
</body>
```

### 服务端获取数据

新建server/data.json

修改search/index.html模版文件，添加一个数据占位符

```html
<body>
  <!--INITIAL_DATA_PLACEHOLDER-->
</body>
```

修改server/index.js加载数据

```javascript
const data = require('./data.json');

const renderMarkup = (str) => {
  const dataStr = JSON.stringify(data);
  return template.replace('<!--HTML_PLACEHOLDER-->', str).replace('<!--INITIAL_DATA_PLACEHOLDER-->', `<script>window.__initial_data=${dataStr}</script>`);
}
```

把数据加载到页面，后续就可以使用数据渲染页面

## 构建时候命令行的日志展示优化

在webpack.config.js设置stats字段

stats预设值：

![图片](/img/webpack/w16.png)

### Friendly-errors-webpack-plugin

Friendly-errors-webpack-plugin识别某些类别的webpack错误，并清理，聚合和优先级，以提供更好的开发人员体验。

安装：`npm install friendly-errors-webpack-plugin --save-dev`

安装好了之后，在webpack.prod.js和webpack.dev.js中分别引入，并使用

```javascript
const FriendlyErrorsWebpackPlugin = require('friendly-errors-webpack-plugin');

plugins: [
    new FriendlyErrorsWebpackPlugin()
  ],
```

之后在运行npm run build活着npm run dev的时候，在构建成功会用绿色提示，并显示构建所需要的时间，有警告的时候会使用黄色的提示，编译报错的时候使用红色的颜色标记
