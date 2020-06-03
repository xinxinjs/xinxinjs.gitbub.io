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

摇树优化，一个模块可能有多个方法，只要有其中一个方法被用到了，整个模块就会被打包到
