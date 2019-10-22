# keepalive 实现

## Props：

- include - 字符串或正则表达式。只有名称匹配的组件会被缓存。
- exclude - 字符串或正则表达式。任何名称匹配的组件都不会被缓存。
- max - 数字。最多可以缓存多少组件实例。

## 用法

`<keep-alive>` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。和 `<transition>`相似，`<keep-alive>` 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。

当组件在 `<keep-alive>`内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。

主要用于保留组件状态或避免重新渲染。

```html
<!-- 基本 -->
<keep-alive>
  <component :is="view"></component>
</keep-alive>

<!-- 多个条件判断的子组件 -->
<keep-alive>
  <comp-a v-if="a > 1"></comp-a>
  <comp-b v-else></comp-b>
</keep-alive>

<!-- 和 `<transition>` 一起使用 -->
<transition>
  <keep-alive>
    <component :is="view"></component>
  </keep-alive>
</transition>
```

注意，`<keep-alive>` 是用在其一个直属的子组件被开关的情形。如果你在其中有 v-for 则不会工作。如果有上述的多个条件性的子元素，`<keep-alive>` 要求同时只有一个子元素被渲染。

## 生命钩子

`keep-alive` 提供了两个生命钩子，分别是 activated 与 deactivated。

因为`keep-alive` 会将组件保存在内存中，并不会销毁以及重新创建，所以不会重新调用组件的 created 等方法，需要用 activated 与 deactivated 这两个生命钩子来得知当前组件是否处于活动状态。

## `keep-alive`实现

`keep-alive`是 vue 中的一个抽象组件，被放放置在`src/core/components/keep-alive.js`中。Vue 补单实现了组件化机制，内部实现也是实现了一些组件。

上源码

```js
 name: 'keep-alive',
  abstract: true,

  props: {
    include: patternTypes,
    exclude: patternTypes,
    max: [String, Number]
  },

  created () {
    this.cache = Object.create(null)
    this.keys = []
  },

  destroyed () {
    for (const key in this.cache) {
      pruneCacheEntry(this.cache, key, this.keys)
    }
  },

  mounted () {
    this.$watch('include', val => {
      pruneCache(this, name => matches(val, name))
    })
    this.$watch('exclude', val => {
      pruneCache(this, name => !matches(val, name))
    })
  },

  render () {
    const slot = this.$slots.default
    const vnode: VNode = getFirstComponentChild(slot)
    const componentOptions: ?VNodeComponentOptions = vnode && vnode.componentOptions
    if (componentOptions) {
      // check pattern
      const name: ?string = getComponentName(componentOptions)
      const { include, exclude } = this
      if (
        // not included
        (include && (!name || !matches(include, name))) ||
        // excluded
        (exclude && name && matches(exclude, name))
      ) {
        return vnode
      }

      const { cache, keys } = this
      const key: ?string = vnode.key == null
        // same constructor may get registered as different local components
        // so cid alone is not enough (#3269)
        ? componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}` : '')
        : vnode.key
      if (cache[key]) {
        vnode.componentInstance = cache[key].componentInstance
        // make current key freshest
        remove(keys, key)
        keys.push(key)
      } else {
        cache[key] = vnode
        keys.push(key)
        // prune oldest entry
        if (this.max && keys.length > parseInt(this.max)) {
          pruneCacheEntry(cache, keys[0], keys, this._vnode)
        }
      }

      vnode.data.keepAlive = true
    }
    return vnode || (slot && slot[0])
  }
```

组件是由对象来实现的.同时设置属性`abstract:true`来表明这个是个抽象组件.
使其在组件实例建立父子关系的时候会被忽略.在 initLifecycle 的过程中

```js
// locate first non-abstract parent
let parent = options.parent;
if (parent && !options.abstract) {
  while (parent.$options.abstract && parent.$parent) {
    parent = parent.$parent;
  }
  parent.$children.push(vm);
}
vm.$parent = parent;
```

在`created`的钩子里定义了`this.cache`和`this.keys`,本质上就是用其去缓存创建过的 vnode,利用传入的`props`,include 和 exclude 去控制缓存哪写组件,用 max 去控制缓存的 vnode 最大数量.
同时在`mounted`内使用了`$watch`的表达式去实时监听`include`和`exculde`两个参数.
其模板的实现,没有使用常规的`template`模板,而是实现了`render`函数.执行其组件渲染的时候就直接执行其`render`函数.
这里说一下即使是常规的模板在变化成 vnode 前仍然是需要编译成 render 函数的.

在这里分析一下这个`render`函数的实现

`vm.$slots`
类型:{ [name: string]: ?Array<VNode> }
作用:用来访问被插槽分发的内容。每个具名插槽 有其相应的属性 (例如：v-slot:foo 中的内容将会在 vm.\$slots.foo 中被找到)。default 属性包括了所有没有被包含在具名插槽中的节点，或 v-slot:default 的内容。

首先获取一下第一个子元素的`VNode`

```js
const slot = this.$slots.default;
const vnode: VNode = getFirstComponentChild(slot);
```

这里是在`<keep-alive>`标签内部写 DOM,所以可以先获取一哈他的默认插槽,再获取他的第一个子节点.`<keep-alive>`只处理第一个子元素,所以一般搭配使用的有`component`动态组件和`router-view`.不与`v-for`的组件搭配.

然后通过正则去判断了组件名和`include\exculde`的关系.此时想起来之前写组件的时候写上组件名就有必要了.
要不这个就会有问题.

```js
// check pattern
const name: ?string = getComponentName(componentOptions);
const { include, exclude } = this;
if (
  // not included
  (include && (!name || !matches(include, name))) ||
  // excluded
  (exclude && name && matches(exclude, name))
) {
  return vnode;
}

function matches(
  pattern: string | RegExp | Array<string>,
  name: string
): boolean {
  if (Array.isArray(pattern)) {
    return pattern.indexOf(name) > -1;
  } else if (typeof pattern === 'string') {
    return pattern.split(',').indexOf(name) > -1;
  } else if (isRegExp(pattern)) {
    return pattern.test(name);
  }
  /* istanbul ignore next */
  return false;
}
```

接下来就是检测当前的组件名是否能对上其 props 传入的`include,exclude`其中分成了数组,字符串,正则表达式的情况,`include,exclude`其中任意一种情况被匹配上就返回 vnode,如果没匹配上就走下一步缓存组件

```js
const { cache, keys } = this;
const key: ?string =
  vnode.key == null
    ? // same constructor may get registered as different local components
      // so cid alone is not enough (#3269)
      componentOptions.Ctor.cid +
      (componentOptions.tag ? `::${componentOptions.tag}` : '')
    : vnode.key;
if (cache[key]) {
  vnode.componentInstance = cache[key].componentInstance;
  // make current key freshest
  remove(keys, key);
  keys.push(key);
} else {
  cache[key] = vnode;
  keys.push(key);
  // prune oldest entry
  if (this.max && keys.length > parseInt(this.max)) {
    pruneCacheEntry(cache, keys[0], keys, this._vnode);
  }
}
```

如果命中缓存，则直接从缓存中拿 vnode 的组件实例，并且把这个 key 放在了最后一个;没命中缓存把 vnode 设置进缓存，最后还有处理，如果配置了 max 并且缓存的长度超过了 this.max，还要从缓存中删除第一个
keys 类似一个队列进行 key 的管理

```js
// 删除特定cache
function pruneCacheEntry(
  cache: VNodeCache,
  key: string,
  keys: Array<string>,
  current?: VNode
) {
  const cached = cache[key];
  if (cached && (!current || cached.tag !== current.tag)) {
    cached.componentInstance.$destroy();
  }
  cache[key] = null;
  remove(keys, key);
}
```

除了从缓存中删除外，还要判断如果要删除的缓存并的组件`tag`不是当前渲染组件`tag`，也执行删除缓存的组件实例的 `$destroy` 方法。`render`

最后设置 `vnode.data.keepAlive = true`

加上之前使用了`watch`监听`include`和`exclude`

```js
// 对cache做遍历删除,没有跟上约定条件变化的
function pruneCache(keepAliveInstance: any, filter: Function) {
  const { cache, keys, _vnode } = keepAliveInstance;
  for (const key in cache) {
    const cachedNode: ?VNode = cache[key];
    if (cachedNode) {
      const name: ?string = getComponentName(cachedNode.componentOptions);
      if (name && !filter(name)) {
        pruneCacheEntry(cache, key, keys, _vnode);
      }
    }
  }
}
```

## 组件渲染

### 首次渲染

Vue 的渲染最后都会到 patch 过程作为一个补丁对象打回到 dom 上,在过程中会使用`createComponent`方法
它定义在`src/core/vdom/patch.js`中

```js
function createComponent(vnode, insertedVnodeQueue, parentElm, refElm) {
  let i = vnode.data;
  if (isDef(i)) {
    const isReactivated = isDef(vnode.componentInstance) && i.keepAlive;
    if (isDef((i = i.hook)) && isDef((i = i.init))) {
      i(vnode, false /* hydrating */);
    }
    // after calling the init hook, if the vnode is a child component
    // it should've created a child instance and mounted it. the child
    // component also has set the placeholder vnode's elm.
    // in that case we can just return the element and be done.
    if (isDef(vnode.componentInstance)) {
      initComponent(vnode, insertedVnodeQueue);
      insert(parentElm, vnode.elm, refElm);
      if (isTrue(isReactivated)) {
        reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm);
      }
      return true;
    }
  }
}
```

这个函数里定义了`isReactivated`,第一次渲染时,componentInstance 为`undefined`,`keepAlive`为 true,因为其父组件的`render`会先执行.那么`vnodse`缓存至内存中,并且设置`keepAlive`为 true,因此`isReactivated`为 false.那么走正常的`init`钩子函数执行组件的`mount`.当其执行完`patch`后,执行`initComponent`函数:

```js
function initComponent(vnode, insertedVnodeQueue) {
  if (isDef(vnode.data.pendingInsert)) {
    insertedVnodeQueue.push.apply(insertedVnodeQueue, vnode.data.pendingInsert);
    vnode.data.pendingInsert = null;
  }
  vnode.elm = vnode.componentInstance.$el;
  if (isPatchable(vnode)) {
    invokeCreateHooks(vnode, insertedVnodeQueue);
    setScope(vnode);
  } else {
    // empty component root.
    // skip all element-related modules except for ref (#3455)
    registerRef(vnode);
    // make sure to invoke the insert hook
    insertedVnodeQueue.push(vnode);
  }
}
```

这里会有 `vnode.elm` 缓存了 vnode 创建生成的 DOM 节点。所以对于首次渲染而言，除了在 `<keep-alive>` 中建立缓存，和普通组件渲染没什么区别。
初始化渲染组件,以及首次切换渲染另一个组件,都属于首次渲染.

## 缓存渲染

当我们从 `B` 组件再次点击 `switch`切换到`A`组件，就会命中缓存渲染。

当数据发送变化，在 `patch` 的过程中会执行 `patchVnode` 的逻辑，它会对比新旧 `vnode` 节点，甚至对比它们的子节点去做更新逻辑，但是对于组件 `vnode` 而言，是没有 `children`的，那么对于`<keep-alive>`组件而言，如何更新它包裹的内容呢？

原来 `patchVnode` 在做各种 diff 之前，会先执行 `prepatch` 的钩子函数，它的定义在 `src/core/vdom/create-component` 中：

```js
  prepatch (oldVnode: MountedComponentVNode, vnode: MountedComponentVNode) {
    const options = vnode.componentOptions
    const child = vnode.componentInstance = oldVnode.componentInstance
    updateChildComponent(
      child,
      options.propsData, // updated props
      options.listeners, // updated listeners
      vnode, // new parent vnode
      options.children // new children
    )
  },
```

其核心就是执行`updateChildComponent`方法,这个定义在`src/core/instance/lifecycle.js`中

```js
export function updateChildComponent(
  vm: Component,
  propsData: ?Object,
  listeners: ?Object,
  parentVnode: MountedComponentVNode,
  renderChildren: ?Array<VNode>
) {
  const hasChildren = !!(
    renderChildren ||
    vm.$options._renderChildren ||
    parentVnode.data.scopedSlots ||
    vm.$scopedSlots !== emptyObject
  );

  // ...
  if (hasChildren) {
    vm.$slots = resolveSlots(renderChildren, parentVnode.context);
    vm.$forceUpdate();
  }
}
```

`updateChildComponent`顾名思义，就是去更新其件实例的一些属性.
这里续关注一下`slot`的部分,用于`<keep-alive>`组件本质上支持了`slot`,所以它执行`prepatch`的时候,需要对自己的`children`,即 slot 做重新解析,并触发其`<keep-alive>`组件的`$forceUpdate`,即重新执行`render`方法,如果它包含的第一个`vnode`命中缓存,则直接返回缓存中的组件实例.接着执行`patch`再次执行到`createComponent`方法

```js
function createComponent(vnode, insertedVnodeQueue, parentElm, refElm) {
  let i = vnode.data;
  if (isDef(i)) {
    const isReactivated = isDef(vnode.componentInstance) && i.keepAlive;
    if (isDef((i = i.hook)) && isDef((i = i.init))) {
      i(vnode, false /* hydrating */);
    }
    // after calling the init hook, if the vnode is a child component
    // it should've created a child instance and mounted it. the child
    // component also has set the placeholder vnode's elm.
    // in that case we can just return the element and be done.
    if (isDef(vnode.componentInstance)) {
      initComponent(vnode, insertedVnodeQueue);
      insert(parentElm, vnode.elm, refElm);
      if (isTrue(isReactivated)) {
        reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm);
      }
      return true;
    }
  }
}
```

这时候由于已经缓存过了故`isReactivated`为 true,并且在`init`钩子函数中不会再执行`mount`过程

```js
 init (vnode: VNodeWithData, hydrating: boolean): ?boolean {
    if (
      vnode.componentInstance &&
      !vnode.componentInstance._isDestroyed &&
      vnode.data.keepAlive
    ) {
      // kept-alive components, treat as a patch
      const mountedNode: any = vnode // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode)
    } else {
      const child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      )
      child.$mount(hydrating ? vnode.elm : undefined, hydrating)
    }
  },
```

这也是被`<keep-alive>`包裹的组件在有缓存时,不会执行`create,mounted`等钩子函数的原因.
在上面当`isReactivated`为 true,`reactivateComponent`方法得以执行

```js
function reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm) {
  let i;
  // hack for #4339: a reactivated component with inner transition
  // does not trigger because the inner node's created hooks are not called
  // again. It's not ideal to involve module-specific logic in here but
  // there doesn't seem to be a better way to do it.
  let innerNode = vnode;
  while (innerNode.componentInstance) {
    innerNode = innerNode.componentInstance._vnode;
    if (isDef((i = innerNode.data)) && isDef((i = i.transition))) {
      for (i = 0; i < cbs.activate.length; ++i) {
        cbs.activate[i](emptyNode, innerNode);
      }
      insertedVnodeQueue.push(innerNode);
      break;
    }
  }
  // unlike a newly created component,
  // a reactivated keep-alive component doesn't insert itself
  insert(parentElm, vnode.elm, refElm);
}
```

前面是解决`reactived`组件`transition`动画不触发问题.
最后通过执行`insert(parentElm, vnode.elm, refElm);`将缓存的 DOM 插入目标元素中,完成其在数据更新下的渲染过程

## 生命周期

之前说了,组件被`<keep-alive>`缓存,就不会执行`create,mounted`等钩子函数.很多情况我们需要用到钩子函数,就预制了`activated` 钩子函数,在包裹组件渲染的最后时候执行
`invokeInsertHook`,又去执行`create-component`中的钩子函数

```js
function invokeInsertHook(vnode, queue, initial) {
  // delay insert hooks for component root nodes, invoke them after the
  // element is really inserted
  if (isTrue(initial) && isDef(vnode.parent)) {
    vnode.parent.data.pendingInsert = queue;
  } else {
    for (let i = 0; i < queue.length; ++i) {
      queue[i].data.hook.insert(queue[i]);
    }
  }
}
```

```js
insert (vnode: MountedComponentVNode) {
    const { context, componentInstance } = vnode
    if (!componentInstance._isMounted) {
      componentInstance._isMounted = true
      callHook(componentInstance, 'mounted')
    }
    if (vnode.data.keepAlive) {
      if (context._isMounted) {
        // vue-router#1212
        // During updates, a kept-alive component's child components may
        // change, so directly walking the tree here may call activated hooks
        // on incorrect children. Instead we push them into a queue which will
        // be processed after the whole patch process ended.
        queueActivatedComponent(componentInstance)
      } else {
        activateChildComponent(componentInstance, true /* direct */)
      }
    }
  },
```

这里一个判断很好理解,即`<keep-alive>`包裹的组件是否已经`mounted`过.
先看看非`mounted`情况
`src/core/instance/lifecycle.js`中

```js
export function activateChildComponent(vm: Component, direct?: boolean) {
  if (direct) {
    vm._directInactive = false;
    if (isInInactiveTree(vm)) {
      return;
    }
  } else if (vm._directInactive) {
    return;
  }
  if (vm._inactive || vm._inactive === null) {
    vm._inactive = false;
    for (let i = 0; i < vm.$children.length; i++) {
      activateChildComponent(vm.$children[i]);
    }
    callHook(vm, 'activated');
  }
}
```

很简单的执行`acitvated`钩子函数,并递归去执行其子组件的`acitvated`钩子函数

再看`queueActivatedComponent`的逻辑,在`src/core/observer/scheduler.js`中

```js
export function queueActivatedComponent(vm: Component) {
  // setting _inactive to false here so that a render function can
  // rely on checking whether it's in an inactive tree (e.g. router-view)
  vm._inactive = false;
  activatedChildren.push(vm);
}
```

很简单将传入的实例添加到`activatedChildren`数组中
,等渲染完毕,在`nextTick`后执行`flushSchedulerQueue`

```js
function flushSchedulerQueue() {
  // ...
  const activatedQueue = activatedChildren.slice();
  callActivatedHooks(activatedQueue);
  // ...
}

function callActivatedHooks(queue) {
  for (let i = 0; i < queue.length; i++) {
    queue[i]._inactive = true;
    activateChildComponent(queue[i], true);
  }
}
```

遍历`activatedChildren`
通过队列的方式,去一个个执行`activated`

同样还有`deactivated`钩子函数,它发生在 vnomde 中的
`destroy`钩子函数

```js
 destroy (vnode: MountedComponentVNode) {
    const { componentInstance } = vnode
    if (!componentInstance._isDestroyed) {
      if (!vnode.data.keepAlive) {
        componentInstance.$destroy()
      } else {
        deactivateChildComponent(componentInstance, true /* direct */)
      }
    }
  }
```

和`activateChildComponent`函数类似,递归执行`deactivateChildComponent`

```js
export function deactivateChildComponent(vm: Component, direct?: boolean) {
  if (direct) {
    vm._directInactive = true;
    if (isInInactiveTree(vm)) {
      return;
    }
  }
  if (!vm._inactive) {
    vm._inactive = true;
    for (let i = 0; i < vm.$children.length; i++) {
      deactivateChildComponent(vm.$children[i]);
    }
    callHook(vm, 'deactivated');
  }
}
```

## 总结

`<keep-alive>` 组件是一个抽象组件，它的 `props`有 `include` , `exclude` 和 `max`.它的实现通过自定义 `render` 函数并且利用了插槽，并且知道了 `<keep-alive>` 缓存 `vnode`，了解组件包裹的子元素——也就是插槽是如何做更新的。且在 `patch` 过程中对于已缓存的组件不会执行 `mounted`，所以不会有一般的组件的生命周期函数,但是又提供了 `activated`和`deactivated` 钩子函数。
