# render 函数好处

## 元素动态命名

template 也会被编译成 render，只有一点，template 中元素的 tag_name 是静态的，不可变化，使用 createEelment 可以生成不同 tag_name, 比如 h1 ... h6, 可以通过一个 number 变量控制（官网用例

## 性能优势

render 函数使用的是 JavaScript 的完全编程的能力，在性能上是有一定的优势的

## slot 的另一种实现方式

写在 render 里面也可以实现 solt 的种种功能
