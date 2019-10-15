#

vue 2.0 加入了 virtual dom，觉得 react 不错可以学习借鉴一下。diff 位于 patch.js 中。复杂度为线性复杂度。

## 虚拟节点

说白了就是用 js 的对象去模拟 dom 中的节点，然后通过特定方法去将其悄悄渲染成真实节点。diff 则是通过计算，返回一个补丁对象 patch，通过解析它，完成页面渲染
diff 算法
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

![](https://user-gold-cdn.xitu.io/2018/4/18/162d4772f1f40c49?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```js
let utils = require('./utils');

let keyIndex = 0;
function diff(oldTree, newTree) {
  //记录差异的空对象。key就是老节点在原来虚拟DOM树中的序号，值就是一个差异对象数组
  let patches = {};
  keyIndex = 0; // 儿子要起另外一个标识
  let index = 0; // 父亲的表示 1 儿子的标识就是1.1 1.2
  walk(oldTree, newTree, index, patches);
  return patches;
}
//遍历
function walk(oldNode, newNode, index, patches) {
  let currentPatches = []; //这个数组里记录了所有的oldNode的变化
  if (!newNode) {
    //如果新节点没有了，则认为此节点被删除了
    currentPatches.push({ type: utils.REMOVE, index });
    //如果说老节点的新的节点都是文本节点的话
  } else if (utils.isString(oldNode) && utils.isString(newNode)) {
    //如果新的字符符值和旧的不一样
    if (oldNode != newNode) {
      ///文本改变
      currentPatches.push({ type: utils.TEXT, content: newNode });
    }
  } else if (oldNode.tagName == newNode.tagName) {
    //比较新旧元素的属性对象
    let attrsPatch = diffAttr(oldNode.attrs, newNode.attrs);
    //如果新旧元素有差异 的属性的话
    if (Object.keys(attrsPatch).length > 0) {
      //添加到差异数组中去
      currentPatches.push({ type: utils.ATTRS, attrs: attrsPatch });
    }
    //自己比完后再比自己的儿子们
    diffChildren(
      oldNode.children,
      newNode.children,
      index,
      patches,
      currentPatches
    );
  } else {
    currentPatches.push({ type: utils.REPLACE, node: newNode });
  }
  if (currentPatches.length > 0) {
    patches[index] = currentPatches;
  }
}
//老的节点的儿子们 新节点的儿子们 父节点的序号 完整补丁对象 当前旧节点的补丁对象
function diffChildren(
  oldChildren,
  newChildren,
  index,
  patches,
  currentPatches
) {
  oldChildren.forEach((child, idx) => {
    walk(child, newChildren[idx], ++keyIndex, patches);
  });
}
function diffAttr(oldAttrs, newAttrs) {
  let attrsPatch = {};
  for (let attr in oldAttrs) {
    //如果说老的属性和新属性不一样。一种是值改变 ，一种是属性被删除 了
    if (oldAttrs[attr] != newAttrs[attr]) {
      attrsPatch[attr] = newAttrs[attr];
    }
  }
  for (let attr in newAttrs) {
    if (!oldAttrs.hasOwnProperty(attr)) {
      attrsPatch[attr] = newAttrs[attr];
    }
  }
  return attrsPatch;
}
module.exports = diff;
```

新旧节点对比的时候，先是同级的进行对比，然后就是对比子节点，有点类似广度优先搜索

patch.js 的简单实现

```js
let keyIndex = 0;
let utils = require('./utils');
let allPatches; //这里就是完整的补丁包
function patch(root, patches) {
  allPatches = patches;
  walk(root);
}
function walk(node) {
  let currentPatches = allPatches[keyIndex++];
  (node.childNodes || []).forEach(child => walk(child));
  if (currentPatches) {
    doPatch(node, currentPatches);
  }
}
function doPatch(node, currentPatches) {
  currentPatches.forEach(patch => {
    switch (patch.type) {
      case utils.ATTRS:
        for (let attr in patch.attrs) {
          let value = patch.attrs[attr];
          if (value) {
            utils.setAttr(node, attr, value);
          } else {
            node.removeAttribute(attr);
          }
        }
        break;
      case utils.TEXT:
        node.textContent = patch.content;
        break;
      case utils.REPLACE:
        let newNode =
          patch.node instanceof Element
            ? path.node.render()
            : document.createTextNode(path.node);
        node.parentNode.replaceChild(newNode, node);
        break;
      case utils.REMOVE:
        node.parentNode.removeChild(node);
        break;
    }
  });
}
module.exports = patch;
```
