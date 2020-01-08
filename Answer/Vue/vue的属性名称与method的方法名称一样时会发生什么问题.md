# vue 的属性名称与 method 的方法名称一样时会发生什么问题？

在 methods 里定义的方法和 data 里的属性名称一样时，
vue 会报 warning

<img src="../../Assets/Images/vueWarn.png" />

在 methods 里定义的方法和 computed 里的属性名一致时，

Vue 会报另一个 warning

<img src="../../Assets/Images/VueWarn2.png" />

并且会提示，同名的方法不是一个 function 的错误
