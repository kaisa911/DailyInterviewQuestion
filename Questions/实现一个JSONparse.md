JSON.parse(text[, reviver]) 

用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。提供可选的reviver函数用以在返回之前对所得到的对象执行变换(操作)。 

 第一种：直接调用 eval 


```javascript
function jsonParse (opt) { 

return eval('(' + opt + ')') 

} 

jsonParse(jsonStringify({ x: 5 })) 

// Object { x: 5} 

jsonParse(jsonStringify([1, 'false', false])) 

// [1, "false", falsr] 

jsonParse(jsonStringify({ b: undefined })) 

// Object { b: "undefined"} 
```
 

避免在不必要的情况下使用 eval，eval() 是一个危险的函数， 他执行的代码拥有着执行者的权利。如果你用 eval()运行的字符串代码被恶意方操控修改，最终可能会在你的网页/扩展程序的权限下，在用户计算机上运行恶意代码。 

它会执行JS代码，有XSS漏洞。 

如果你只想记这个方法，就得对参数json做校验。 

```javascript
var rx_one = /^[\],:{}\s]*$/; 
var rx_two = /\\(?:["\\\/bfnrt]|u[0-9a-fA-F]{4})/g; 
var rx_three = /"[^"\\\n\r]*"|true|false|null|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?/g; 
var rx_four = /(?:^|:|,)(?:\s*\[)+/g; 
if ( 
    rx_one.test( 
        json 
            .replace(rx_two, "@") 
            .replace(rx_three, "]") 
            .replace(rx_four, "") 
    ) 
) { 
    var obj = eval("(" +json + ")"); 
} 
```

3.2 第二种：Function 

核心：Function与eval有相同的字符串参数特性。 
```javascript
var func = new Function(arg1, arg2, ..., functionBody); 
```
在转换JSON的实际应用中，只需要这么做。 
```javascript
var jsonStr = '{ "age": 20, "name": "jack" }' 
var json = (new Function('return ' + jsonStr))(); 
```

eval 与 Function 都有着动态编译js代码的作用，但是在实际的编程中并不推荐使用。 