# vue 使用 mixin

混入 (mixin) 提供了一种非常灵活的方式，来分发 Vue 组件中的可复用功能。一个混入对象可以包含任意组件选项。当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项。
使用场景，例如多个页面需要制定一个相同的数据结构。或使用同一个函数进行跳转等操作。

## 选项混入

### 数据递归混入

当组件和混入对象含有同名选项时，这些选项将以恰当的方式进行“合并”。

比如，数据对象在内部会进行递归合并，并在发生冲突时以组件数据优先。

```js
var mixin = {
  data: function() {
    return {
      message: 'hello',
      foo: 'abc',
    };
  },
};

new Vue({
  mixins: [mixin],
  data: function() {
    return {
      message: 'goodbye',
      bar: 'def',
    };
  },
  created: function() {
    console.log(this.$data);
    // => { message: "goodbye", foo: "abc", bar: "def" }
  },
});
```

### 钩子混入

同名钩子函数将合并为一个数组，因此都将被调用。另外，混入对象的钩子将在组件自身钩子之前调用。

```js
var mixin = {
  created: function() {
    console.log('混入对象的钩子被调用');
  },
};

new Vue({
  mixins: [mixin],
  created: function() {
    console.log('组件钩子被调用');
  },
});

// => "混入对象的钩子被调用"
// => "组件钩子被调用"
```

### 函数混入

值为对象的选项，例如 methods、components 和 directives，将被合并为同一个对象。两个对象键名冲突时，取组件对象的键值对。

```js
var mixin = {
  methods: {
    foo: function() {
      console.log('foo');
    },
    conflicting: function() {
      console.log('from mixin');
    },
  },
};

var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function() {
      console.log('bar');
    },
    conflicting: function() {
      console.log('from self');
    },
  },
});

vm.foo(); // => "foo"
vm.bar(); // => "bar"
vm.conflicting(); // => "from self"
```

> 注意：Vue.extend() 也使用同样的策略进行合并。

## 自定义规则

自定义选项将使用默认策略，即简单地覆盖已有值。如果想让自定义选项以自定义逻辑合并，可以向 Vue.config.optionMergeStrategies 添加一个函数：

```js
Vue.config.optionMergeStrategies.myOption = function(toVal, fromVal) {
  // 返回合并后的值
};
```

对于多数值为对象的选项，可以使用与 methods 相同的合并策略：

```js
var strategies = Vue.config.optionMergeStrategies;
strategies.myOption = strategies.methods;
```

可以在 Vuex 1.x 的混入策略里找到一个更高级的例子：

```js
const merge = Vue.config.optionMergeStrategies.computed;
Vue.config.optionMergeStrategies.vuex = function(toVal, fromVal) {
  if (!toVal) return fromVal;
  if (!fromVal) return toVal;
  return {
    getters: merge(toVal.getters, fromVal.getters),
    state: merge(toVal.state, fromVal.state),
    actions: merge(toVal.actions, fromVal.actions),
  };
};
```
