# vue 生命周期总共有几个阶段，每个阶段的作用是什么？

Vue 实例从开始创建、初始化数据、编译模板、挂载 Dom → 渲染、更新 → 渲染、卸载等一系列过程，我们称这是 Vue 的生命周期，各个阶段有相对应的事件钩子。这次从头整理一下 Vue 的生命周期相关的知识点。希望能把相关的问题解决掉～

先上一张官方的图。
![生命周期](https://ustbhuangyi.github.io/vue-analysis/assets/lifecycle.png)

根据图上，我们可以看到，Vue 的生命周期有创建，挂载，更新，卸载等过程，相应的有八个钩子函数。下面我们来详细的解释一下这几个钩子函数。

## beforeCreate 和 created

这两个钩子函数是在 Vue 实例初始化，然后创建的阶段。在这个阶段，Vue 会初始化 props、data、methods、watch、computed 等属性。但是在这个阶段，没有渲染 DOM，所以不能够访问 DOM。

在 **beforeCreate** 的时候，this 指向创建的实例，但是**不能访问**到 data、computed、watch、methods 上的方法和数据。常用于初始化非响应式变量。

在 **created** 的时候，实例创建完成，可访问 data、computed、watch、methods 上的方法和数据，但是因为并未挂载到 DOM，所以**不能访问到$el属性，$ref 属性内容为空数组**。常用于简单的 ajax 请求，页面的初始化，如果大量的 ajax 请求，会出现长时间的白屏。

## beforeMount 和 mounted

顾名思义，beforeMount 钩子函数发生在 mount，也就是 DOM 挂载之前，它的调用时机是在 mountComponent 函数中。  
在执行 vm.\_render() 函数渲染 VNode 之前，执行了 beforeMount 钩子函数，在执行完 vm.\_update() 把 VNode patch 到真实 DOM 后，执行 mounted 钩子

### 1、created 到 beforeMount 之间，发生了很多事。

- 首先会判断对象是否有 el 选项。如果有的话就继续向下编译，如果没有 el 选项，则停止编译，也就意味着停止了生命周期，直到在该 vue 实例上调用 vm.\$mount(el)。
- 然后，判断 template 参数选项的有无对生命周期的影响。
  1. 如果 vue 实例对象中有 template 参数选项，则将其作为模板编译成 render 函数。
  2. 如果没有 template 选项，则将外部 HTML 作为模板编译。
  3. template 中的模板优先级要高于 outer HTML 的优先级。

### 2、beforeMount

在挂载开始之前被调用，beforeMount 之前，会找到对应的 template，并编译成 render 函数

### 3、mounted

实例挂载到 DOM 上，此时可以通过 DOM API 获取到 DOM 节点，\$ref 属性可以访问。常用于获取 VNode 信息和操作，ajax 请求。mounted 不会承诺所有的子组件也都一起被挂载。如果你希望等到整个视图都渲染完毕，可以用 vm.\$nextTick

## beforeUpdate 和 updated

当 vue 发现 data 中的数据发生了改变，会触发对应组件的重新渲染，先后调用 beforeUpdate 和 updated 钩子函数。

**beforeUpdate**响应式数据更新时调用，发生在虚拟 DOM 打补丁之前。适合在更新之前访问现有的 DOM，比如手动移除已添加的事件监听器。

**updated**虚拟 DOM 重新渲染和打补丁之后调用，组件 DOM 已经更新，可执行依赖于 DOM 的操作。避免在这个钩子函数中操作数据，可能陷入死循环

## beforeDestroy 和 destroyed

**beforeDestroy**  
钩子函数在实例销毁之前调用。在这一步，实例仍然完全可用。  
beforeDestroy 钩子函数的执行时机是在 $destroy 函数执行最开始的地方，接着执行了一系列的销毁动作，包括从 parent 的 $children 中删掉自身，删除 watcher，当前渲染的 VNode 执行销毁钩子函数等，执行完毕后再调用 destroy 钩子函数。

**destroyed**  
钩子函数在 Vue 实例销毁后调用。调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。  
在 \$destroy 的执行过程中，它又会执行 vm.\_\_patch\_\_(vm.\_vnode, null) 触发它子组件的销毁钩子函数，这样一层层的递归调用，所以 destroy 钩子函数执行顺序是先子后父，和 mounted 过程一样。
