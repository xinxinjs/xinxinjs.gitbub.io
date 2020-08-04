---
title: webpack构建速度、体积分析和优化
date: 2020-06-16 15:11:30
tags: webpack
---

![图片](/img/webpack/36.png)

## 初级分析：构建体积和速度

基本的分析可以使用webpack内置的stats进行构建的统计分析，例如总共消耗的时间和每个构建资源的大小，使用方法就是在根目录的package.json方法里面添加运行脚本：

```json
"scripts": {
    "build:stats": "webpack --config webpack.prod.js --json > stats.json"
  },
```

运行npm run build:stats会在根目录生成一个stats.json文件，里面包含构建资源的信息，是否构架成功，大致用了多少时间构建，但是这个分析只是作为一个基础的分析，颗粒度泰国粗糙

## 精细分析：构建速度分析

借助speed-measure-webpack-plugin 插件分析详细的构建速度，可以清晰的看到每个loader和plugin的耗时情况

安装：`npm i speed-measure-webpack-plugin -D`

在webpack.prod.js中引入并使用：

```javascript
// webpack.prod.js
// ...
const SpeedMeasireWebpackPlugin = require('speed-measure-webpack-plugin');

const smp = new SpeedMeasireWebpackPlugin();

// ...

const webpackConfig = smp.wrap({
  // ...
});

module.exports = webpackConfig;
```

在终端运行npm run build，结果：

![图片](/img/webpack/w22.png)

速度正常的会用绿色显示，速度稍慢的会用黄色表示，速度很慢需要重点关注的会用红色表示，然后可以做一下调整的优化方便的工作

## 构建体积的分析

借助webpack-bundle-analyzer可以分析构建体积

安装：`npm install webpack-bundle-analyzer -D`

在webpack.prod.js中引入并使用：

```javascript
// webpack.prod.js
// ...
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

// ...

const webpackConfig = smp.wrap({
  // ...
  plugins: [
    // ...
    new BundleAnalyzerPlugin(),
  ],
  // ...
});

module.exports = webpackConfig;
```

运行npm run build 会在浏览器自动打开一个http://127.0.0.1:8888/ 页面，页面里面展示的是构建包的体积大小，可以针对性的找出是哪个包的体积大，还是某个组件的体积太大，针对性的作出处理

## 高版本的webpack

![图片](/img/webpack/w23.png)

## 多进程/多实例构建

使用happypack实现多进程：安装`npm i happypack -D`

在webpack.prod.js中引入并使用：

```javascript
// webpack.prod.js
const HappyPack = require('happypack');

const webpackConfig = smp.wrap({
  // ...
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        // use: ['babel-loader', 'eslint-loader']
        use: ['happypack/loader']
      },
      // ...
    ]
  },
  plugins: [
    // ...
    new HappyPack({
      loaders: ['babel-loader']
    })
  ],
  // ...
});

module.exports = webpackConfig;

```

先看一下没有使用happypack前的构建速度,3726ms

![图片](/img/webpack/w24.png)

使用了happypack之后的构建速度：

![图片](/img/webpack/w24.png)

可以看到happy默认启用了三个线程构建，构建速度2605ms，速度的提升还是很明显的

再看一下使用webpack提供的thread-loader实现多进程的构建，安装： `npm i thread-loader -D`,

在webpack.prod.js中引入并使用：

```javascript
// webpack.prod.js
const HappyPack = require('happypack');

const webpackConfig = smp.wrap({
  // ...
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: ['babel-loader', {loader: 'thread-loader', options: {workers: 3}}]
        // use: ['happypack/loader']
      },
      // ...
    ]
  },
  plugins: [
    // ...
    // new HappyPack({
    //   loaders: ['babel-loader']
    // })
  ],
  // ...
});

module.exports = webpackConfig;

```

运行npm run build，在终端查看构建速度，2122ms

![图片](/img/webpack/26.png)

## 多进程并行压缩

在webpack推荐使用TerserPlugin，安装：`npm install terser-webpack-plugin --save-dev`

在webpack.prod.js中引入并使用：

```javascript
// webpack.prod.js
const TerserPlugin = require('terser-webpack-plugin');

const webpackConfig = smp.wrap({
  // ...
  optimization: {
    // 
    minimizer: [new TerserPlugin({
      parallel: true
    })]
  },
  // ...
});

module.exports = webpackConfig;

```

添加并行压缩前后构建时间对比：

![图片](/img/webpack/27.png)

![图片](/img/webpack/28.png)

## 分包-预编译资源模块

前面使用的HtmlWebpackExternalsPlugin，可以把一些静态资源通过cdn的方式引入，以减少构建包的体积，那么如果这种静态引入的包越多，就需要每个配置，然后在模版页面每个引入

可以使用DLLPlugin可以对多个组件或者框架哭进行提取

根目录新建webpack.dll.js配置文件，配置dll预编译的配置:先给目前使用的react相关的进行分包预编译配置，

```javascript
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {
    library: [
      'react', 'react-dom'
    ]
  },
  output: {
    filename: '[name]_[chunkhash].dll.js',
    path: path.join(__dirname, 'build/library'),
    library: '[name]'
  }, 
  plugins: [
    new webpack.DllPlugin({
      name: '[name]_[hash]',
      path: path.join(__dirname, 'build/library/[name].json'),
    })
  ]
}
```

在根目录的的package.json增加dll命令脚本：

```json
"scripts": {
    "dll": "webpack --config webpack.dll.js"
  },
```

运行npm run dll，会在根目录生成一个build文件夹，里面是预编译的react代码，同时会生成一个json文件，包含预编译代码的信息，

然后在webpack.prod.js中引用预编译的代码：

```javascript
const webpack = require('webpack');

const webpackConfig = smp.wrap({
  plugins: [
    new webpack.DllReferencePlugin({
      manifest: './build/library/library.json'
    })
  ],
});

module.exports = webpackConfig;

```

## 利用缓存提升二次构建的速度

缓存对二次构建速度有用，先看一下没有开启缓存之前的构建速度：

![图片](/img/webpack/29.png)

目前的项目比较小，构建一下大概2秒多一点

### babel-loader开启转换缓存

babel转换js的语法缓存，在缓存之后下一次再构建的时候就可以直接使用缓存好的转换结果，从而提升转换的速度，开启babel-loader缓存的方法很简单，在webpack.prod.js文件中给babel-loader加一个开启缓存的参数

```javascript
// webpack.prod.js
const webpackConfig = smp.wrap({
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: ['babel-loader?cacheDirectory=true', {loader: 'thread-loader', options: {workers: 3}}]
        // use: ['happypack/loader']
      },
    ]
  },
});

module.exports = webpackConfig;

```

加好缓存之后，运行一次npm run build，这次build会生成缓存，在node_modules问价夹下生成一个.cache文件夹，里面存放的是缓存资源

![图片](/img/webpack/30.png)

然后，第二次执行npm run build，因为使用了缓存，速度的提升还是有的，只不过因为项目太小了，不是很明显

![图片](/img/webpack/31.png)

### terser开启压缩缓存

开启压缩缓存提升压缩速度，也是很简单，在webpack.prod.js中开启terser的缓存参数就好

```javascript
// webpack.prod.js
const webpackConfig = smp.wrap({
  optimization: {
    minimizer: [new TerserPlugin({
      parallel: true,
      cache: true
    })]
});

module.exports = webpackConfig;
```

开启压缩缓存之后，第一次运行npm run build:

![图片](/img/webpack/32.png)

再一次运行npm run build:

![图片](/img/webpack/33.png)

### 针对模块的缓存

HardSourceWebpackPlugin为模块提供中间缓存，使用方法：

安装：`npm install --save-dev hard-source-webpack-plugin`

```javascript
// webpack.prod.js
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin');

const webpackConfig = smp.wrap({
  plugins: [
    new HardSourceWebpackPlugin()
  ],
});

module.exports = webpackConfig;

```

配置好之后，第一次运行npm run build，可以看到这个插件在写入缓存

![图片](/img/webpack/34.png)

第二次运行之后，可以看到插件提示使用了1M的缓存，速度也是有了明显的提升

![图片](/img/webpack/35.png)

## 缩小构建体积

webpack自带的配置也可以用来缩小构建体积，例如配置模块查找字段resolve,include,exclude等字段

```javascript
// webpack.prod.js
const webpackConfig = smp.wrap({
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        include: path.resolve('src'),
        use: ['babel-loader?cacheDirectory=true', {loader: 'thread-loader', options: {workers: 3}}]
        // use: ['happypack/loader']
      },
    ]
  },
  resolve: {
    alias: {
      'react': path.resolve(__dirname, './node_modules/react/umd/react.production.min.js'),
      'react-dom': path.resolve(__dirname, './node_modules/react-dom/umd/react-dom.production.min.js')
    },
    extensions: ['.js'],
    mainFields: ['main'],
  }
});

module.exports = webpackConfig;

```

## tree shaking清除无用的代码

purgecss-webpack-plugin擦除无用的css，安装： `npm i purgecss-webpack-plugin -D`

使用：

```javascript
// webpack.prod.js
const purgecssWebpackplugin = require('purgecss-webpack-plugin');

const PATHS = {
  src: path.join(__dirname, 'src')
}

const webpackConfig = smp.wrap({
  module: {
    rules: [
    new purgecssWebpackplugin({
      paths: glob.sync(`${PATHS.src}/**/*`,  { nodir: true }),
    })
  ],
});

module.exports = webpackConfig;

```

写一行没有用到的css，运行npm run build，样式并没有被打包到bundle文件中去

## 图片压缩

安装loader: `npm install image-webpack-loader --save-dev`

```javascript
// webpack.prod.js
const webpackConfig = smp.wrap({
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif|jpeg)$/,
        use: [{
          loader: 'file-loader',
          options: {
            name: '[name]_[hash:8].[ext]'
          }
        },{
          loader: 'image-webpack-loader',
          options: {
            mozjpeg: {
              progressive: true,
              quality: 65
            },
            // optipng.enabled: false will disable optipng
            optipng: {
              enabled: false,
            },
            pngquant: {
              quality: [0.65, 0.90],
              speed: 4
            },
            gifsicle: {
              interlaced: false,
            },
            // the webp option will enable WEBP
            webp: {
              quality: 75
            }
          }
        }]
      },
    ]
  },
});

module.exports = webpackConfig;

```

## 动态polyfill

polyfill可以垫平不同浏览器之间的差异，但是有些浏览器版本较高，对于es6的语法支持很好，依然还是会使用polyfill，其实这些高版本的浏览器都是不需要的，所有会造成浪费，动态polyfill会判断当前浏览器的内核以及版本，动态的返回这个浏览器所需要的代码，从而提升速度