
# Typescript

> TypeScript 由微软开发的开源编程语言，它是 JavaScript 的一个超集，而且本质上向这个语言添加了可选的静态类型和基于类的面向对象编程。

> *TypeScript 扩展了 JavaScript 的语法，所以任何现有的 JavaScript 程序可以不加改变的在 TypeScript 下工作。*

# 优势

- 静态类型检查和代码重构、IDE支持更强大，更好的智能语法提示、让代码预测性更高，可控性更高，易于维护和调试。
- 对模块、命名空间和面向对象的支持，更容易组织代码开发大型复杂程序。
- 相对运行时，编译步骤可以提前捕获错误。

# 使用

```ts
// 基本类型
let num:number = 10;

let str:string = 'abc';

let bol:boolean = true;

let undef:undefined = undefined;

let empry:null = null;

// 组合类型
let user:(number|string);

// 任意类型： 尽可能不用！
let data:any;

// 类型转换
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```

```ts
// 数组
let numArr:number[] = [1, 2, 3];
let strArr:string[] = ['a', 'b', 'c'];
let usrArr:(string|number)[] = [1, 'a', 2, 'b', 'c'];
```

```ts
// 函数 - 声明形参和返回值的类型（形参个数要求符合！）
function sum(a:number, b:number):number {
  return a + b;
}

// 函数 - 可选参数（可选参数需定义在必选参数的后面）
function sum(a:number, b?:number, c:number=50):number {
  return b ? (a + b) : a;
}

// 函数 - es6语法接收任意多参数
function sum(...arg: number[]):number {
  return arg.reduce((sum,v) => {
    return sum + v;
  })
}

// 函数 - 无返回值
function log(msg: string):void {
  console.log(msg);
}

// 函数 - 一定报错或无法执行完毕的情况
function error(msg: string):never {
  msg = `some bad thing: ${msg}`;
  throw new Error(msg);
}
```

```ts
// 枚举
enum Color {red = '红', yellow = '黄', blue = '蓝', green = '绿'};
let col:Color = Color.red;
```

```ts
// 字面量对象（不能动态添加属性！！！）
let obj = {
  a: 1,
};
let obj.b = 2; // error
```

```ts
// 类
class Person {
  protected name: string;
  protected constructor(theName: string) { this.name = theName; }
}

class Employee extends Person {
  private department: string;
  constructor(name: string, department: string) {
    super(name);
    this.department = department;
  }
  public getElevatorPitch() {
    return `Hello, my name is ${this.name} and I work in ${this.department}.`;
  }
}

let howard = new Employee("Howard", "Sales");

// 访问修饰符： public（默认）, private, protected
// 只读修饰符： readonly
// 存取器： get/set
// 静态属性： static
// 抽象类： abstract
```

```ts
// 接口： 提前约定和描述类或对象、函数、参数等的构成，不做任何实现

// 使用接口约束字面量对象: 属性类型和个数需严格执行，不多不少！
interface Idata {
  a: number,
  b: string,
};
let obj1: Idata = {a:10, b:'abc'};

// 使用接口约束类: 定义过的属性和方法必须实现！ 可以添加其他属性和方法。
interface Iperson {
  name: string,
  age: number,
  eat(food: string):void,
};
class Chinese implements Iperson { // 可以实现多个接口
  constructor(public name: string, public age: number, public gender: string) {}
  eat(food: string): void {
    console.log('我要吃${food}');
  }
  say(): void {
    console.log(`我是${this.name}`);
  }
};
```

```ts
// 泛型: 数据类型不确定或多变时使用。

// demo1
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;

// demo2
let arr2: Array<string> = [];
arr2.push(2);

// demo3
function createArray<Type>(...arg: Type[]):Type[] {
  return arg;
};
let arr3:number = createArray<number>(10, 20);
let arr4:string = createArray<string>('a', 'b');
```