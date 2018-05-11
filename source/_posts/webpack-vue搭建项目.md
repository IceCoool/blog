---
title: webpack+vue搭建项目
date: 2017-07-13 17:54:14
tags:
categories:
---

webpack近两年太火了，出去找工作要是不懂点webpack都不好意思说是做前端的（哭~），而我。。。（哭个不停~）。之前虽然有一直在用，但都不是我配置的，而我平时写demo的时候，嗯。。。vue-cli 搞定！，所以为了加深自己对webpack的理解，我决定一步步配置它，并且记载下来，这也变相的练习了两遍，挺好。

<!-- more -->

首先我们要明白自己想要搭建一个什么样的项目，实现什么样的功能，会用到什么工具等，弄清楚了就要开始做准备了。这个项目算是一个比较大众的项目，本地开发，打包（分离公共文件，将css从js中提取出来，压缩，等）。

这里先介绍一下依赖包的作用
```
webpack-dev-server => 本地起服务
vue-loader vue-template-compiler => 解析.vue文件
babel-core babel-loader babel-preset-es2015 babel-preset-stage-0 => 编译es6 
html-webpack-plugin => 将js，css自动引入html
extract-text-webpack-plugin => 分离css
node-sass css-loader  style-lodader sass-lodader => 编译sass css 
url-lodader file-loader => 处理图片路径问题
webpack-merge => 合并公共部分代码
```

### 搭建基本环境

新建项目目录结构如下
![](/images/catalog.png)

* 安装依赖
```
npm install vue --save
npm install vue-loader vue-template-compiler --save-dev
npm install webpack@3.6.0 webpack-dev-server@2.4.5 --save-dev
npm install babel-core babel-loader babel-preset-es2015 babel-preset-stage-0 --save-dev
```

* 安装好之后配置webpack.config.js
```javascript
const path = require('path')
const webpack = require('webpack')
module.exports = { 
  // 入口文件
  entry: {
    main: path.resolve(__dirname, 'src/index.js')
  },
  output: {
    path: path.resolve(__dirname, './build'),
    publicPath: '/build',
    filename: 'main.js'
  },
  devServer: {
    port: 3333, // 端口
    inline: true, // 热更新
    historyApiFallback: true
  },
  module: { // 这里主要是做解析文件
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        // 排除依赖的文件
        exclude: /node_modules/
      }
    ]
  }
}
```

* 编写模板文件index.html，构建的时候js会被打包进来
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div id="app"></div>
</body>
</html>
<script src="/build/build.js"></script>
```
   这里说一下 index.html 最后引入的就是构建好的js，当然真实项目中肯定不需要自己手动引入，而是借助工具自动构建进去的，后面会说到。

* 编写App.vue
```html
<template>
  <div>
      hello webpack
  </div>
</template>
<script>
    export default {
        data () {
            
        }
    }
</script>
```

* 配置入口文件index.js
```javascript
import vue from 'vue'
import App from './App.vue'

new Vue({
    el: '#app',
    render: h => h(App)
})
```

    到这里基本的环境就搭建好了，我们来试试有没有成功，在package.json里配置如下代码
    ```
    "scripts": {
       "dev": "webpack-dev-server --config webpack.config.js"
     },
    ```
    执行 `npm run dev`，浏览器打开localhost:3333 看到hello webpack 说明我们已经向成功迈出了关键的一步。

### 细节优化

* html-webpack-plugin
```
npm install html-webpack-plugin --save-dev
```
    之前模板index.html引入的js是我们自己手动添加的，但是真实项目中为了避免缓存问题的出现，生成的js往往会带有哈希值，这时我们在想手动引入的话是不现实的，所有这里要用到html-webpack-plugin，它可以将生成的js，css自动插入到模板文件。

    配置webpack.config.js

    ```javascript
    const htmlWebpackPlugin = require('html-webpack-plugin')

    module.exports = {
        entry: {
            ...
        },
        output: {
            ...
        },
        devServer: {
            ...
        },
        module: {
            rules: [
                ...
            ]
        },
        plugins: [
           new htmlWebpackPlugin({
               template: './index.html'
           })
        ]
    }
    ```

    这里修改了一个地方
    ```javascript
    output: {
        path: path.resolve(__dirname, '/'),
        publicPath: '/',
        filename: '[name].[hash:8].js'
    },
    ```
    我们将生成的js用后面的8位hash值命名，publicPath改为根目录，因为我们想在根目录下预览。

* 处理css sass img
```
npm install extract-text-webpack-plugin@2.1.2 --save-dev 
```

    extract-text-webpack-plugin 这个插件是用来提取css的，因为之前打包将css构建到js里面去了。

    ```
    npm install node-sass css-loader style-loader sass-loader url-loader file-loader --save-dev
    ```

    配置webpack.config.js

    ```javascript
    module.exports = {
        module: {
            rules: [
                {
                    test: /\.vue$/,
                    loader: 'vue-loader'
                },
                {
                    test: /\.js$/,
                    loader: 'babel-loader',
                    // 排除依赖的文件
                    exclude: /node_modules/
                },
                {
                    test: /\.css/,
                    use: extractTextPlugin.extract({
                        use: 'css-loader'
                    })
                },
                {
                    test: /\.scss$/,
                    use: extractTextPlugin.extract({
                        fallback: 'style-loader',
                        use: ['css-loader','sass-loader']
                    })
                },
                {
                    test: /\.(png|woff|woff2|eot|ttf|svg|jpg)$/,
                    loader: 'url-loader',
                    options: {
                        limit: 8192
                    }
                }
            ]
        }
        plugins: [
           new htmlWebpackPlugin({
               template: './index.html'
           }),
           new extractTextPlugin('style.[contenthash:8].css')
        ]
    }
    ```

* 提取公共文件
一个项目里肯定是要引用到一些公共的js的，比如vue,webpack在打包的时候是不会主动区分这些公共的js的，所以需要我们自己把它们区分开来。

    webpack内有内置的插件，将配置写在plugins里面
    ```javascript
    plugins: [
      new htmlWebpackPlugin({
          template: './index.html'
      }),
      new extractTextPlugin('style.[contenthash:8].css'),
      // 提取公告文件
      new webpack.optimize.CommonsChunkPlugin({
        name: 'vendors'
      })
    ]
    ```

    在plugins里，新创建了一个构造函数，new webpack.optimize.CommonsChunkPlugin(),它的参数里定义了一个name:”vendors”。所以我们在entry对象加上vendors:[]
    ```
    entry: {
        main: path.resolve(__dirname, 'src/index.js'),
        vendors: ['vue']
    },
    ```


    为了查看优化有没有成功，我在项目里写了一些样式之类的，然后运行，如下结果，ok，这样一个基于vue的开发环境就有了。
    ![](/images/vendors.png)
    ![](/images/vendors2.png)



### 开发环境，生产环境分离

一个完整的项目肯定要不止有一个环境，要考虑很多东西，开发过程中我们要为了调试方便做配置，自然就不能压缩代码，而生产环境要压缩代码，所以我们接下来就要针对开发和生产环境分别做配置。两者有很大一部分配置是相同的，所以我们要把公共的部分提取出不来。

* 公共部分 webpack.common.js
```javascript
const path = require('path')
const webpack = require('webpack')
const htmlWebpackPlugin = require('html-webpack-plugin')
const extractTextPlugin = require('extract-text-webpack-plugin')
let commonConfig = {
  // 入口文件
  entry: {
    main: path.resolve(__dirname, 'src/index.js'),
    vendors: ['vue']
  },
  devServer: {
    port: 8888, // 端口
    inline: true, // 热更新
    historyApiFallback: true
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        // 排除依赖的文件
        exclude: /node_modules/
      },
      {
          test: /\.css/,
          use: extractTextPlugin.extract({
              use: 'css-loader'
          })
      },
      {
          test: /\.scss$/,
          use: extractTextPlugin.extract({
              fallback: 'style-loader',
              use: ['css-loader','sass-loader']
          })
      },
      {
          test: /\.(png|woff|woff2|eot|ttf|svg|jpg)$/,
          loader: 'url-loader',
          options: {
              limit: 8192,
          }
      }
    ]
  },
  resolve: {
    alias: {
      styles: path.resolve(__dirname, 'src/style/'),
      images: path.resolve(__dirname,'src/images')
    },
    extensions: ['.js']
  },
  plugins: [
      new htmlWebpackPlugin({
          template: './index.html'
      }),
      new extractTextPlugin('style.[contenthash:8].css'),
      new webpack.optimize.CommonsChunkPlugin({
        name: 'vendors'
      })
  ]
}

module.exports = commonConfig
```
 这里有个resolve属性，之前没有说到，它其中的一个功能就是定义变量。

   ```javascript
   //配置文件webpack.common.js
    resolve: {
        alias: {
            styles: path.resolve(__dirname, 'src/style/'),
            image: path.resolve(__dirname, 'src/images/'),
        },
        extensions: ['.js']
    }

    //main.js
    // 引用scr/style下的scss文件
    import 'style/main.scss';

    //App.vue
    //引用src/images/下的图片
    <img src="~image/picture.png" alt="">
   ```

* 开发环境配置 webpack.dev.js
首先下载一个插件 用来合并代码
```
npm i webpack-merge --save-dev
```

    ```javascript
    const merge = require('webpack-merge')//用于合并文件
    const path = require('path')
    const webpack = require('webpack')
    const common = require('./webpack.common')

    module.exports = merge(common, {
        output: {
            path: path.resolve(__dirname, '/'),
            publicPath: '/',
            filename: '[name].[hash:8].js'
            },
        devtool: '#source-map',
        devServer: {
            port: 9999,
            hot: true,
            // inline: true,
            // 文件更新，页面自动刷新
            host: '0.0.0.0',
            // 允许本地ip访问
            historyApiFallback: true,
            stats: 'errors-only'// 仅显示错误日志
        },
        plugins: [
            new webpack.HotModuleReplacementPlugin()
        ]
    })
    ```
* 生产环境配置 webpack.prod.js
```javascript
const merge = require('webpack-merge')
const path = require('path')
const webpack = require('webpack')
const common = require('./webpack.common')

module.exports = merge(common, {
  output: {
    path: path.resolve(__dirname, 'build'),
    publicPath: '/',
    filename: '[name].[hash:8].js'
  },
  plugins: [
    // js 压缩
    new webpack.optimize.UglifyJsPlugin({ compress: {warnings: false}})
  ]
})
```
  这里的ouput:{}中，path属性就是我们最后打包后的路径，所以最好不要放在根目录下，所以这也是我为了和开发环境拆开的原因。
   
   这里插个话题，就是image打包。我们打包后想把文件放到一个文件夹里，所以我们在url-loader内稍作修改，添加name属性即可：

 ```javascript
 module: {
     rules: [
        {
          test: /\.(png|woff|woff2|eot|ttf|svg|jpg)$/,
          loader: 'url-loader',
          options: {
              limit: 8192,
              name: 'images/[hash:8].[name].[ext]'
          }
        }
     ]
 }
 ```
 最后在package.json中添加以下代码方便项目的启动和打包
 ```
 "scripts": {
    "dev": "webpack-dev-server --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js",
    "git": "rm -rf dist/* && webpack --config webpack.prod.js"
  },
 ```
 OK，大功告成！   











