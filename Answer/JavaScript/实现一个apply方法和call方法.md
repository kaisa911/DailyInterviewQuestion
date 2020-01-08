# 实现一个 call 和 apply 方法

## Function.prototype.call()

```
// call 函数的签名
fun.call(thisArg, arg1, arg2, ...)
```

call 函数可以这样理解

允许为不同的对象分配和调用属于一个对象的函数/方法
提供新的 this 值给当前调用的函数/方法。

**参数：**

- thisArg： 在 fun 函数运行时指定的 this 值。
- arg1..：指定的参数列表。

call 函数把某个方法的 this 改成了第一个参数。给该函数传入多个参数，并执行当前函数。  
call 函数有这么几个要点，需要理解：

- this 可以传 null，可以传空，这时候 this 指向 window
- 函数是可以有返回值的！有返回值返回返回值，没有返回值返回 undefined

```js
/*
    	把函数当成对象，然后删除对应的属性值
    */
Function.prototype.call2 = function(context) {
  // 判断当前的this对象是否为null，为空指向window
  var context = context || window;
  // 首先要获取调用call的函数，用this可以获取
  context.fn = this;

  var args = [];
  // 获取call的参数
  for (var i = 1, len = arguments.length; i < len; i++) {
    args.push('arguments[' + i + ']');
  }
  // eval执行该函数
  var result = eval('context.fn(' + args + ')');
  // 删掉调用对象的这个属性
  delete context.fn;
  return result;
};
```

## Function.prototype.apply()

```js
// apply 的函数签名
func.apply(thisArg, [argsArray]);
```

apply 函数和 call 函数实现的是相同的功能

- 允许为不同的对象分配和调用属于一个对象的函数/方法
- 提供新的 this 值给当前调用的函数/方法。

apply 函数的参数：

- thisArg： 在 fun 函数运行时指定的 this 值。
- argsArray：指定的参数数组。

**实现：**

```js
Function.prototype.apply = function(context, arr) {
  // 判断当前的this对象是否为null，为空指向window
  var context = context || window;
  // 首先要获取调用call的函数，用this可以获取
  context.fn = this;

  var result;
  if (!arr) {
    result = context.fn();
  } else {
    var args = [];
    for (var i = 0, len = arr.length; i < len; i++) {
      args.push('arr[' + i + ']');
    }
    result = eval('context.fn(' + args + ')');
  }

  delete context.fn;
  return result;
};
```
