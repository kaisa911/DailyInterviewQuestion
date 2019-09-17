# vue 怎么获取 DOM 节点

> vue 中，在 mounted 之后的钩子 函数中才能进行 dom 操作，mounted 完成 dom 挂载

## 选择器获取

能通过原生方法去修改，获得 DOM 对象属性,例如

```js
//直接取其id
this.$el.querySelector('#copy');
```

## ref 获取

ref 被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的 \$refs 对象上。如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例：

```html
<!-- `vm.$refs.p` will be the DOM node -->
<p ref="p">hello</p>

<!-- `vm.$refs.child` will be the child component instance -->
<child-component ref="child"></child-component>
```

当 v-for 用于元素或组件的时候，引用信息将是包含 DOM 节点或组件实例的数组。

关于 ref 注册时间的重要说明：因为 ref 本身是作为渲染结果被创建的，在初始渲染的时候你不能访问它们 - **它们还不存在**！\$refs 也**不是响应式**的，因此你不应该试图用它在模板中做数据绑定。
可在 nextTick 中回调使用这个，使其变为响应式更新

```js
this.$nextTick(function() {
  this.$refs.list;
});
```
