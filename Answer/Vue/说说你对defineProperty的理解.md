# 说说你对 Object.defineProperty 的理解

先来刷一下 MDN
`Object.defineProperty`方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。

## 语法

```js
Object.defineProperty(obj, prop, descriptor);
```

- obj 要在其上定义属性的对象。
- prop 要定义或修改的属性的名称。
- descriptor 将被定义或修改的属性描述符。

## 描述

该方法允许精确添加或修改对象的属性。通过赋值操作添加的普通属性是可枚举的，能够在属性枚举期间呈现出来（for...in 或 Object.keys 方法）， 这些属性的值可以被改变，也可以被删除。这个方法允许修改默认的额外选项（或配置）。默认情况下，使用 Object.defineProperty() 添加的属性值是不可修改的。

## 属性描述符

对象里目前存在的属性描述符有两种主要形式：**数据描述符和存取描述符**。数据描述符是一个具有值的属性，该值可能是可写的，也可能不是可写的。存取描述符是由 getter-setter 函数对描述的属性。**描述符必须是这两种形式之一；不能同时是两者**。

**数据描述符和存取描述符均具有以下可选键值**(默认值是在使用 Object.defineProperty()定义属性的情况下)：

- configurable
  当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除。默认为 false。

- enumerable
  当且仅当该属性的 enumerable 为 true 时，该属性才能够出现在对象的枚举属性中。默认为 false。

### 数据描述符同时具有以下可选键值：

- value
  该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined。

- writable
  当且仅当该属性的 writable 为 true 时，value 才能被赋值运算符改变。默认为 false。

### 存取描述符同时具有以下可选键值：

- get
  一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入 this 对象（由于继承关系，这里的 this 并不一定是定义该属性的对象）。
  默认为 undefined。

- set
  一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。
  默认为 undefined。

### 描述符可同时具有的键值

|            | configurable | enumerable | value | writable | get | set |
| ---------- | ------------ | ---------- | ----- | -------- | --- | --- |
| 数据描述符 | Yes          | Yes        | Yes   | Yes      | No  | No  |
| 存取描述符 | Yes          | Yes        | No    | No       | Yes | Yes |

如果一个描述符不具有 value,writable,get 和 set 任意一个关键字，那么它将被认为是一个数据描述符。如果一个描述符同时有(value 或 writable)和(get 或 set)关键字，将会产生一个异常。

记住，这些选项不一定是自身属性，如果是继承来的也要考虑。为了确认保留这些默认值，你可能要在这之前冻结 Object.prototype，明确指定所有的选项，或者通过 Object.create(null)将**proto**属性指向 null。

## 在 Vue 里，是用在响应式的时候。

Vue 把 data 里的数据，用该方法添加了一个存储依赖的数组，通过 getter 和 setter 来监听数据的改变，在数据改变之后通过 watcher 通知这些依赖
