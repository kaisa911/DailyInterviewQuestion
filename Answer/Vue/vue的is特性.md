# vue 的 is 特性

使用对象：`string | Object` (组件的选项对象)

用于动态组件且基于 DOM 内模板的限制来工作。

示例：

```html
<!-- 当 `currentView` 改变时，组件也跟着改变 -->
<component v-bind:is="currentView"></component>

<!-- 这样做是有必要的，因为 `<my-row>` 放在一个 -->
<!-- `<table>` 内可能无效且被放置到外面 -->
<table>
  <tr is="my-row"></tr>
</table>
```

my-row,可以为一个已经注册的组件的名字，或一个组件的选项对象。
