# 你自己有搭建过 vue 的开发环境吗？

搭建一个 Vue 开发环境以及一个简单的 cli 的过程小记

## 搭建一个 Vue 开发环境

步骤大概有那么几步：

- 新建一个 git 仓库 初始化 项目
- 创建目录及下载相关基本的第三方库
- 写开发环境的 webpack.config.js
- 添加 Vuex 和 Vue-Router,及相关的代码
- 优化代码写生产环节的 webpack.js
- 给项目添加 eslint 和 prettier 控制代码质量
- 给项目添加 git 钩子函数
- 添加单元测试模块（TODO）

下面开始逐步的来搞定它：

### 创建 git 仓库并初始化 项目

在 Github 上创建一个 repo，然后 clone 到本地，然后打开文件夹，在命令行里输入`npm init`  
输入正常的信息，然后就可以开始开发啦～～

### 创建目录及下载相关基本的第三方库

在 Vue 的开发的过程中，我们一般对用到以下目录：public，src，utils，webpack 等四个文件夹，所以先创建这四个文件夹。

然后安装 webpack 及 Vue

```
npm i webpack webpack-cli webpack-dev-server -D
npm i vue -S
```

好了第二步搞定～，我们可以开始开发这个开发环境了。

### 写开发环境的 webpack.config.js

写 webpack 文件的前置步骤，在 src 文件夹里面创建一个 main.js 和 App.vue 和 index.html

```js
// main.js
import Vue from 'vue';
import App from './App.vue';

Vue.config.productionTip = false; // 阻止产生提示

new Vue({
  el: '#app',
  render: h => h(App),
});
```

```html
// App.vue
<template>
  <div id="app">
    欢迎使用Vue，详情请见 https://github.com/kaisa911
    <router-view />
  </div>
</template>

<script>
  export default {
    name: 'App',
  };
</script>

<style scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    transform: rotate(0deg);
  }
</style>
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="x-ua-compatible" content="IE=edge,Chrome=1" />
    <meta
      name="viewport"
      content="width=device-width,minimum-scale=1,maximum-scale=1,initial-scale=1,user-scalable=no"
    />
    <meta name="format-detection" content="telephone=no" />
    <link rel="icon" href="favicon.ico" />
    <title><%= title %></title>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
```

在 webpack 的文件夹下创建一个 webpack.common.js 的文件

```js
const path = require('path');
const config = {
  entry: path.join(__dirname, '../src/main.js'),
  output: {
    path: path.join(__dirname, '../dist'), // 所有的文件都输出到dist/目录下
    filename: 'js/bundle.[hash].js', // 每次保存 hash 都变化
  },
};

module.exports = config;
```

这就是初始的 webpack 的配置文件，但是我们写的是 Vue 的文件，所以要添加一些用来解析不同文件的 loader  
因为要处理.vue 的但文件组件，所以我们要添加一个 vue-loader  
要处理.js 文件，所以还需要 babel-loader  
要处理 css，所以要添加 css 的预处理和后处理的相关的 loader，我们要使用 less 作为 css 的预处理器，所以添加 css-loader，style-loader，less-loader，postcss-loader

下面要安装相关的 loader

```
npm i vue-loader -D
npm i babel-loader -D
npm i less less-loader -D
npm i css-loader style-loader postcss-loader
```

安装完成之后，我们可以在 webpack.common.js 里加入 module，来处理不同的文件

```js
...

module:{
  rules:[
    {
      // 使用vue-loader解析.vue文件
      test: /\.vue$/,
      loader: 'vue-loader',
    },
    {
      // 使用babel-loader解析js文件
      test: /\.js$/,
      exclude: /node_modules/,
      loader: 'babel-loader',
    },
    {
      test: /\.(le|c)ss$/,
      use: ['style-loader', 'css-loader', 'postcss-loader', 'less-loader'],
    },
  ]
}
...
```

安装完 postcss-loader 之后，还需要在根目录，添加一个 postcss.config.js

```js
module.exports = {
  plugins: {
    autoprefixer: {},
  },
};
```

安装完 babel-loader 之后，还需要配置一下 babel 的相关配置，在根目录添加一个 babel.config.js

所以要安装

```js
npm i @babel/core @vue/babel-preset-app -D
```

```js
module.exports = {
  presets: ['@vue/app'],
};
```

新版本的 vue-loader 还需要在 plugin 里添加 vue-loader-plugin 才能使用,  
所以要在 webpack.common.js 里添加 plugins。  
这里需要添加的 plugin 还有：html-webpack-plugin

```js
const VueLoaderPlugin = require('vue-loader/lib/plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
...
plugins: [
  new VueLoaderPlugin(), // 最新版的vue-loader需要配置插件
  new HtmlWebpackPlugin({
    filename: 'index.html',
    template: path.resolve(__dirname, '../src/index.html'),
    inject: true,
    templateParameters: {
      title: '框架demo',
    },
    favicon: path.resolve(__dirname, '../public/favicon.ico'),
  }),
],
...
```

最近本的 webpack 已经写完了，现在可以在 package.json 里写一个命令，就可以打包出你写的东西了。

```js
...
script:{
  "start": "webpack --config ./webpack/webpack.common.js",
}
...
```

打包的东西，就会出现在 dist 目录里了。

但是我们在开发环境的时候，都需要 webpack 直接启动一个服务，所以我们需要写一个开发环境的配置，在 webpack 文件夹里新建一个 webpack.dev.js，来处理 webpack-dev-server 的问题

```js
const common = require('./webpack.common');
const webpack = require('webpack');

const config = {
  ...common,
  mode: 'development',
  devtool: 'inline-source-map',
  devServer: {
    // 开发服务器
    contentBase: '../dist',
    hot: true,
    inline: true,
    host: '0.0.0.0',
    port: '6001',
    disableHostCheck: true,
    historyApiFallback: true,
    stats: 'minimal',
    compress: true,
  },

  plugins: [
    ...common.plugins,
    new webpack.HotModuleReplacementPlugin(), // 热更新插件
  ],
};

module.exports = config;
```

然后修改一下 package.json 里的命令

```js
"scripts": {
  "start": "webpack-dev-server --color --open --config ./webpack/webpack.dev.js",
},
```

这样你在命令行里运行`npm start`就可以在浏览器里看到你写的内容了

ps.我在这个地方出现了一个问题，我之前使用 switchhost 的时候，把 127.0.0.1 对应的 localhost 的代理给关了。。打开就好了

### 添加 Vuex 和 Vue-Router,及相关的代码

现在可以在项目里添加一些内容了，因为热更替可以直接看到变化的内容。

- 在 src 里添加 views，assets，components 等文件夹
- 在 views 文件夹里创建 index.vue 和 about.vue 文件
- 安装 vuex 和 vue-router，以及在 src 里创建 store.js 和 router.js

```js
npm i vuex vue-router -S
```

```js
// store.js
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {},
  mutations: {},
  actions: {},
});
```

```js
// router.js
import Vue from 'vue';
import Router from 'vue-router';

Vue.use(Router);

export default new Router({
  routes: [
    { path: '/', name: 'index', component: () => import('./views/index.vue') },
    {
      path: '/about',
      name: 'about',
      component: () => import('./views/about.vue'),
    },
  ],
});
```

修改 main.js

```js
import Vue from 'vue';
import App from './App.vue';
import store from './store';
import router from './router';

Vue.config.productionTip = false; // 阻止产生提示

new Vue({
  el: '#app',
  store,
  router,
  render: h => h(App),
});
```

```html
<template>
  <div>index</div>
</template>

<script>
  export default {
    name: 'Index',
  };
</script>
```

```html
<template>
  <div>about</div>
</template>

<script>
  export default {
    name: 'About',
  };
</script>
```

修改 App.vue

```html
<template>
  <div id="app">
    欢迎使用Vue，详情请见 https://github.com/kaisa911
    <div class="menu">
      <div @click="$router.push('/')">index</div>
      <div @click="$router.push('/about')">about</div>
    </div>
    <router-view />
  </div>
</template>

<script>
  export default {
    name: 'App',
    data: () => ({}),
    methods: {},
  };
</script>

<style lang="less" scoped>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    transform: rotate(0deg);
    .menu {
      display: flex;
      justify-content: center;
      div {
        margin: 10px 10px;
        color: #157ec4;
        font-size: 20px;
        cursor: pointer;
      }
    }
  }
</style>
```

这样就可以在页面上看到了一个页面，和一个目录，点击目录可以更换 router。  
基本的已经开发完了～  
可以优化一下啦～～

### 优化代码写生产环节的 webpack.js

相关优化：

- 拆分代码
- happyPack 多进程打包
- 添加 babel-polyfill
- clean-webpack-plugin 来清理之前打包的文件

具体内容可以参考 [Github](https://github.com/kaisa911/vue-template)

webpack 文件夹下创建 webpack.prod.js

```js
const common = require('./webpack.common');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssPlugin = require('optimize-css-assets-webpack-plugin');
const UglifyjsWebpackPlugin = require('uglifyjs-webpack-plugin');

const config = {
  ...common,
  mode: 'production',
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          name: 'vendor',
          test: /[\\/]node_modules[\\/]/,
          priority: 10,
          chunks: 'all',
        },
      },
    },
    minimizer: [
      new UglifyjsWebpackPlugin({
        uglifyOptions: {
          compress: {
            comparisons: false,
            drop_console: true,
          },
        },
        // Use multi-process parallel running to improve the build speed
        // Default number of concurrent runs: os.cpus().length - 1
        parallel: true,
        // Enable file caching
        cache: true,
        sourceMap: true,
      }),
      new OptimizeCssPlugin(),
    ],
    namedModules: true,
    occurrenceOrder: false,
  },

  plugins: [
    ...common.plugins,
    new MiniCssExtractPlugin({
      filename: 'css/[name].css',
    }),
  ],
};

module.exports = config;
```

### 给项目添加 eslint 和 prettier 控制代码质量

安装 eslint 和 prettier 相关的第三方插件

```js
"eslint": "^5.5.1",
"eslint-config-prettier": "^6.4.0",
"eslint-friendly-formatter": "^4.0.1",
"eslint-loader": "^3.0.2",
"eslint-plugin-html": "^6.0.0",
"eslint-plugin-prettier": "^3.1.1",
"eslint-plugin-vue": "^5.2.3",
"prettier": "^1.18.2",
"prettier-eslint": "^9.0.0",
"prettier-eslint-cli": "^5.0.0",
```

在 common.js 里添加 eslint-loader 的处理

```js
{
  test: /\.(js|vue)$/,
  loader: 'eslint-loader',
  enforce: 'pre',
  exclude: /node_modules/,
  options: {
    formatter: require('eslint-friendly-formatter'),
    fix: true, // 自动修复
  },
},
```

在根目录里创建.eslintrc.js

```js
module.exports = {
  root: true,
  extends: ['plugin:vue/essential', 'prettier', 'eslint:recommended'],
  parserOptions: {
    sourceType: 'module',
    parser: 'babel-eslint',
    ecmaVersion: 6,
  },
  env: {
    browser: true,
    node: true,
    commonjs: true,
    es6: true,
  },
  plugins: ['html', 'vue', 'prettier'],
  rules: {
    'prettier/prettier': [
      'error',
      {
        singleQuote: true,
        trailingComma: 'es5',
        bracketSpacing: true,
        jsxBracketSameLine: true,
      },
    ],
    complexity: ['warn', { max: 4 }],
  },
};
```

在 package.json 里添加一些命令

```js
 "scripts": {
    "eslint": "eslint --ext .js,.vue src && eslint --ext .js,.vue utils",
    "start": "webpack-dev-server --color --config ./webpack/webpack.dev.js",
    "build": "webpack  --progress --config  ./webpack/webpack.prod.js",
    "lint": "eslint --ext .js,.vue --fix src && eslint --ext .js,.vue --fix utils"
  },
```

### 给项目添加 git 钩子函数

安装 husky

```js
npm i husky -D
```

在 package.json 里添加 precommit 和 prepush 命令

```js
 "scripts": {
    "precommit": "npm run eslint",
    "prepush": "npm run eslint",
    "eslint": "eslint --ext .js,.vue src && eslint --ext .js,.vue utils",
    "start": "webpack-dev-server --color --config ./webpack/webpack.dev.js",
    "build": "webpack  --progress --config  ./webpack/webpack.prod.js",
    "lint": "eslint --ext .js,.vue --fix src && eslint --ext .js,.vue --fix utils"
  },
```

### TODO 给 component 和页面添加单元测试

## 创建一个 cli 脚手架

步骤：

- 创建一个 repo 和 npm init
- 下载第三方库
- 写命令
- 上传到 npm 上

### 创建 repo 和 npm init

在 Github 上创建一个 repo，然后 clone 到本地，然后打开文件夹，在命令行里输入`npm init`

### 下载第三方库

```js
"chalk": "^2.4.2",
"commander": "^3.0.2",
"git-clone": "^0.1.0",
"shelljs": "^0.8.3"
```

### 写 shell 命令

新建一个 bin 文件夹，里面创建一个 PajiCli.js

```js
#!/usr/bin/env node

const clone = require('git-clone');
const program = require('commander');
const shell = require('shelljs');
const chalk = require('chalk');

program.version('1.0.0').description('PajiCli模板工程的cli');
program.command('* <tpl> <project>').action(function(tpl, project) {
  console.info(chalk.blueBright('欢迎使用Paji-Cli'));
  if (tpl === 'create' && project) {
    let pwd = shell.pwd();
    console.info(chalk.blueBright(`正在从github下载Vue模版`));
    console.info(chalk.blueBright(`下载位置：${pwd}/${project}/ ...`));
    clone(`https://github.com/kaisa911/vue-template.git`, pwd + `/${project}`, null, function() {
      shell.rm('-rf', pwd + `/${project}/.git`);
      console.info(chalk.blueBright('下载成功'));
    });
  } else {
    console.info(chalk.redBright('请使用正确的命令 PajiCli create myApp'));
  }
});
program.parse(process.argv);
```

在 package.json 里添加 bin 命令

```js
"bin": {
  "PajiCli": "./bin/PajiCli.js"
},
```

### 上传到 npm 上

```js
npm login
```

```js
npm publish
```

上传到 npm 上之后，就可以下载了

```js
npm i -g paji-cli

```

然后在需要的文件夹下

```js
PajiCli create my-app
cd my-app
npm i
```

就可以使用了

## 总结

这个项目的开发环境，大概花了 2 天的时间，包括写 cli，添加 eslint 等问题。  
这算是在 3.0 之前的一个总结

后续可能还需要继续学习添加单元测试，还有以后的 3.0。
