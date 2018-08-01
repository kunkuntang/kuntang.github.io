---
layout: '[post]'
title: 从零到一搭建Vue项目工程（Vue全家桶、Vue测试、持续集成）
date: 2018-07-08 12:49:49
tags:
---

## 开篇前记
入门前端有一段时间了，从大学就开始学习前端，到现在刚好毕业就整整4年，其中学习了不少东西，也新出不了少东西，前段的发展总是很快的，一不小心之前所学的技术就开始落后了。以前刚开始学的时候还在学html,css,js三件套，当然还会有JQuery，现在有些人入门都开始直接学Vue框架了。踩过很多坑。但是坑还要一个一个地爬出来，未知的东西还是要一个一个地探索，所以才有了冲动来写这篇文章，第一是为了总结我之前学过的知识和经验，比如Vue全家桶，wepback构建Vue环境等。其次是在我没有实践过的领域进行探索的同时记录下来拿，以后有需要的时候再看回这篇文章。

## 摘要
我把这篇文章的内容拆成几个版块，作为一个连载的文章。首先列一下主要章节内容：

- 搭建一个运行Vue的webpack环境
- 加入Vue全家桶
- 编写项目工程代码
- 总结Vue测试
- 关于项目的持续集成
- 项目总结

因为在编写项目的时候总会附上对应的测试代码，但这样有时候需要看关于Vue测试的内容时会过于零散，所以在写完项目之后专门给了一个小节来写关于Vue的测试内容。当然如果你是从头往下看的话可以跳过这一小节。最后项目 弄好了按照国际惯例总是要总结一下经验的。

## 搭建运行Vue的webpack环境

### 工程目录

```
|____.gitignore
|____index.html
|____LICENSE
|____package-lock.json
|____package.json
|____README.md
|____src
| |____App.vue
| |____assets
| |____components
| |____main.js
| |____view
|____static
|____test
|____webpack.config.js
```

### webpack配置

这是我摸索了几个钟弄出来的配置（不得不说webpack的配置是真的麻烦），以前都是懒所以直接使用vue-cli初始化出来的项目，webpack配置什么的都已经帮我配好了，这次自己动刀搭了一下才发现这么费劲，期间不断地报错，再加上由于用了4.0版本，也出现了一些坑，这里一并也记录一下，以后再遇到的时候好回头翻翻。

 首先是最基础的webpack框架，不清楚的可以去到[webpack官网](https://www.webpackjs.com/configuration/#%E9%80%89%E9%A1%B9)上找（现在都已经有翻译很好的中文网啦，哪像以前那么苦还要去啃英文 (T^T) ）。当然我们也只会用到其中的几个属性。

```
const path = require('path')
module.exports = {
  entry: '',
  output: {},
  module: {
  	rules: []
  },
  plugins: []
}
```

OK,这就完成了webpack最基本的框架，接下来我们要分别搭建不同方式启动webpack的方案。（其实这两者在实现原理上本没有什么区别，而对于我们使用者来说区别最大的大概就是工程目录的划分和关于wepback-dev-server的配置）

### 基于webpack-dev-server插件来启动webpack

基于`webpack-dev-server`这种方式启动webpack，我们首先把之前的webpack的配置文件命名为`webpack.config.js`并放在项目的根目录下。然后在`package.json`里的`script`对象添加一个`dev`属性，并在后面添加脚本`webpack-dev-server --inline --hot`再保存。例如：

```
// package.json

{
  ...
  "scripts": {
    "dev": "webpack-dev-server --inline --hot",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  ...
}
```

然后我们在webpack.config.js文件里添加devServer配置：

```
// webpack.config.js

const path = require('path')

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    publicPath: '/dist/',
    filename: 'build.js'
  },
  module: {
    rules: []
  },
  plugins: [],
  devServer: {
    // contentBase: path.join(__dirname, "dist"),
    // 确保 publicPath 总是以斜杠(/)开头和结尾。
    // 默认会取output.path的值，设置之后会覆盖使用devServer.publicPath的值
    // publicPath: "/",
    historyApiFallback: true,
    compress: true,
    proxy: { // proxy URLs to backend development server
      // '/api': 'http://localhost:3000',
      // '/api': {
      //   target: 'http://localhost:3000',
      //   pathRewrite: {'^/api' : ''}
      // }
    },
    hot: true,
    inline: true
  },
  devtool: '#eval-source-map'
}
```

这样就可以把webpack-dev-server启动起来啦~~

### 基于node + express 方式来启动webpack

### 基本可以运行Vue项目的webpack配置

完成了上面的步骤之后，接下来就是配置vue相关的东西了，先回想在使用vue开发的时候，我们会新建一个vue后缀的文件，所以我们就需要一个loader来解析和读取.vue文件的内容。先安装：

```
npm install --save-dev vue-loader vue-template-compiler
```

然后我们再在wepback配置中的module.rules中使用：

```
// webpack.config.js/webpack.base.conf.js

module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {}
      }
    ]
  },
  ...
```

至于这里为什么要安装`vue-template-compiler`，是因为单纯使用`vue-loader`启动的时候会报错，提示在没有`vue-template-compiler`的情况下使用`vue-loader`，我在网上查了一下也有人写了[blog](https://blog.csdn.net/cominglately/article/details/80555210)说明了这一情况，意思大概是升级到webpack 4.0之后会出现的问题。为了解决这个问题我们要在webpack配置上使用一下。

```
// webpack.config.js/webpack.base.conf.js

const path = require('path')
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {}
      }
    ]
  },
  plugins: [
    new VueLoaderPlugin()
  ],
  ...
```
现在就可以读vue文件了，接下来就是对vue文件里的css、非标准的js、图片分别配置，也让webpack可以有能力解析。

解析css（css-loader是解析css的，style-loader是把css以<style></style>内嵌的方式注入到html中）:
```
npm install --save-dev style-loader css-loader
```

解析非标准的js（[使用babel](https://www.babeljs.cn/docs/setup/#installation)）:
```
npm install --save-dev babel-loader babel-preset-env
```

解析img:
```
npm install --save-dev file-loader
```

然后再分别写到rules里面，就完成了。整个配置如下：

```
// webpack.config.js/webpack.base.conf.js

const path = require('path')
const VueLoaderPlugin = require('vue-loader/lib/plugin');

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    publicPath: '/dist/',
    filename: 'build.js'
  },
  module: {
    rules: [{
        test: /\.css$/,
        use: [
          'vue-style-loader',
          'css-loader'
        ]
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {}
      },
      {
        test: /\.(png|jpg|gif|svg)$/,
        loader: 'file-loader',
        options: {
          name: '[name].[ext]?[hash]'
        }
      }
    ]
  },
  plugins: [
    new VueLoaderPlugin()
  ],
  devServer: {
    // contentBase: path.join(__dirname, "dist"),
    // 确保 publicPath 总是以斜杠(/)开头和结尾。
    // 默认会取output.path的值，设置之后会覆盖使用devServer.publicPath的值
    // publicPath: "/",
    historyApiFallback: true,
    compress: true,
    proxy: { // proxy URLs to backend development server
      // '/api': 'http://localhost:3000',
      // '/api': {
      //   target: 'http://localhost:3000',
      //   pathRewrite: {'^/api' : ''}
      // }
    },
    hot: true,
    inline: true
  },
  devtool: '#eval-source-map'
}
```

### 项目优化

按照刚刚的步骤进行操作的话，会发现当项目运行起来的时候webpack会报如下错误，现在慢慢地按照提示来对项目进行优化。

[![qqEhK.png](https://s1.ax2x.com/2018/07/19/qqEhK.png)](https://simimg.com/i/qqEhK)

#### WARNING in configuration

```
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```
翻译成中文的意思大概是：`mode`这个选项没有被设置，webpack将会默认设置为`production`。你可以把`mode`选项设置为`development`或者`production`，以让webpack以指定的环境启动。你也可以设置为`none`以禁用任何的默认行为。[了解更多：https://webpack.js.org/concepts/mode/](https://webpack.js.org/concepts/mode/)。

这个问题主要是告诉我们要让webpack知道现是要按什么环境去构建应用，通常我们会通过一个配置文件来记录所有会用到的环境，然后在调用的时候通过传入参数的形式来指定使用哪一个环境。

我们新建一个目录`config`，在目录里面再分别建立不同环境的配置，如：

```
├── config
│   ├── dev.js
│   ├── index.js
│   ├── prod.js
│   └── test.js
```

我们分别区别了开发环境、生产环境、和测试环境，然后再通过一个index.js来判断当前使用哪一个环境：

```
// index.js
const prod = require('./prod')
const dev = require('./dev')
const test = require('./test')

let env = prod;

switch (process.env.NODE_ENV) {
  case 'development':
    env = dev;
    break;
    // case 'test':
    //   env = test;
    //   break;
  default:
    env = prod;
}

module.exports = env
```

这里的`process.env`是node.js的一个环境变量对象，挂载到这个对象上的属性可以在node的全局运行环境中访问到，而`NODE_ENV`是node社区约定作为当前运行环境的属性（当然你也可以用其它属性代替）。

为了我们在启动的时候可以为node指定当前运行环境，我们新加入一个依赖包叫[cross-env](https://www.npmjs.com/package/cross-env)，然后修改`package.json`文件里的运行脚本命令，添加`cross-env NODE_ENV=[prototype]`在命令前面即可，如：

```
// package.json

...
"scripts": {
  "dev": "cross-env NODE_ENV=development webpack-dev-server --inline --hot",
  "test": "echo \"Error: no test specified\" && exit 1"
},
...
```

#### WARNING in asset size limit

```
The following asset(s) exceed the recommended size limit (244 KiB).
This can impact web performance.
Assets:
  build.js (2.13 MiB)
```

#### WARNING in entrypoint size limit

```
The following entrypoint(s) combined asset size exceeds the recommended limit (244 KiB). This can impact web performance.
Entrypoints:
  main (2.13 MiB)
      build.js
      0.9256098444c5c2f2e51c.hot-update.js
```

#### WARNING in webpack performance recommendations

```
You can limit the size of your bundles by using import() or require.ensure to lazy load some parts of your application.
For more info visit https://webpack.js.org/guides/code-splitting/
```

