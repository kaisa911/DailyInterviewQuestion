# TypeScript 中的内置类型

刚入门 ts，看到好多内置类型，就想整理一下，方便以后在使用中能运用到。

## Partial

- 描述：将类型 T 的所有属性标记为可选属性
- 实现：

  ```ts
  type Partial<T> = {
  	[P in keyof T]?: T[P];
  };
  ```

- 用法：`Partial<T>`

partial 这个类型，就是将某个类型里的属性添加上`？`，有了这个 modifier 之后，很多属性就可以是 undefined 了。

例如：

```ts
// 自定义的用户类型
interface Person {
 name: string;
 gender: 0 ｜ 1; // 0：女， 1：男
 age: number;
}

const userList: Person[] = [];

// 当我们不需要用到所有属性进行过滤时，此时可以定义
const model: Partial<Person> = {
 name: '',
 gender: undefined,
};
```

## Required

- 描述： Required 将类型 T 的所有属性标记为必选属性
- 实现：

  ```ts
  type Required<T> = {
  	[P in keyof T]-?: T[P];
  };
  ```

- 用法：`Required<T>`

这个类型跟 Partial 相反，Partial 是将所有属性变成可选，Required 则是将所有属性改成必须。其中 -? 是代表移除 ?。

## Readonly

- 描述：将所有属性标记为 readonly, 即不能修改
- 实现：

  ```ts
  type Readonly<T> = {
  	readonly [P in keyof T]: T[P];
  };
  ```

- 用法: `Readonly<T>`

## Pick

- 描述：从某个类型中过滤出某个或某些属性
- 实现：

  ```ts
  type Pick<T, K extends keyof T> = {
  	[P in K]: T[P];
  };
  ```

- 用法：`Pick<T, K>`

这个类型则可以将某个类型中的子属性挑出来，比如上面的那个 Person 属性，我们想把 name 和 age 挑出来

```ts
type otherPerson = Pick<Person, 'name' | 'age'>;
/*
{
  name: string
  age: number
}
*/
```

## Record

- 描述：标记对象的属性（key）或者属性值（value）的类型
- 实现：

  ```ts
  type Record<K extends keyof any, T> = {
  	[P in K]: T;
  };
  ```

- 用法：`Record<K, T>`

可以获得根据 K 中的所有可能值来设置 key 以及 value 的类型

```ts
// 设置key和value的类型
const userMap = Record<number, Person> = {
  1: {
    name: '张三',
    gender: 0,
    age: 12;
  }
};

// 设置value的类型
const info = Record<'name' | 'id', string> = {
  name: '张三',
  id: '1',
}
```

## Exclude

- 描述：移除特定类型中的某个（些）属性
- 实现：

  ```ts
  type Exclude<T, U> = T extends U ? never : T;
  ```

- 用法：`Exclude<T, U>`

比如

```ts
type Type1 = Exclude<'a' | 'b' | 'c' | 'd', 'a' | 'c' | 'f'>; // "b" | "d"
```

Exclude 和 Pick 一起用可以重写某些属性的类型，

```ts
type FilterPick<T, U extends keyof T> = Pick<T, Exclude<keyof T, U>>;

interface newPerson extends FilterPick<Person, 'name'> {
	name: number;
}
```

## Extract<T, U>

- 描述：Exclude 的反操作，取 T，U 两者的交集属性
- 实现：

  ```ts
  type Extract<T, U> = T extends U ? T : never;
  ```

- 用法：`Extract<T, U>`

```ts
type A = Extract<'a' | 'b' | 'c' | 'd', 'b' | 'c' | 'e'>; // 'b'|'c'
```

## NonNullable

- 描述： 排除类型 T 的 null | undefined 属性
- 实现：

  ```ts
  type NonNullable<T> = T extends null | undefined ? never : T;
  ```

- 用法：`NonNullable<T>`

## ReturnType

- 描述： 获取函数类型 T 的返回类型
- 实现：

  ```ts
  type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
  ```

- 用法：`ReturnType<T>`

这个类型也非常好用，可以获取方法的返回类型，使用了 Conditional Types ，推论 ( infer ) 泛型 T 的返回类型 R 来拿到方法的返回类型。
