# ajax、fetch、axios 这三都有什么区别？

这三个都是发起 http 请求的东西,都算是对原生 XHR 的封装。从 XHR -> Jquery ajax() -> fetch -> axios  
现在来分别看一下这三个东西

## jQuery ajax

它是对原生 XHR 的封装，支持 JsonP，是 MVC 编程的一种请求方式，已经逐渐的不适应前端的 MVVM 框架的潮流了。  
而且，在项目中使用的时候，要使用 ajax，就得把 jquery 都引进来，很冗余。

```js
$.ajax({
  type: 'POST',
  url: url,
  data: data,
  dataType: dataType,
  success: function() {},
  error: function() {},
});
```

## fetch

fetch 是一个出现比较久的，但是依旧有好多不兼容的 ajax 的替代品。采用了 es 的 promise。  
fetch 不会默认的传 cookies，需要添加配置项 credentials: 'include'  
fetch 只会对网络请求错误报错，400，500 等问题会进入 then，而不会进入 catch。  
fetch 不支持超时的问题。  
fetch 不能监听请求的进度，而 XHR 可以。

```js
try {
  let response = await fetch(url);
  let data = response.json();
  console.log(data);
} catch (e) {
  console.log('Oops, error', e);
}
```

## axios

axios 是一个很高档的东西。它不仅支持浏览器的数据请求，还支持 node 创建 http 请求。  
它和 fetch 一样，也是基于 Promise 的 API，能够从客户端支持防止 CSRF，提供了很多接口，可以并发等。

```js
axios({
  method: 'GET',
  url: url,
})
  .then(res => {
    console.log(res);
  })
  .catch(err => {
    console.log(err);
  });
```
