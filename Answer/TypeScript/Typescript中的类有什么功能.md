# Typescript 中的类有什么功能

JavaScript 中在 ES6 中有了类的概念，这个类是原型的语法糖。通过 Class 来实现继承

## 类的相关概念

**类：**

- 类（Class）：定义了一件事物的抽象特点，描述了所创建的对象共同的属性和方法
- 对象（Object）：通过 new 操作符生成的类的实例
- 面向对象(Object-oriented programming): 面向对象是相对于面向过程来讲的，面向对象方法，把相关的数据和方法组织为一个整体来看待，从更高的层次来进行系统建模。其特点包括：封装、继承、多态。

**特点：**

- 封装（Encapsulation）：封装就是信息隐藏，是指利用抽象数据类型将数据和基于数据的操作封装在一起，使其构成一个不可分割的独立实体，数据被保护在抽象数据类型的内部，尽可能地隐藏内部的细节，只保留一些对外接口使之与外部发生联系。
- 继承（Inheritance）：继承是使用已存在的类的定义作为基础建立新类。
- 多态（Polymorphism）：多态是指在父类中定义的属性和方法被子类继承之后，可以具有不同的数据类型或表现出不同的行为，这使得同一个属性或方法在父类及其各个子类中具有不同的含义。
- 抽象类（Abstract Class）：抽象类是供其他类继承的基类，抽象类不允许被实例化。抽象类中的抽象方法必须在子类中被实现

**属性和方法：**

- 构造方法（constructor）：构造器作为一种方法，负责类中成员变量（域）的初始化。
- 存取器（getter & setter）：用来改变属性的读取和赋值行为
- 修饰符（Modifiers）：修饰符是一些关键字，用于限定成员或类型的性质。比如 public、private、protect、readOnly 等
- 接口（Interfaces）：不同类之间公有的属性或方法，可以抽象成一个接口。接口可以被类实现（implements）。一个类只能继承自另一个类，但是可以实现多个接口

## TypeScript 中类的功能

### 1、实现了类的声明和使用

```ts
class Greeter {
	greeting: string;
	constructor(message: string) {
		this.greeting = message;
	}
	greet() {
		return 'Hello, ' + this.greeting;
	}
}

let greeter = new Greeter('world');
```

### 2、实现了修饰符、构造函数、静态属性、存取器等相关的功能

```ts
class Greeter {
	static age: number;
	public greeting: string;
	constructor(message: string) {
		this.greeting = message;
	}
	private name: string = 'john';
	greet() {
		return 'Hello, ' + this.name + this.greeting;
	}
}

class Employee {
	private _fullName: string;

	get fullName(): string {
		return this._fullName;
	}

	set fullName(newName: string) {
		this._fullName = newName;
	}
}
```

### 3、实现了继承

- 使用 extends 来继承父类的。
- 派生类包含了一个构造函数，它必须调用 super()，它会执行基类的构造函数。 而且，在构造函数里访问 this 的属性之前，我们 一定要调用 super()。

```ts
class Animal {
	name: string;
	constructor(theName: string) {
		this.name = theName;
	}
	move(distanceInMeters: number = 0) {
		console.log(`Animal moved ${distanceInMeters}m.`);
	}
}

class Dog extends Animal {
	bark() {
		console.log('Woof! Woof!');
	}
}

class Snake extends Animal {
	constructor(name: string) {
		super(name);
	}
	move(distanceInMeters = 5) {
		console.log('Slithering...');
		super.move(distanceInMeters);
	}
}
```

### 4、实现了抽象类

abstract 用于定义抽象类和其中的抽象方法。

- 抽象类是不允许被实例化
- 抽象类中的抽象方法必须被子类实现

```ts
abstract class Department {
	constructor(public name: string) {}

	printName(): void {
		console.log('Department name: ' + this.name);
	}

	abstract printMeeting(): void; // 必须在派生类中实现
}

class AccountingDepartment extends Department {
	constructor() {
		super('Accounting and Auditing'); // 在派生类的构造函数中必须调用 super()
	}

	printMeeting(): void {
		console.log('The Accounting Department meets each Monday at 10am.');
	}

	generateReports(): void {
		console.log('Generating accounting reports...');
	}
}
```

### 5、把类当做接口使用

类定义会创建两个东西：类的实例类型和一个构造函数。 因为类可以创建出类型，所以能够在允许使用接口的地方使用类。

```ts
class Point {
	x: number;
	y: number;
}

interface Point3d extends Point {
	z: number;
}

let point3d: Point3d = { x: 1, y: 2, z: 3 };
```

### 6、实现了类实现接口（implements）

实现（implements）是面向对象中的一个重要概念。一般来讲，一个类只能继承自另一个类，有时候不同类之间可以有一些共有的特性，这时候就可以把特性提取成接口（interfaces），用 implements 关键字来实现。

```ts
interface ClockInterface {
	currentTime: Date;
	setTime(d: Date);
}

class Clock implements ClockInterface {
	currentTime: Date;
	setTime(d: Date) {
		this.currentTime = d;
	}
	constructor(h: number, m: number) {}
}
```
