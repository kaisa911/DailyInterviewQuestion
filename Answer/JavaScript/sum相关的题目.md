# 写一个方法，使得 sum(x)(y)和 sum(x,y)返回的结果相同

```js
var sum = function(x) {
  if (arguments[1]) {
    return x + arguments[1];
  } else {
    return function(y) {
      return x + y;
    };
  }
};
console.log(sum(1)(2)); //3
console.log(sum(1, 2)); //3
```

增加难度： 使得 sum(x)(y)(z)(...)(a)和 sum(x,y,z,... a)返回的结果相同

```js
function sum(x) {
  if (arguments[1]) {
    var arr = Array.prototype.slice.apply(arguments);
    x = arr.reduce((a, c) => a + c);
    return x;
  } else {
    function add(b) {
      x = x + b;
      return add;
    }
    add.toString = function() {
      return x;
    };
    return add;
  }
}
var res1 = sum(1)(2)(3)(4)(5)(6);
var res2 = sum(1, 2, 3, 4, 5, 6);
console.log(res1); //21
console.log(res2); //21
```
