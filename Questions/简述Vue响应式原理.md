# 简述 Vue 响应式原理

Vue 是一个响应式的框架，响应式是指：数据模型仅仅是普通的 JavaScript 对象。而当你修改它们时，视图会进行更新。

那么 Vue 是怎么做到响应式的呢？

可以从两个方面来讨论：Object 和 Array。

## Object

受现代 JavaScript 的限制 (以及废弃 Object.observe)，Vue 不能检测到对象属性的添加或删除。所以 Vue 通过 Object.defineProperty 来处理数据，把属性变成 getter/setter 属性，因为 Object.defineProperty 是 ES5 的一个不能 shim 的特性，所以不支持 IE8 及以下。

### 1、如何追踪变化：

Vue 追踪变化，是把对象用 Object.defineProperty 把遍历对象，使用 Observer 类把对象的每个属性改写成用 getter/setter 来设置和获取值的改变。这样 Vue 就可以通过触发 getter 和 setter 方法来处理数据，就可以获取数据的改变。

### 2、依赖的收集和触发

2.1 什么是依赖？

用到该属性的地方，就是依赖，但是 Vue 封装了一个 watch 类来当作依赖，这个 watch 类能通知使用该属性的所有依赖。所以 Vue 收集的依赖就是收集 watch 类。

2.2 如何收集依赖？

Vue 在 getter 方法里收集依赖，Vue 把依赖解耦成一个专门的 Dep 类，来处理收集和触发依赖。在使用该数据的时候，便把依赖收集到 Dep 里。

2.3 如何触发依赖

在数据发生改变的 setter 函数里触发依赖，Vue 声明的 watch 类，当数据发生了改变的时候，Vue 通知 watch 数据发生变化，让 watch 自己去通知依赖发生改变。

### 3、Object 的问题

Vue 使用 Object.defineProperty 的 getter 和 setter 来把对象变成响应式的对象，但是也因为这个，Vue 能监听到该 Object 属性值的变化，却监听不到 Object 增加和删除属性。所以 Vue 提供了两个 API：vm.$set和vm.$delete 来处理增加属性和删除属性。

## Array

Vue 对象是通过 getter/setter 来监听到对象的数据变化从而让对象变成响应式的。但是在数组里，有一些原型上的方法比如 push，pop，shift…等，并不会触发 getter/setter 就可以改变数组的值，Vue 在这个时候，就没有办法进行监听从而触发视图的改变了。

### 1、如何追踪变化

Vue 重写了 push，pop，shift，unshift，reverse，sort，splice 这 7 个会改变 Array 本身的原型方法，并通过一个拦截器，把需要响应的数组，覆盖他们的原型方法。value.**proto**=arraysMethods.如果不支持**proto**的属性，那么 Vue 会暴力的把这些方法放到这些数组本身。

### 2、依赖的收集和触发

2.1 在哪儿收集和触发依赖

Vue 对数组和对象都一样，都是在 getter 里收集依赖，在 setter 里触发依赖。

2.2 依赖放在哪儿

对象的依赖放在 Dep 类里，而数组的依赖放在 Observer 实例的 Dep 里。目的是为了在 getter 和 Array 的拦截器都里能访问到 Observer 的实例。

Vue 改写了上面 Object 的 Observer 的类，给每个属性加一个**ob**属性，通过对该属性的来判断是否响应式，且是否发生改变，从而触发依赖。
