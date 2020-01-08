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

computed 的话能够智能处理分析的依赖，并返回计算后的值，同时也可以使用 set 方法来实现，根据返回的属性值的变化，去控制别的属性值。
computed 无非是创建了个函数返回相关的计算的值，有点类似，在 method 里写值，不同的是，计算属性是基于它们的响应式依赖进行缓存的，computed 值存在缓存中，如果依赖没有变化便不会执行。而是返回缓存。值得注意的是“fullName”不能在组件的 props 和 data 中定义，否则会报错。

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

watch 则是设置监控的值，每次监控的值发生变化便去执行函数。使用 watch 选项允许我们执行异步操作 (访问一个 API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

## 异同

**相同**:computed 和 watch 都起到监听/依赖一个数据，并进行处理的作用

**不同**:它们都是 vue 对监听器的实现，只不过 computed 主要用于对同步数据的处理，watch 则主要用于观测某个值的变化去完成一段开销较大的复杂业务逻辑。能用 computed 的时候优先用 computed，避免了多个数据影响其中某个数据时多次调用 watch 的尴尬情况。

## watch 的高级用法

### handler

作为 watch 默认执行的函数

```js
watch: {
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
```

等同于的默认写法

```js
 watch: {
    a: {
      handler: function(val, oldVal) {
        console.log('new: %s, old: %s', val, oldVal);
      }
    }
  },
```

### immediate 属性

这个英文单词就是立即的意思,所以很清楚,这个函数在 watch 中设立的时候便会被执行一次.

```js
d: {
      handler: 'someMethod',
      immediate: true
    },
```

### deep 属性

```js
 obj: {
      handler: function(val, oldVal) {/xxxxx/
      },
      deep:true
    }
  },
```

deep 属性的意思是深度遍历，会在对象一层层往下遍历，在每一层都加上监听器。从此可以监听对象内部值的变化.这个一般在动态响应中是检测不到的.
但是使用 deep 属性会给每一层都加上监听器，性能开销可能就会非常大了。
所以也可以直接监听 obj.a 这个属性

### 总结

计算属性本质上是一个 computed watch，侦听属性本质上是一个 user watch。
