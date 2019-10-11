# watch 怎么深度监听对象变化？

在 watch 的属性里，添加一个 deep 的属性，且值为 true。  
也可以把想监听的对象里的某个属性，写在 watch 里

Vue 的 watch 属性，除了直接在上面写方法，还有写对象的方式；

```js
watch: {
  firstName: {
    handler(newName, oldName) {
      this.fullName = newName + ' ' + this.lastName;
    },
    immediate: true,
    deep:true
  },
  'obj.a.b': 'handlerB'
}

```

除了 handler 直接写方法外，还有很多方式

```js
watch: {
  firstName: {
    handler: 'handleFullName',
    immediate: true,
    deep:true
  }
},
methods: {
  handleFullName(newName, oldName) {
    this.fullName = newName + ' ' + this.lastName;
  }
}
```

```js
watch: {
  firstName: 'handleFullName',
},

methods: {
  handleFullName(newName, oldName) {
    this.fullName = newName + ' ' + this.lastName;
  }
}
```

注销监听

正常写的监听，会随着组件的销毁而注销，但是用\$watch 写的，就需要手动注销

```js
const unWatch = app.$watch('text', (newVal, oldVal) => {
  console.log(`${newVal} : ${oldVal}`);
});

unWatch(); // 手动注销watch
```
