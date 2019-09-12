# Vue 组件之间的通信方法

Vue 组件之间的通信，包括这么几种情况：父子组件之间通信，兄弟组件之间通信，其他通用的一些方法等。

## 父子组件之间通信

父子组件之间的通信，使用最多的是 props 和\$emit()事件。还有一种情况是父组件使用 provide 向子组件传递信息，子组件，子子组件都可以用 inject 来获取到父组件传递的数据。

### 1、props 和 \$emit()

父组件通过 props 向子组件传递数据，子组件通过\$emit()事件，向父组件反馈一个事件，并可以在后面添加许多参数。

### provide 和 inject

```js
<template>
    <c1 message="hello world">
        <c2></c2>
    </c1>
</template>

// c1.vue
<template>
  <div id="c1">
    <slot></slot>
  </div>
</template>

<script>
export default {
  props: ['message'],
  provides () {
    return {
      message: this.message
    }
  }
}
</script>
// c2.vue
<template>
  <div id="c2">
      {{ message }}
  </div>
</template>

<script>
export default {
  inject: ['message']
}
</script>
```

在 c1 组件里，会被渲染上'hello world'

## 兄弟组件之间的通信

兄弟组件之间的通信，通常都是用 vuex 和 eventBus 来传递

## Vuex

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。使用了 flux 的思想。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

主要还是为了解决单项数据流在多组件共享数据时的一些问题。类似在一颗组件树中建立了一个中立区域，可以跨越组件之间的单向流动关系来获取这块地方的数据。

## eventBus

通过 eventBus 来传递参数。（小项目少页面时使用）
声明一个 bus.js 文件
可以使用 $emit， $on， $once，$off 分别来分发、监听、取消监听事件：

```js
//bus.js
import Vue from 'vue';
export default new Vue();
```

A 组件：

```js

<template>
  <div>
    A组件:
    <span>{{elementValue}}</span>
    <input type="button" value="点击触发" @click="elementByValue">
  </div>
</template>
<script>
  // 引入公共的bug，来做为中间传达的工具
  import Bus from './bus.js'
  export default {
    data () {
      return {
        elementValue: 4
      }
    },
    methods: {
      elementByValue: function () {
        Bus.$emit('val', this.elementValue)
      }
    }
  }
</script>

```

B 组件

```js
<template>
<div>
 B组件:
 <input type="button" value="点击触发" @click="getData">
 <span>{{name}}</span>
</div>
</template>
<script>
import Bus from './bus.js'
export default {
 data () {
   return {
     name: 0
   }
 },
// 清除事件监听
beforeDestroy () {
Bus.$off('val')
},
 mounted: function () {
   var vm = this
   // 用$on事件来接收参数
   Bus.$on('val', (data) => {
     console.log(data)
     vm.name = data
   })
 },
 methods: {
   getData: function () {
     this.name++
   }
 }
}
</script>
```

## 其他通用的参数传递方法

1. 通过 url 的 query 和 state
2. 通过 localStorage，sessionStorage 等前端持久化的方式传递。
3. 单页可以存放在 window，Vue 上。
