# keep-alive 有关的生命周期

## keep-alive 是什么

使用一些组件的时候，例如一个 tab，根据不同的 tab 显示不同里面的子内容，其实是要求 tab 作为一个动态组件的。
这时候可以用

```html
<!-- 组件会在 `currentTabComponent` 改变时改变 -->
<component v-bind:is="currentTabComponent"></component>
```

其中`currentTabComponent`可以是

- 已注册组件的名字，或
- 一个组件的选项对象

每次切换都会重新建立这个组件实例，对于一些切换想保存之前内容，很显然是不适用的
我们更希望那些标签的组件实例能够被在它们第一次被创建的时候缓存下来。为了解决这个问题，我们可以用一个 `<keep-alive>` 元素将其动态组件包裹起来。

```html
<!-- 失活的组件将会被缓存！-->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

> 注意这个 <keep-alive> 要求被切换到的组件都有自己的名字，不论是通过组件的 name 选项还是局部/全局注册

## keep-alive 的生命周期

分为第一次和第二次进入，很显然通过 activated，和 deactivated 限制了完整的生命周期走完

1. activated： 页面第一次进入的时候，钩子触发的顺序是 created->mounted->activated
2. deactivated: 页面退出的时候会触发 deactivated，当再次前进或者后退的时候只触发 activated
