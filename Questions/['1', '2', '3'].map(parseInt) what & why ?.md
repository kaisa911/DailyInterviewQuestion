# ['1', '2', '3'].map(parseInt) what & why ?

**这个题，首先要考虑两个问题：**

1、parseInt 的函数的签名  
2、map 的函数签名,然后再考虑这两个函数在一起使用的时候，会有什么结果

## parseInt

parseInt()函数解析一个字符串参数，并返回一个指定基数的整数 (数学系统的基础)。  
parseInt(param, radix) 相当于 parseInt(String(param).trim(), radix)

```js
const value = parseInt(string[, radix]);
```

- 其中 **string** 是一个要解析的字符串，如果不是字符串，则使用 toString()方法给转换成 string，开头的空格会被忽略。
- **radix** 是一个介于 2-36 的整数。表示一个基数，要将字符串转换成这个基数的进制。默认是 10。
- 返回值是一个整数或者 NaN。

```js
parseInt(100); // 100
parseInt(100, 10); //100
parseInt(100, 2); // 4 因为 100 先转成了'100'，然后在 2 进制下，100 就是 4。
```

**注意：**

在 radix 为 undefined，或者 radix 为 0 或者没有指定的情况下，JavaScript 作如下处理：

- 如果字符串 string 以"0x"或者"0X"开头, 则基数是 16 (16 进制).
- 如果字符串 string 以"0"开头, 基数是 8（八进制）或者 10（十进制），那么具体是哪个基数由实现环境决定。ECMAScript 5 规定使用 10，但是并不是所有的浏览器都遵循这个规定。因此，永远都要明确给出 radix 参数的值。
- 如果字符串 string 以其它任何值开头，则基数是 10 (十进制)。

radix 为 1 或 radix > 36 时，返回值默认为 NaN。

## map

map()方法创建一个新数组，其结果是该数组中的每个元素都调用一个提供的函数后返回的结果。

```js
const new_array = arr.map(function callback(currentValue[,index[, array]]) {
// Return element for new_array
}[, thisArg])
```

可以看到 callback 回调函数需要三个参数, 我们通常只使用第一个参数 (其他两个参数是可选的)。

- currentValue  是 callback 数组中正在处理的当前元素。

- index 可选, 是 callback 数组中正在处理的当前元素的索引。

- array 可选, 是 callback map 方法被调用的数组。

另外还有 thisArg 可选, 执行 callback 函数时使用的 this 值。

```js
const arr = [1, 2, 3];
arr.map(num => num + 1); // [2, 3, 4]
```

了解了这两个函数的签名，我们再来看这个题目：

```js
['1', '2', '3'].map(parseInt);
```

这意味着，每个 map 的迭代，都要给 parseInt 传递两个参数，item 和 index，就是字符串和基数。

那这个表达式可以写成

```js
['1', '2', '3'].map((item, index) => parseInt(item, index));
```

返回的结果应该是：

```js
parseInt('1', 0); // 1
parseInt('2', 1); // NaN
parseInt('3', 2); // NaN 因为 3 不是 2 进制里的数，所以就报错。
```

所以该表达式返回的数组为：

```js
['1', '2', '3'].map(parseInt);
// [1, NaN, NaN]
```

加里·伯恩哈德例子也就很好解释了，这里不再赘述

```js
['10', '10', '10', '10', '10'].map(parseInt);
// [10, NaN, 2, 3, 4]
```
