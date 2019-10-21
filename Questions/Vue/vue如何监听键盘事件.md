# vue 如何监听键盘事件?

在监听键盘事件时，我们经常需要检查详细的按键。Vue 允许为 v-on 在监听键盘事件时添加按键修饰符：

```html
<!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
<input v-on:keyup.enter="submit" />
```

你可以直接将 KeyboardEvent.key 暴露的任意有效按键名转换为 kebab-case 来作为修饰符。

```html
<input v-on:keyup.page-down="onPageDown" />
```

在上述示例中，处理函数只会在 \$event.key 等于 PageDown 时被调用。

旧的浏览器还可以用 keyCode 来处理键盘事件,新的浏览器可能不支持

```html
<input v-on:keyup.13="submit" />
```

为了在必要的情况下支持旧浏览器，Vue 提供了绝大多数常用的按键码的别名：

- .enter
- .tab
- .delete (捕获“删除”和“退格”键)
- .esc
- .space
- .up
- .down
- .left
- .right

常用的键盘事件：

- keyup
- keypress
- keydown
