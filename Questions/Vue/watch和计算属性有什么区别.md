# watch 和计算属性有什么区别

## computed 例子

```html
<div id="example">
  <div id="demo">{{ fullName }}</div>
</div>
```

```js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
  },
  computed: {
    fullName: function() {
      return this.firstName + ' ' + this.lastName;
    },
  },
});
```

computed 无非是创建了个函数返回相关的计算的值，有点类似，在 method 里写值，不同的是，计算属性是基于它们的响应式依赖进行缓存的，computed 值存在缓存中，如果依赖没有变化便不会执行。而是返回缓存。

## watch 例子

```js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar',
  },
  watch: {
    firstName: function(val) {
      this.fullName = val + ' ' + this.lastName;
    },
    lastName: function(val) {
      this.fullName = this.firstName + ' ' + val;
    },
  },
});
```

watch 则是设置监控的值，每次监控的值发生变化便去执行函数。如果需要监控多个值，且需返回这些值的依赖的话，使用
watch 是有点舍近求远了。

## 异同

watch 相比 computed 不算那么智能，监视的是一个个自己设定的变量值，但处理方法多种多样。
computed 的话能够智能处理分析的依赖，并返回计算后的值，同时也可以使用 set 方法来实现，根据返回的属性值的变化，去控制别的属性值。

watch 是一个经常会被滥用的方法还是得根据具体工况去选择适合的监听方法。
