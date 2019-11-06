# The difference between “require(x)” and import x

![2](https://i.stack.imgur.com/5WgFJ.png)

还没有 JavaScript 引擎本身支持 ES6 模块。 无论如何，Babel 仍默认将导入和导出声明转换为 CommonJS（require / module.exports）。 因此，即使您使用 ES6 模块语法，但如果在 Node 中运行代码，您也将在后台使用 CommonJS。 CommonJS 和 ES6 模块之间存在技术差异，例如 CommonJS 允许您动态加载模块。 ES6 不允许这样做，但是正在为此开发一个 API。 由于 ES6 模块是标准的一部分，因此使用其是没有问题的。

require：

- 可动态加载的
- 同步的，一起全部加载

ES6 Imports:

- 可以使用解构有选择地仅加载所需的片段。这样可以节省内存
- 异步的，一个个的加载

|      |       require       |       Imports        |
| :--- | :-----------------: | :------------------: |
| 区别 | 同步的 ，可动态加载 | 异步的，不可动态加载 |
