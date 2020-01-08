# 你有使用过 JSX 吗？说说你对 JSX 的理解

JSX 是 React 的一种标签语法，既不是 Js，也不是 Html。
它把两者结合到了一起。通过 JSX 把逻辑和 UI 耦合在了一起。
通过 JSX 可以生成 React 或者 Vue 的 vDom。

babel 会把 JSX 编译成一个函数，然后用这个函数，把 JSX 编译成对象。

## 在 Vue 里使用 JSX

通过添加这三个插件，在 babel 里配置一下，就可以使用了

```
"babel-helper-vue-jsx-merge-props",
"babel-plugin-syntax-jsx",
"babel-plugin-transform-vue-jsx",
```

```
// .babelrc file
 "plugins": ["transform-vue-jsx", "transform-runtime"]
```

```js
// 组件
const Footer = {
  data: () => {
    return {
      author: 'paji',
    };
  },
  render() {
    return (
      <div id='footer'>
        <span>written by {this.author}</span>
      </div>
    );
  },
};

export default Footer;
```

就 ok 了
