# v-on 可以绑定多个方法吗？

v-on 是支持绑定多个事件的。  
具体的使用方法是：

```html
<!-- v-on绑定多个事件 -->
<p v-on="{click:dbClick,mousemove:MouseClick}"></p>

<!-- 一个事件绑定多个函数 -->
<p @click="one(),two()">点击</p>
```
