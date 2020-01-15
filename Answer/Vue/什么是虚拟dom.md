<!--
 * @Author: Dongzy
 * @since: 2020-01-15 19:56:45
 * @lastTime     : 2020-01-15 20:25:03
 * @LastAuthor   : Dongzy
 * @文件相对于项目的路径: \DailyInterviewQuestion\Answer\Vue\什么是虚拟dom.md
 * @message:
 -->

# 什么是虚拟 dom

## 起

当应用功能变多， 直接操作 dom 节点变得难以维护时，有人提出在 js 中维护一个真实 dom 的抽象树，将操作放置在抽象树上，抽象树修改后再将其重绘到页面上。抽象树变化就去修改视图。

## 承

最简单的修改方法时直接将整个结构用 innerHTML 打在页面上，但是这样的重绘是非常消耗浏览器性能的。所以就有了虚拟 dom

## 转

vue 以 vnode 去模拟真实 dom，将其构造成抽象树，在上面执行各种操作，同时将修改操作移在这个抽象树上，经过 diff 算法，计算出最小的差异，再将这些最小差异更新。

## 合

vue 的 vnode 是一个类，里面定义了大量的功能属性，
同时有几种方法可以去生成 vnode

- createEmptyVNode ，创建一个空 vnode
- createTextVNode，创建一个文字 vnode
- createComponent，创建一个组件 vnode
- cloneVNode 克隆一个 vnode
