# vue 中怎么重置 data

Vue 提供了获取组件当前状态下 data 的值和获取组件 data 初始状态的值的 api

- `this.$data`可以拿到当前状态下 data 的值
- `this.$options.data()` 可以拿到组件初始状态的 data 的值

所以重置 data 就可以用各种各样的方式了…

```js
// assign
Object.assign(this.$data, this.$options.data());

// rest
this.$data = { ...this.$data, ...this.$options.data() };
```

上面提供了整体重置的方式，还有单个属性的重置：

- 如果知道类型，可以直接替换 this.name = '';
- 如果不知道类型，可以使用 `this.$data[name] = this.$options.data()[name]`
