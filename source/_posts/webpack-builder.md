---
title: 编写可维护的webpack构建配置，进阶学习
date: 2020-05-27 17:26:16
tags: webpack
---

如何把webpack的配置编写的更加通用，把webpack的配置编写成一个构建包，在每个项目里都能快速使用

现在的webpack配置分为webpack.prod.js，  webpack.dev.js，  webpack.ssr.js，分别对应三个不同的环境场景，三个配置文件里面的配置有些都是相同的例如es6代码的转换，react的使用、处理less样式文件等，不得不每次复制到每个配置文件代码里面去，而不同的环境场景的配置又具有差异，解决思路就是新建一个webpack.base.js文件存放共用的配置，使用webpack-merge组件合并配置

在根目录新建build-webpack文件夹，cd build-webpack，进入目录，npm init -y初始化package.json文件

build-webpack文件夹下新建lib目录，存放webpack配置文件，在lib文件下新建webpack.base.js，webpack.prod.js，  webpack.dev.js，  webpack.ssr.js文件：

```javascript
// webpack.base.js
const {CleanWebpackPlugin} = require('clean-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const FriendlyErrorsWebpackPlugin = require('friendly-errors-webpack-plugin');

// 动态设置entry 和 htmlWebpackPlugin
const glob = require('glob');
setMPA = () => {
  const entry = {};
  const htmlWebpackPlugin = [];

  const entryFiles = glob.sync(path.join(__dirname, './src/*/index.jsx'));
  Object.keys(entryFiles).map((index) => {
    const entryFile = entryFiles[index];
    const match = entryFile.match(/src\/(.*)\/index\.jsx/);
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

const {entry, htmlWebpackPlugin} = setMPA();

module.exports = {
  entry: entry,
  output: {
    path: path.resolve(projectRoot, 'dist'), //必须是绝对路径
    filename: '[name]_[chunkhash:8].js',
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: ['babel-loader']
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
    new MiniCssExtractPlugin({
      filename: '[name]_[contenthash:8].css'
    }),
    ...htmlWebpackPlugin,
    new FriendlyErrorsWebpackPlugin(),
    function() {
      this.hooks.done.tab('done', (stats) => {
        if(stats.compilation.errors && stats.compilation.errors.length && process.argv.indexOf('--watch') == -1) {
          console.log('build error');
          process.exit(1);
        }
      })
    }
  ],
  stats: 'errors-only'
}
```

安装 :`npm i webpack-merge -D`

webpack.dev.js包括热更新和source-map，使用webpack-merge合并配置

```javascript
// webpack.dev.js
const merge = require('webpack-merge');
const baseConfig = require('./webpack.base');
const webpack = require('webpack');

// 热更新，source-map

const devConfig = {
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ],
  devServer: {
    contentBase: './dist', // 服务基础目录
    hot: true, // 开启热更新
    stats: 'errors-only'
  },
  devtool: 'source-map',
  mode: 'development',
}

module.exports = merge(baseConfig, devConfig);
```

webpack.prod.jsb包括代码压缩，文件指纹,公共资源的提取：

```javascript
// webpack.prod.js
const merge = require('webpack-merge');
const baseConfig = require('./webpack.base');
const OpimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');

// 代码压缩，文件指纹,公共资源的提取

const prodConfig = {
  mode: 'production',
  plugins: [
    new OpimizeCssAssetsPlugin({
      AccetNameRegExp: /\.css$/g,
      cssProcessor: require('cssnano')
    }),
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
  optimization: {
    splitChunks: {
      minSize: 0,
      cacheGroups: {
        commons: {
          name: 'commons',
          chunks: 'all',
          minChunks: 2
        }
      }
    }
  },
}

module.exports = merge(baseConfig, prodConfig);
```

webpack.ssr.js主要是不解析css文件

```javascript
// webpack.ssr.js
const merge = require('webpack-merge');
const baseConfig = require('./webpack.base');
const OpimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const HtmlWebpackExternalsPlugin = require('html-webpack-externals-plugin');

// 代码压缩，文件指纹,公共资源的提取

const prodConfig = {
  mode: 'production',
  module: {
    rules: [
      {
        test: /\.css$/,
        use: 'ignore-loader'
      },
      {
        test: /\.less$/,
        use: 'ignore-loader'
      },
    ]
  },
  plugins: [
    new OpimizeCssAssetsPlugin({
      AccetNameRegExp: /\.css$/g,
      cssProcessor: require('cssnano')
    }),
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
  optimization: {
    splitChunks: {
      minSize: 0,
      cacheGroups: {
        commons: {
          name: 'commons',
          chunks: 'all',
          minChunks: 2
        }
      }
    }
  },
}

module.exports = merge(baseConfig, prodConfig);
```

新建.gitignore文件

```json
/node_modules
/logs
```

## 使用eslint规范构建脚本

eslint可以帮助在构建之前检查代码的规范，而不是等到运行的时候在报错

在build-webpack目录安装eslint以及eslint所需要的依赖：`npm i eslint eslint-config-airbnb-base babel-eslint -D`

新建.eslintrc.js文件

```javascript
// build-webpack/.eslintrc.
module.exports = {
  "parser": "babel-eslint",
  "extends": "airbnb-base",
  "env": {
    "browser": true,
    "node": true
  }
}
```

在package.json增加eslint script

```json
// build-webpack/package.json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "eslint": "eslint ./lib --fix"
  },
```

运行npm run eslint可以帮助在构建之前检查规范，

在命令行运行npm run eslint

![图片](/img/webpack/w17.png)

这边的报错是因为我的webpack配置中使用的插件没有安装

根据报错一次安装插件到dependencies，根据提示解决报错

## 冒烟测试

冒烟测试确保构建没有大问题，检测一些基本的功能可用

例如构建是否成功，每次构建完成是否正确的打包静态资源

在test新建template目录，作为模版目录，也就是要使用构建脚本构建的项目，可以把之前的项目复制一份进来，删除不需要的文件：

![图片](/img/webpack/w18.png)

在build-webpack目录新建test目录，test新建smoke目录，smoke新建index.js文件用来编写测试脚本：

```javascript
// test/smoke/index.js
const path = require('path');
const webpack = require('webpack');
const rimraf = require('rimraf');

// 进入tamplate，__dirname代表运行时候的目录
process.chdir(path.join(__dirname, 'template'));

// 每次构建之前，删除dist目录
rimraf('./dist', () => {
  // 引入配置
  const prodConfig = require('../../lib/webpack.prod.js');
  // 通过webpack对template目录的代码运行prodConfig这个配置
  webpack(prodConfig, (err, stats) => {
    if(err) {
      console.log(err);
      process.exit(2);
    }
    console.log(stats.toString({
      color: true,
      modules: false,
      children: false
    }))
  })
})
```

在build-webpack目录安装： `npm i rimraf -S`

修改webpack.base.js文件，增加projectRoot，修改执行的目录为tamplate目录

```javascript
// webpack.base.js
const projectRoot = process.cwd(); // 当前目录
const setMPA = () => {
  const entryFiles = glob.sync(path.join(projectRoot, './src/*/index.jsx'));
  Object.keys(entryFiles).map((index) => {
    // ...
    return htmlWebpackPlugin.push(
      new HtmlWebpackPlugin({
        template: path.join(projectRoot, `src/${pageName}/index.html`),
        // ...
      }),
    );
  });

  return {
    entry, htmlWebpackPlugin,
  };
};
```

在终端运行：`node test/smoke/index.js`

### 检查dist目录是否有构建出来的文件

使用mocha编写单元测试的测试用例，安装： `npm i mocha -D`

`npm i glob-all -D`

新建smoke/html-test.js：

```javascript
const glob = require('glob-all');

describe('checking generated html files', () => {
  it('should genarate html files', (done) => {
    const files = glob.sync(['./dist/index.html', './dist/search.html']);
    if(files.length > 0) done();
    else throw new Error('no html files genarated');
  })
})
```

新建smoke/css-js-test.js:

```javascript
const glob = require('glob-all');

describe('checking generated css js files', () => {
  it('should genarate css js files', (done) => {
    const files = glob.sync(['./dist/index_*.js', './dist/search_*.js', './dist/index_*.css', './dist/search_*.css']);
    if(files.length > 0) done();
    else throw new Error('no css js files genarated');
  })
})
```

修改smoke/index.js

```javascript
// /...
const Mocha = require('mocha');

const mocha = new Mocha({
  timeout: '10000ms'
})
// ...
// 每次构建之前，删除dist目录
rimraf('./dist', () => {
  webpack(prodConfig, (err, stats) => {
    // ...
    console.log('webpack build success, begin run test');
    mocha.addFile(path.join(__dirname, 'html-test.js'));
    mocha.addFile(path.join(__dirname, 'css-js-test.js'));
    mocha.run();
  })
})
```

在终端运行：`node test/smoke/index.js`，冒烟测试成功之后会在终端打印

![图片](/img/webpack/w19.png)

## 单元测试

使用抹茶和断言库进行单元测试

安装断言库：`npm i assert -D`

在test目录新建index文件，作为单元测试的入口，在test目录新建unit目录，这个目录放单元测试的代码

测试webpack.base配置，test/unit/webpack-base-test.js:

```javascript
// test/unit/webpack-base-test.js
describe('webpack.base.js test case', () => {
  const baseConfig = require('../../lib/webpack.base');
  const assert = require('assert');
  // console.log(baseConfig)
  // 测试entry字段
  it('entry', () => {
    assert.equal(baseConfig.entry.index, '/Users/lizhaoxin/mypro/webpack-first/build-webpack/test/smoke/template/src/index/index.jsx');
    assert.equal(baseConfig.entry.search, '/Users/lizhaoxin/mypro/webpack-first/build-webpack/test/smoke/template/src/search/index.jsx')
  })
})
```

test/index：

```javascript
const path = require('path');

// 进入tamplate, __dirname代表运行时候的目录
process.chdir(path.join(__dirname, 'smoke/template'));
// 引入测试代码
describe('builder-webpack test case', () => {
  require('./unit/webpack-base-test');
})
```

修改build-webpack/package.json，增加test运行脚本

```json
"scripts": {
    "test": "./node_modules/.bin/_mocha",
  },
```

运行npm run test:测试用例成功了，会在前面打勾

![图片](/img/webpack/w20.png)

测试覆盖率， istanbul是一个单元测试代码覆盖率检查工具，可以很直观地告诉我们，单元测试对代码的控制程度。，安装 npm i istanbul -D, 修改build-webpack/package.json

```json
"scripts": {
    "test": "istanbul cover ./node_modules/.bin/_mocha",
  },
```

运行npm run build,可以在终端看到，测试用例的覆盖情况,例如单元测试覆盖了多少函数，多少代码行数，多少分支：

![图片](/img/webpack/w21.png)

## 持续集成