# 关于 vue 的 extend（构造器） 方法

Vue 的 extend(options)  
是一个用来构建 Vue 的子类的构造器方法，参数是一个对象，用来创建一个组件

使用基础 Vue 构造器，创建一个“子类”。参数是一个包含组件选项的对象。

data 选项是特例，需要注意 - 在 Vue.extend() 中它必须是函数

```html
<div id="mount-point"></div>
```

```js
// 创建构造器
var Profile = Vue.extend({
  template: '<p>{{firstName}} {{lastName}} aka {{alias}}</p>',
  data: function() {
    return {
      firstName: 'Walter',
      lastName: 'White',
      alias: 'Heisenberg',
    };
  },
});
// 创建 Profile 实例，并挂载到一个元素上。
new Profile().$mount('#mount-point');
```

结果如下：

```html
<p>Walter White aka Heisenberg</p>
```
