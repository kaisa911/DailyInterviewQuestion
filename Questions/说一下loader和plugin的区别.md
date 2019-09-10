# loader 和 plugin 的区别

## 主要区别：

- loader 用于加载某些资源文件。 因为 webpack 本身只能打包 commonjs 规范的 js 文件，对于其他资源例如 css，图片，或者其他的语法集，比如 jsx， coffee， vue，是没有办法加载的。 这就需要对应的 loader 将资源转化，加载进来。从字面意思也能看出，loader 是用于加载的，它作用于一个个文件上。loader 用于对模块的源代码进行转换。loader 可以使你在 import 或"加载"模块时预处理文件。因此，loader 类似于其他构建工具中“任务(task)”，并提供了处理前端构建步骤的强大方法。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript，或将内联图像转换为 data URL。loader 甚至允许你直接在 JavaScript 模块中 import CSS 文件！ 因为 webpack 本身只能处理 JavaScript，如果要处理其他类型的文件，就需要使用 loader 进行转换，loader 本身就是一个函数，接受源文件为参数，返回转换的结果。

- plugin 用于扩展 webpack 的功能。它直接作用于 webpack，扩展了它的功能。 plugin 的功能更加的丰富，而不仅局限于资源的加载。Plugin 是用来扩展 Webpack 功能的，通过在构建流程里注入钩子实现，它给 Webpack 带来了很大的灵活性。 通过 plugin（插件）webpack 可以实 loader 所不能完成的复杂功能，使用 plugin 丰富的自定义 API 以及生命周期事件，可以控制 webpack 打包流程的每个环节，实现对 webpack 的自定义功能扩展。

## 常用的 loader

样式：style-loader、css-loader、less-loader、sass-loader 等
文件：raw-loader、file-loader 、url-loader 等
编译：babel-loader、coffee-loader 、ts-loader 等
校验测试：mocha-loader、jshint-loader 、eslint-loader 等

## 常用的 plugin

- 官网介绍 plugins
- 第三方插件 awesome-webpack
- 首先 webpack 内置 UglifyJsPlugin，压缩和混淆代码。
- webpack 内置 CommonsChunkPlugin，提高打包效率，将第三方库和业务代码分开打包。
- ProvidePlugin：自动加载模块，代替 require 和 import
- html-webpack-plugin 可以根据模板自动生成 html 代码，并自动引用 css 和 js 文件
- extract-text-webpack-plugin 将 js 文件中引用的样式单独抽离成 css 文件
- DefinePlugin 编译时配置全局变量，这对开发模式和发布模式的构建允许不同的行为非常有用。
- HotModuleReplacementPlugin 热更新

  - 添加 HotModuleReplacementPlugin
  - entry 中添加 "webpack-dev-server/client?http://localhost:8080/",
  - entry 中添加 "webpack/hot/dev-server"
  - (热更新还可以直接用 webpack_dev_server --hot --inline,原理也是在 entry 中添加了上述代码)

- webpack 内置的 DllPlugin 和 DllReferencePlugin 相互配合，前置第三方包的构建，只构建业务代码，同时能解决 Externals 多次引用问题。DllReferencePlugin 引用 DllPlugin 配置生成的 manifest.json 文件,manifest.json 包含了依赖模块和 module id 的映射关系

- babili-webpack-plugin、transform-runtime 、transform-object-rest-spread

  - babili-webpack-plugin:构建在 babel 之上
  - transform-runtime :解决了 babel 在每个文件都插入了辅助代码，代码体积过大的问题。
  - transform-object-rest-spread：Transform rest properties for object destructuring assignment and spread properties for object literals 为对象字面量添加解构赋值和 spread 属性

- optimize-css-assets-webpack-plugin 不同组件中重复的 css 可以快速去重

- webpack-bundle-analyzer 一个 webpack 的 bundle 文件分析工具，将 bundle 文件以可交互缩放的 treemap 的形式展示。
- compression-webpack-plugin 生产环境可采用 gzip 压缩 JS 和 CSS
- happypack：通过多进程模型，来加速代码构建
