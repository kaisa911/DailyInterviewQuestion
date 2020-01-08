# v-model 的原理

`v-model` 本质上不过是语法糖。它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。

> v-model 会忽略所有表单元素的 value、checked、selected 特性的初始值而总是将 Vue 实例的数据作为数据来源。你应该通过 JavaScript 在组件的 data 选项中声明初始值。

v-model 在内部为不同的输入元素使用不同的属性并抛出不同的事件：

- text 和 textarea 元素使用 value 属性和 input 事件；
- checkbox 和 radio 使用 checked 属性和 change 事件；
- select 字段将 value 作为 prop 并将 change 作为事件。

相当于一个语法糖封装了部分组件的值的监听与输入的绑定

`v-bind` 了对应的值，`v-on`去监听对应的事件变化

```html
<my-component v-model="price"></my-component>
```

```html
// 拆解如下
<my-component
  :value="price"
  @input="price = $event.target.value"
></my-component>
```
