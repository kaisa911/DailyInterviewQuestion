# vue 强制刷新组件

## 使用 v-if 指令

如果是强制刷新某个子组件的话，使用 v-if 会让子组件的值重新渲染一边，例如将其先命名为 false，再赋值为 true

## 使用 this.\$forceUpdate 强制重新渲染

这个比较简单直接在组件内设置函数执行这个函数就行。this.\$forceUpdate()

## 改变子组件的 key

改变子组件的 key 值也相当于从新开启个子组件，便不会在新开子组件后，复用子组件之前的数据了。
