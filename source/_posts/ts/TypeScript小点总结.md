---
title: TypeScript 小点总结
date: 2022/11/22 22:12:00
tags: TypeScript 
categories: TypeScript
cover: /img/bg.jpg
---

# TypeScript 小点归纳总结

### 1. @ts-expect-error

```ts
const multiply = (a:number,b:number) => a+b; 
multiply('12',13); 
```

TS 编译器会提示报错：Argument of type 'string' is not assignable to parameter of type你不能修改第一个参数的类型，并且暂时想要忽略 TS 编译器报出的错误，就可以用 @ts-ignore 来抑制错误。

```ts
const multiply = (a:number,b:number) => a*b; 
// @ts-ignore 
multiply('12',13); 
```

当你修复了错误，并将传给 multiply 函数的第一个参数 '12' 改为 12 后：

千万别忘了把我们之前使用的 @ts-ignore 指令删除，因为它会永远忽略下一行，除非你把它删掉，否则将来可能会导致代码出现错误。

如果担心自己忘记删除，也可以用 @ts-expect-error 指令，它与 @ts-ignore 指令类似，不同的是一旦错误被修复，TS 编译器就会提示报错。

```ts
const multiply = (a:number,b:number) => a+b; 
// @ts-expect-error 
multiply(12,13); 
Unused '@ts-expect-error' directive.
```

这样能够提醒你在修复错误后立即删除指令。

---

### 2. never 类型
假设有一个接受错误状态码并根据状态抛出错误的函数，在这种情况下，当 function 不会正常结束时，never 类型就派上用场了。

never 和 void 之间的区别是：void 意味着至少要返回一个 undefined 或 null，而 never 意味着不会正常执行到函数的终点。

```ts
function throwError(error: string): never {  
        throw new Error(error);  
}  
```

---

### 3. 模板文字类型
模板文字类型类似于 javascript 中的字符串类型，但是特定于类型。假设你想实现一个弹出框的库，并且有一个用于定位弹出框的类型：

```ts
type popoverPositions = 'top'|'bottom'|'left'|'right'|'top-left'|'top-right'|'top-center'|'bottom-left'|'bottom-right'|'bottom-center';
```

但是在实际使用中，对这些类型进行排列组合会让你抓狂。

通过使用模板字面量类型，你可以很方便的进行分解并组合类型，这样就可以得倒包含所有可能组合出的新类型了：

```ts
type positions = 'top'|'bottom'|'center'; 
type directions = 'left'|'right'|'center' 
type popoverPositions = positions | directions | `${positions}-${directions}`
```

---

### 4. 空断言
空断言用来告诉 TS 编译器你的值既不是 null 也不是 undefined。假设已经把值初始化为：

```ts
let myNumber:null | number = null; 
```

假设我们有一个只接受 number 类型的函数，

```ts
const add = (a:number,b:number) => { 
    return a + b; 
} 
add(myNumber,1); 
```

这时编译器会报错：Argument of type 'null' is not assignable to parameter of type 'number'.所以在这里，可以在变量末尾使用带有 ! 的空断言，告诉编译器传入的值不为空。

```ts
const add = (a:number,b:number) => { 
    return a + b; 
} 
add(myNumber!,1); 
```

上面的代码能够编译成功。

---

### 5. Record

Record是一个很好用的工具类型。

他会将一个类型的所有属性值都映射到另一个类型上并创造一个新的类型, 先看看·它的源码：

```ts
/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```
好像源码也比较简单，即将K中的每个属性([P in K]),都转为T类型。常用的格式如下：

```ts
type proxyKType = Record<K,T>
```

会将K中的所有属性值都转换为T类型，并将返回的新类型返回给proxyKType，K可以是联合类型、对象、枚举…

```ts
// demo
type petsGroup = 'dog' | 'cat' | 'fish';

interface IPetInfo {
    name:string,
    age:number,
}

type IPets = Record<petsGroup | 'otherAnamial', IPetInfo>;

const animalsInfo:IPets = {
    dog:{
        name:'dogName',
        age:2
    },
    cat:{
        name:'catName',
        age:3
    },
    fish:{
        name:'fishName',
        age:5
    },
    otherAnamial:{
        name:'otherAnamialName',
        age:10
    }
}
```

demo 在 `type IPets = Record<petsGroup | ‘otherAnamial’, IPetInfo>` 中除了petsGroup的值之外，还追加了 'otherAnamial’ 这个值。

##### N1: Record 类型 Vs 索引签名
在 TypeScript 中，我们访问带有方括号的对象的方式称为索引签名。 它广泛用于具有未知字符串键和特定值的对象类型。下面是一个例子：

```TS
type studentScore = { [ name:string]:number }
```

上面的索引签名示例也可以使用 Record 类型表示

```TS
type studentScore = Record<string, number>
```

对于这个用例，从类型断言的角度看，这两种类型的声明的作用是一致的。从语法的角度看，索引签名是更好些，因为其中的name 的键表达式更清晰的表达了它的意图，并在vscode的展示中更智能感知。

所以，为什么我们要去使用Record类型呢？

##### N2: 为什么Record 类型有用？

Record类型的好处是简明的。当我们想要去限制属性时，也就是Record类型大显身手的时候。
下面的示例是我们在Record中使用联合字符串去限制属性键:

```TS
type roles = 'tester' | 'developer' | 'manager'
const staffCount: Record<roles, number> = {
  tester: 10,
  developer: 20,
  manager: 1
}
```

另一个有用的功能是 keys 可以是枚举。在下面的例子中，我们使用 staffTypes 枚举作为 Record 类型的限制值，因此可读性更好。请注意，尽在 TypeScript2.9 之后才支持枚举。因此，在2.9版本之前，key 的类型被限制为 string 类型。

##### N3: Record 类型和 keyof 组合

通过使用keyof从现有类型中获取所有的属性，并和字符串组合，我们可以做如下事情：

```TS
interface Staff {
    name: string,
    salary: number,
}
  
type StaffJson = Record<keyof Staff, string>

const product: StaffJson = {
    name: 'John',
    salary: '3000'
}
```

当你想要保留现有类型的属性但将值类型转换为其他类型时，这很便捷。


Record 是一个有用和简要的工具类型，可以让你的代码更健壮。

--- 

### 6. What