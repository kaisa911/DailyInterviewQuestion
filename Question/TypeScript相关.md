# TypeScript

1. Typescript 是什么？
2. TypeScript 与 JavaScript 有何不同(区别)？
3. 为什么需要 TypeScript？
4. Typescript 有哪些特性？
5. Typescript 中的内置类型有哪些？
6. 如何编译 Typescript 文件？
7. Typescript 中有哪些变量？ 如何在 Typescript 中创建变量？
8. 如何将多个.ts 文件合并到一个.js 文件中？
9. 是否可以通过.ts 文件中的实时更改自动编译.ts？
10. 接口是什么？ 参考 Typescript 解释它们。
11. 您对 Typescript 中的类有什么了解？列出类的一些功能。
12. Native Javascript 是否支持模块？
13. TypeScript 支持哪些面向对象的术语？
14. 如何从 TypeScript 中的子类调用基类构造函数？
15. 如何在 TypeScript 中实现继承？
16. Typescript 中的模块是什么？
17. 内部模块和外部模块有什么区别？
18. Typescript 中的命名空间是什么？ 如何在 Typescript 中声明一个名称空间？
19. 请解释 Typescript 中的装饰器？
20. 什么是 Mixins？
21. TypeScript 类中属性/方法的默认可见性是什么？
22. TypeScript 如何在函数中支持可选参数，每个参数对于函数都是可选的？
23. TypeScript 是否支持函数重载，因为 JavaScript 不支持函数重载？
24. 是否可以调试任何 TypeScript 文件？
25. 什么是 TypeScript Definition Manager（TSD）？为什么需要它？
26. 什么是 TypeScript 声明关键字？
27. 如何从.ts 文件生成 TypeScript 定义文件？
28. 什么是 tsconfig.json 文件？
29. 请解释一下 TypeScript 中泛型？
30. TypeScript 是否支持所有面向对象的原则？
31. 如何在 TypeScript 中检查 null 和 undefined？
32. 可以在后端使用 TypeScript 吗？ 如果可以，那么应该怎么样做？
33. “interface”与“type”语句有什么区别？
34. TypeScripts 中的环境是什么以及何时使用它们？
35. 什么是 TypeScript Map 文件？
36. TypeScript 中的 Type 断言是什么？
37. TypeScript 中的“as”语法是什么？
38. 什么是 JSX？ 可以在 TypeScript 中使用 JSX 吗？
39. rest 参数是什么？
40. 请解释一下 TypeScript 中枚举？
41. 解释相对与非相对模块导入。
42. 什么是匿名函数？
43. 声明合并是什么？
44. TypeScript 中的方法覆盖是什么？
45. 什么是 Lambda/Arrow 功能？
46. void 和 undefined 有什么区别？
47. 什么是 never 类型？
48. readonly 和 const 有什么区别
49. 什么是 Abstract Class？
50. 什么是 class mixin, 如何实现？
51. typeof 关键词有什么用？
52. keyof 关键词有什么用？
53. 类型声明里 「&」和「|」有什么作用？
54. tsconfig.json 里 --strictNullChecks 参数的作用是什么？
55. declare 关键字有什么用？
56. module 关键字有什么用？
57. namespace 和 module 有什么区别
58. 哪些声明类型既可以当做 type 也可以当做 value？
59. 下面代码会不会报错？怎么解决？

    ```ts
    const obj = { a: 1, b: 'string' };
    obj.c = null;
    ```

60. 下面代码中，foo 的类型应该如何声明

    ```ts
    function foo(a: number) {
     return a + 1;
    }

    foo.bar = 123;
    ```

61. 下面代码中，foo 的类型如何声明

    ```ts
    let foo = {};
    for (let i = 0; i < 100; i++) {
     foo[String(i)] = i;
    }
    ```

62. 实现 MyInterface

    ```ts
    interface MyInterface {
      ...
    }

    class Bar implements MyInterface {
      constructor(public name: string) {}
    }

    class Foo implements MyInterface {
      constructor(public name: string) {}
    }

    function myfn(Klass: MyInterface, name: string) {
      return new Klass(name);
    }
    myfn(Bar);
    myfn(Foo);
    ```
