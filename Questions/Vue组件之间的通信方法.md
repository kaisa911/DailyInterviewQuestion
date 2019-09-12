# Vue 组件之间的通信方法

## props

props 可能是写在官方文档上最直白的组件间通信的方法了。

Prop 是你可以在组件上注册的一些自定义特性。当一个值传递给一个 prop 特性的时候，它就变成了那个组件实例的一个属性。为了给博文组件传递一个标题，我们可以用一个 props 选项将其包含在该组件可接受的 prop 列表中：

```js
Vue.component("blog-post", {
  // 在 JavaScript 中是 camelCase 的
  props: ["postTitle"],
  template: "<h3>{{ postTitle }}</h3>"
});
```

```html
<!-- 在 HTML 中是 kebab-case 的 -->
<blog-post post-title="hello!"></blog-post>
```

## Vuex

Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。使用了 flux 的思想。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。

主要还是为了解决单项数据流在多组件共享数据时的一些问题。类似在一颗组件树中建立了一个中立区域，可以跨越组件之间的单向流动关系来获取这块地方的数据。

## 浏览器方式

浏览器中可以获取信息的地方有 url,session storage,local storage

### 路由

使用 query 可以在路由跳转的时候带出部分 Id 信息.
发送

```js
const xxxId = "213213";
this.$router.push({ path: "/path", query: { xxxId } });
```

接收

```js
const xxxId = "213213";
this.$router.push({ path: "/path", query: { xxxId } });
```

### session storage

设置

```js
const code = "2323123";
window.sessionStorage.setItem("myId", code);
```

读取

```js
myId: window.sessionStorage.getItem("myId");
```

local storage 也大同小异就不再赘述

## 大佬的补充

1. 子组件通过\$emit()方法，向父组件传递事件和参数。
2. 父组件用 provide 向子组件传递信息。子组件，子子组件都可以用 inject 来获取到数据
3. 通过 eventBus 来传递参数。（小项目少页面时使用）
   声明一个 bus.js 文件
   可以使用 $emit， $on， $once，$off 分别来分发、监听、取消监听事件：

   ```js
   //bus.js
   import Vue from "vue";
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

4. 传到服务器，新组件从服务端获取
5. 变量挂载到 window，vue 上，然后在上面获取（只限单页）
