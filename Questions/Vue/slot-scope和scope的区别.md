# slot-scope 和 scope 的区别

> 在 2.6.0 中，我们为具名插槽和作用域插槽引入了一个新的统一的语法 (即 v-slot 指令)。它取代了 slot 和 slot-scope 这两个目前已被废弃但未被移除且仍在文档中的特性。新语法的由来可查阅这份 [RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0001-new-slot-syntax.md)。

## 插槽

进入三部分之前，先让还没接触过插槽的同学对什么是插槽有一个简单的概念：插槽，也就是 slot，是组件的一块 HTML 模板，这块模板显示不显示、以及怎样显示由父组件来决定。 实际上，一个 slot 最核心的两个问题这里就点出来了，是显示不显示和怎样显示。

由于插槽是一块模板，所以，对于任何一个组件，从模板种类的角度来分，其实都可以分为非插槽模板和插槽模板两大类。
非插槽模板指的是 html 模板，指的是‘div、span、ul、table’这些，非插槽模板的显示与隐藏以及怎样显示由插件自身控制；插槽模板是 slot，它是一个空壳子，因为它显示与隐藏以及最后用什么样的 html 模板显示由父组件控制。但是插槽显示的位置确由子组件自身决定，slot 写在组件 template 的哪块，父组件传过来的模板将来就显示在哪块。

## slot-scope

作为插槽，需要的不仅仅是父组件去控制显示，以及显示的方式。子组件去显示其插槽的位置。就相当于父组件在子组件中插了一块地方，用于不用取决于子组件中写了 slot 没。同时既然已经写了一块地方了，数据的传输就显得很重要。其中 slot-scope 就是为此而准备的。

子组件，准备好数据，并绑定至插槽

```js
<slot name="up" :data="data"></slot>
 export default {
    data: function(){
      return {
        data: ['zhangsan','lisi','wanwu','zhaoliu','tianqi','xiaoba']
      }
    },
}
```

父组件，通过 template 的控制来，控制数据的显示

```html
<child>
  <template slot-scope="user">
    <div class="tmpl">
      <span v-for="item in user.data">{{item}}</span>
    </div>
  </template>
</child>
```

## scope

scope 和 slot-scope 不过是同样的传递数据的功能，文档里都没得了，同样的 slot 和 slot-scope 不过也是一样的，这个在 vue3.0 后也不会再支持。

## 参考

[深入理解 vue 中的 slot 与 slot-scope](https://segmentfault.com/a/1190000012996217#articleHeader6)
