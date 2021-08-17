# 谈谈对 Vue 表单修饰符 lazy 的理解

这个 lazy 修饰符，是放在 v-model 上的，

添加之后，vue 并不会立即监听 input 的@input 事件，会在 input 失去焦点之后，才会触发其 value 的改变。

相当于 input 事件变成了 change 事件
