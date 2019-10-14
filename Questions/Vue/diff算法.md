# diff 算法

vue 2.0 加入了 virtual dom，觉得 react 不错可以学习借鉴一下。diff 位于 patch.js 中。复杂度为线性复杂度。

## 虚拟节点

说白了就是用 js 的对象去模拟 dom 中的节点，然后通过特定方法去将其悄悄渲染成真实节点。diff 则是通过计算，返回一个补丁对象 patch，通过解析它，完成页面渲染

就是在 model 上改啊改，再到虚拟 dom 上去找，哪里改了，以最小的开销去进行页面的重新渲染。

虚拟 dom 对应就是真实 dom

```js
var odiv = document.createElement('div');
for (var k in odiv) {
  console.log(k);
}
```

## 实现步骤

- 用 JavaScript 对象模拟 DOM
- 把此虚拟 DOM 转成真实 DOM 并插入页面中
- 如果有事件发生修改了虚拟 DOM
- 比较两棵虚拟 DOM 树的差异，得到差异对象
- 把差异对象应用到真正的 DOM 树上

```js
class crtateElement {
  constructor(el, attr, child) {
    this.el = el;
    this.attrs = attr;
    this.child = child || [];
  }
  render() {
    let virtualDOM = document.createElement(this.el);
    // attr是个对象所以要遍历渲染
    for (var attr in this.attrs) {
      virtualDOM.setAttribute(attr, this.attrs[attr]);
    }

    // 深度遍历child
    this.child.forEach(el => {
      console.log(el instanceof crtateElement);
      //如果子节点是一个元素的话，就调用它的render方法创建子节点的真实DOM，如果是一个字符串的话，创建一个文件节点就可以了
      // 判断一个对象是否是某个对象的实力
      let childElement =
        el instanceof crtateElement ? el.render() : document.createTextNode(el);
      virtualDOM.appendChild(childElement);
    });
    return virtualDOM;
  }
}
function element(el, attr, child) {
  return new crtateElement(el, attr, child);
}

module.exports = element;
```

插入

```js
let element = require('./element');

let myobj = {
  class: 'big_div',
};
let ul = element('div', myobj, [
  '我是文字',
  element('div', { id: 'xiao' }, ['1']),
  element('div', { id: 'xiao1' }, ['2']),
  element('div', { id: 'xiao2' }, ['3']),
]);
console.log(ul);
ul = ul.render();
document.body.appendChild(ul);
```

## DOM DIFF

比较两棵 DOM 树的差异是 Virtual DOM 算法最核心的部分.简单的说就是新旧虚拟 dom 的比较，如果有差异就以新的为准，然后再插入的真实的 dom 中，重新渲染。、 借网络一张图片说明:

比较只会在同层级进行, 不会跨层级比较。
比较后会出现四种情况：
1、此节点是否被移除 -> 添加新的节点
2、属性是否被改变 -> 旧属性改为新属性
3、文本内容被改变-> 旧内容改为新内容
4、节点要被整个替换 -> 结构完全不相同 移除整个替换
