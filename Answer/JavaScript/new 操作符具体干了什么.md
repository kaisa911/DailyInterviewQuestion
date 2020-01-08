# new 操作符具体干了什么呢?

## 定义

> new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象类型之一

在创建一个新的实例的时候，必须使用 new 操作符。而使用 new 操作符会经历以下 4 个步骤：

1. 创建一个新对象
2. 将构造函数的作用域赋值给新对象（this 指向新对象）
3. 执行构造函数中的代码（给新对象添加熟悉）
4. 返回新对象

## 模拟 new 操作符的方法：

### new 实现了什么功能：

1. 访问到了构造函数里的属性
2. 访问到了构造函数 prototype 里的属性

我们定义一个 mockNew 来模拟一下 new 的效果

因为 new 返回一个新的对象，所以 mockNew 也要新建一个对象，然后返回这个对象

```js
function mockNew() {
  var obj = new Object();

  return obj;
}
```

这个这个对象能访问到构造函数和构造函数 prototype 里的属性，所以我们也要把构造函数当参数传入。其他的参数放在 rest 里，要让 obj 能访问到构造函数里的属性，可以使用 apply 实现

```js
function mockNew(constructor, ...rest) {
  const obj = new Object();
  // 把构造函数prototype上的属性给obj的__proto__方法
  obj.__proto__ = constructor.prototype;
  // 让this函数指向obj
  constructor.apply(obj, rest);

  return obj;
}
```

还要考虑以下，如果构造函数里有返回值的问题。

- 如果返回值是对象，则返回构造函数里的对象
- 如果不是对象，就返回 obj

```js
function mockNew(constructor, ...rest) {
  const obj = new Object();
  // 把构造函数prototype上的属性给obj的__proto__方法
  obj.__proto__ = constructor.prototype;
  // 让this函数指向obj
  const ret = constructor.apply(obj, rest);

  return typeof ret === 'object' ? ret : obj;

  return obj;
}
```

验证以下：

```js
function Otaku(name, age) {
  this.name = name;
  this.age = age;

  this.habit = 'Games';
}

Otaku.prototype.strength = 60;

Otaku.prototype.sayYourName = function() {
  console.log('I am ' + this.name);
};

function mockNew(constructor, ...rest) {
  const obj = new Object();
  obj.__proto__ = constructor.prototype;
  const ret = constructor.apply(obj, rest);
  return typeof ret === 'object' ? ret : obj;

  return obj;
}

var person = mockNew(Otaku, 'Kevin', '18');

console.log(person.name); // Kevin
console.log(person.habit); // Games
console.log(person.strength); // 60

person.sayYourName(); // I am Kevin
```

ok 搞定～

## 参考文档

1. [new 操作符到底干了什么？](https://juejin.im/post/5b4af862e51d4519984118d1)
2. [JavaScript 温故而知新——new 操作符的实现](https://juejin.im/post/5d2c3e466fb9a07ecd3d8dbf)
3. [JavaScript 深入之 new 的模拟实现](https://github.com/mqyqingfeng/Blog/issues/13)
