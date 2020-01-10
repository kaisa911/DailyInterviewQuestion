# 节流函数

函数节流的核心是，让一个函数不要执行得太频繁，减少一些过快的调用来节流。

```js
const throttle = (func, delay) => {
  let last;
  let deferTimer;
  return (...args) => {
    const now = +new Date();
    if (last && now < last + delay) {
      clearTimeout(deferTimer);
      deferTimer = setTimeout(() => {
        last = now;
        func.apply(this, args);
      }, delay);
    } else {
      last = now;
      func.apply(this, args);
    }
  };
};

export default throttle;
```
