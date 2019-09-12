## 实现一个 bind 方法

**Function.prototype.bind()**

```js
// bind 函数的函数签名
function.bind(thisArg[, arg1[, arg2[, ...]]])
```

bind()方法创建一个新的函数，在 bind()被调用时，这个新函数的 this 被 bind 的第一个参数指定，其余的参数将作为新函数的参数供调用时使用。

**参数：**

- thisArg 在 fun 函数运行时指定的 this 值。
- arg1..：指定的参数列表。

bind() 函数生成的新函数，不能自己执行，需要手动执行一下。但是这个新函数的 this 永远都是 bind 的第一个参数。  
因为 bind 还有一个特点，就是

> 一个绑定函数也能使用 new 操作符创建对象：这种行为就像把原函数当成构造器。提供的 this 值被忽略，同时调用时的参数被提供给模拟函数。

**_也就是说当 bind 返回的函数作为构造函数的时候，bind 时指定的 this 值会失效，但传入的参数依然生效_**

**实现**

```js
Function.prototype.bind2 = function(context) {
  // 调用 bind 的是否是函数做一个判断
  if (typeof this !== 'function') {
    throw new Error('Function.prototype.bind - what is trying to be bound is not callable');
  }
  // 把调用者和要传给函数的参数保存一下。
  var self = this;
  // 获取 bind2 函数从第二个参数到最后一个参数
  var args = Array.prototype.slice.call(arguments, 1);
  // 创建一个空函数来中转
  var fNOP = function() {};

  var fBound = function() {
    var bindArgs = Array.prototype.slice.call(arguments);
    // 当作为构造函数时，this 指向实例，此时结果为 true，将绑定函数的 this 指向该实例，可以让实例获得来自绑定函数的值
    // 以上面的是 demo 为例，如果改成 `this instanceof fBound ? null : context`，实例只是一个空对象，将 null 改成 this ，实例会具有 habit 属性
    // 当作为普通函数时，this 指向 window，此时结果为 false，将绑定函数的 this 指向 context
    return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
  };
  // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
  fNOP.prototype = this.prototype;
  fBound.prototype = new fNOP();
  return fBound;
};
```
