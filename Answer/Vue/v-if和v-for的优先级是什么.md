# v-if 和 v-for 的优先级是什么

## 优先级

当 Vue 处理指令时，`v-for` 比 `v-if` 具有更高的优先级，所以这个模板：

```html
<ul>
  <li v-for="user in users" v-if="user.isActive" :key="user.id">
    {{ user.name }}
  </li>
</ul>
```

会经过如下运算

```js
this.users.map(function(user) {
  if (user.isActive) {
    return user.name;
  }
});
```

## 详情信息

这样的计算是得不偿失的，哪怕我们只渲染出一小部分用户的元素，也得在每次重渲染的时候遍历整个列表，不论活跃用户是否发生了变化。

通过将其变为一个计算属性上的遍历

```js
computed: {
  activeUsers: function () {
    return this.users.filter(function (user) {
      return user.isActive
    })
  }
}
```

```html
<ul>
  <li v-for="user in activeUsers" :key="user.id">
    {{ user.name }}
  </li>
</ul>
```

会获得如下好处：

- 过滤后的列表只会在 users 数组发生相关变化时才被重新运算，过滤更高效。
- 使用 v-for="user in activeUsers" 之后，我们在渲染的时候只遍历活跃用户，渲染更高效。
- 解耦渲染层的逻辑，可维护性 (对逻辑的更改和扩展) 更强。

对于另外一种判断使用逻辑可以这样

```html
<ul>
  <li v-for="user in users" v-if="shouldShowUsers" :key="user.id">
    {{ user.name }}
  </li>
</ul>
```

更新为

```html
<ul v-if="shouldShowUsers">
  <li v-for="user in users" :key="user.id">
    {{ user.name }}
  </li>
</ul>
```

通过将 `v-if` 移动到容器元素，不会再对列表中的每个用户检查 `shouldShowUsers`。取而代之的是，我们只检查它一次，且不会在 `shouldShowUsers` 为否的时候运算 `v-for`。

## 结论

- 为了过滤一个列表中的项目 (比如 `v-for="user in users"` `v-if="user.isActive"`)。在这种情形下，请将 users 替换为一个计算属性 (比如 activeUsers)，让其返回过滤后的列表。

- 为了避免渲染本应该被隐藏的列表 (比如 `v-for="user in users"` `v-if="shouldShowUsers"`)。这种情形下，请将 v-if 移动至容器元素上 (比如 ul, ol)。
