<!--
 * @Author: Dongzy
 * @since: 2020-01-03 10:24:23
 * @lastTime     : 2020-01-03 15:06:28
 * @LastAuthor   : Dongzy
 * @文件相对于项目的路径: \DailyInterviewQuestion\Questions\Vue\template标签有什么用.md
 * @message:
 -->

# template 标签有什么用

template 是模板占位符，

1. 在 vue-load 来说，最外层的 template 标签，意为包裹根元素的一个 html 标签，用来提示在此 template 标签下包裹的即为该组件的根元素，所以在此元素下，**只允许有一个元素**，同时子元素也不能为 template 这种不渲染 dom 的无实意的提示标签（不能保证下一个 template 内只有**一个**实渲染元素。

2. 当你需要循环多个元素时，外侧又不想产生多余的包裹 div 时，可以将`v-for` 写在 template 上，将 key 绑定在 循环内部的元素上
