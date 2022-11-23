---
title: TypeScript 基础篇(一)
date: 2022/11/23 16:14:00
tags: TypeScript 
categories: TypeScript
cover: /img/6.jpg
---

# TypeScript 基础篇（一）

### 1. TS 4 种特殊类型
- never 代表不可达，比如函数抛异常的时候，返回值就是 never。
- void 代表空，可以是 undefined 或 never。
- any 是任意类型，任何类型都可以赋值给它，它也可以赋值给任何类型（除了 never）。
- unknown 是未知类型，任何类型都可以赋值给它，但是它不可以赋值给别的类型。

##### N1: any 和 unknown 的使用场景？
> any 和 unknown 的最大区别是, unknown 是 top type (任何类型都是它的 subtype) , 而 any 即是 top type, 又是 bottom type (它是任何类型的 subtype ) ,这导致 any 基本上就是放弃了任何类型检查.

> 使用 unknown 可以保证类型安全，使用 any 则彻底放弃了类型检查

##### N2: unknown

TypeScript 3.0中引入的 unknown 类型也被认为是 top type ，但它更安全。与 any 一样，所有类型都可以分配给unknown。

```TS
let uncertain: unknown = 'Hello'!;
uncertain = 12;
uncertain = { hello: () => 'Hello!' };
```

我们只能将 unknown 类型的变量赋值给 any 和 unknown。

```TS
let uncertain: unknown = 'Hello'!;
let notSure: any = uncertain;
```

它确实在很多方面不同于 any 类型。如果不缩小类型，就无法对 unknown 类型执行任何操作。

```TS
function getDog() {
 return '22'
}

const dog: unknown = getDog();
dog.hello(); //Object is of type 'unknown'
	缩小类型例子

type obj1 = {
    name: string
}
type obj2 = {
    age: number
}

const isTrue = (tar: obj1 | obj2):tar is obj1 => {
    return (tar as obj1).name !== undefined
}

function handle(tar: obj1 | obj2) {
    return isTrue(tar) ? tar.name : tar.age
    // 如果是 obj1 返回 name 反之返回 obj2
}

let tar1: obj1 = { name: 'a' }
let tar2: obj2 = {age: 2}
handle(tar1)    // 'a'
handle(tar2)    // 2
```

实际开发使用场景：

```TS
let id = user.id as unknown as string	// 后端返回的 user.id 是 number
```

##### N3: any
多使用于接口返回的数据或实在不知道类型的函数参数。

---

### 2. type 和 interface 的区别？

> TS 官方给出的解释: Type aliases and interfaces are very similar, and in many cases you can choose between them freely. Almost all features of an interface are available in type, the key distinction is that a type cannot be re-opened to add new properties vs an interface which is always extendable.
https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces
类型别名和接口非常相似，在很多情况下你可以在它们之间自由选择。几乎所有接口的功能都可以在类型中使用，关键的区别在于，类型不能被重新打开以添加新的属性，而接口则总是可以扩展的..

个人认为主要是用法上的区别，语义上的区别不大。

##### N1: 相同点

都可以描述一个对象或者函数

```TS
<!--interface-->

interface User {
  name: string
  age: number
}

interface SetUser {
  (name: string, age: number): void;
}


<!--type-->

type User = {
  name: string
  age: number
};

type SetUser = (name: string, age: number)=> void;  
```

都允许拓展（extends）

interface 和 type 都可以拓展，并且两者并不是相互独立的，也就是说 interface 可以 extends type, type 也可以 extends interface 。 虽然效果差不多，但是两者语法不同。

```TS
<!--interface extends interface-->

interface Name {
  name: string;
}
interface User extends Name {
  age: number;
}


<!--type extends type-->

type Name = { 
  name: string; 
}
type User = Name & { age: number  };


<!--interface extends type-->

type Name = { 
  name: string; 
}
interface User extends Name { 
  age: number; 
}


<!--type extends interface-->
interface Name { 
  name: string; 
}
type User = Name & { 
  age: number; 
}
```

##### N2: 不同点

type 可以而 interface 不行
- type 可以声明基本类型别名，联合类型，元组等类型

```TS
// 基本类型别名
type Name = string

// 联合类型
interface Dog {
    wong();
}
interface Cat {
    miao();
}

type Pet = Dog | Cat

// 具体定义数组每个位置的类型
type PetList = [Dog, Pet]
```
- type 语句中还可以使用 typeof 获取实例的 类型进行赋值

```TS
// 当你想获取一个变量的类型时，使用 typeof
let div = document.createElement('div');
type B = typeof div
```
- 其它操作

```TS
type StringOrNumber = string | number;  
type Text = string | { text: string };  
type NameLookup = Dictionary<string, Person>;  
type Callback<T> = (data: T) => void;  
type Pair<T> = [T, T];  
type Coordinates = Pair<number>;  
type Tree<T> = T | { left: Tree<T>, right: Tree<T> };
```


interface 可以而 type 不行
- interface 能够声明合并

```TS
interface User {
  name: string
  age: number
}

interface User {
  sex: string
}

/*
User 接口为 {
  name: string
  age: number
  sex: string 
}
*/
```

##### N3: 总结
用 interface 描述 **数据结构**，用 type 描述 **类型关系**

---

### 3. What...
