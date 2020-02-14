# 接口

接口是 TS 的一个重要概念。它可以用来约束对象、函数以及类的结构和类型。这是一种代码协作的契约，我们必须遵守，不能改变。

## 对象类的型接口

### 定义和使用

下面我们用 interface 关键字定义两个接口。

```typescript
interface List {
  id: number;
  name: string;
}
interface Result {
  data: List[];
}
```

然后定义一个渲染函数，它接收一个 Result 接口类型的参数，遍历 List 类型数组的 成员 id 和 value。

```typescript
function render(result: Result) {
  result.data.forEach(value => {
    console.log(value.id, value.name);
  });
}
```

最后模拟一个符合接口定义的数据，并执行渲染函数。可以看到结果符合预期。

```typescript
let result = {
  data: [
    { id: 1, name: "A" },
    { id: 2, name: "B" }
  ]
};
render(result);
// 1 "A"
// 2 "B"
```

### 类型检查

#### 传参时的类型检查

如果模拟数据包含约定之外的字段，编译器并不会报错。因为 TS 采用了一种鸭式变形法。你一定听过它的形象描述：一只鸟如果看起来像鸭子，游起来像鸭子，叫起来像鸭子，那么这只鸟就可以被认为是一只鸭子。这是一种动态语言风格。

回到 TS，我们只要传入的对象满足接口的必要条件，即使还有多余属性，也是被允许的。

```typescript
let result2 = {
  data: [
    { id: 1, name: "A", age: 18 },
    { id: 2, name: "B" }
  ]
};
render(result2);
// 1 "A"
// 2 "B"
```

但是，如果我们直接以字面量的形式传入，TS 就会对额外的字段进行类型检查，编译器会报错。

```typescript
render({
  data: [
    { id: 1, name: "A", age: 18 },
    { id: 2, name: "B" }
  ]
});
===================================
#不能将类型“{ id: number; name: string; age: number; }”分配给类型“List”。对象文字可以只指定已知属性，并且“age”不在类型“List”中。
```

#### 绕过类型检查的三种方式

有三种方式：

##### 1.使用变量赋值 

如上面例子那样，把对象字面量赋值给一个变量

##### 2.使用类型断言

类型断言的含义就是要明确地告诉编译器，我们知道对象的类型，这样编译器就会绕过类型检查。

```typescript

render({
  data: [
    { id: 1, name: "A", age: 18 },
    { id: 2, name: "B" }
  ]
} as Result);
```

类型断言的另外一种方式：

```typescript
render(<Result>{
  data: [
    { id: 1, name: "A", age: 18 },
    { id: 2, name: "B" }
  ]
});
```

两种方式是等价的，但更建议使用第一种方式，因为方式二在 React 中会有歧义。

##### 3.使用字符串索引签名

签名格式如下。它的含义是：用任意的字符串去索引 List，可以得到任意的结果。这样 List 就可以支持多个属性了。

```typescript
interface List {
  id: number;
  name: string;
  [x: string]: any;
}
```

### 接口属性

#### 可选属性

假如现在需要判断一下遍历模拟数据时的 value 是否含有新字段（两个接口恢复最开始的定义）。编译器会报错。

```typescript
function render2(result: Result) {
  result.data.forEach(value => {
    if (value.age) {
      console.log(value.age);
    }
  });
}
=====================================
#类型“List”上不存在属性“age”。
```

但是在 List 接口上添加该属性，也还是会报错：

```typescript
interface List {
  id: number;
  name: string;
  age: number;
}
=================
#类型“{ data: { id: number; name: string; }[]; }”的参数不能赋给类型“Result”的参数。
#属性“data”的类型不兼容。不能将类型“{ id: number; name: string; }[]”分配给类型“List[]”。
#Property 'age' is missing in type '{ id: number; name: string; }' but required in type 'List'.
```

此时需要用到可选属性，就能符合预期。

```typescript
interface List {
  id: number;
  name: string;
  age?:number
}
```

#### 只读属性

我们可以给属性加上 readonly，就变成只读属性。

```typescript
interface List {
  readonly id: number;
  name: string;
}
```

如果修改只读属性会报错，也就是说只读属性不能被修改。

```typescript
function render3(result: Result) {
  result.data.forEach(value => {
    value.id++;
  });
}
===================================
#Cannot assign to 'id' because it is a read-only property.
```

### 可索引接口

当不确定一个接口中有多少个属性时，就可以使用前面提到的 **可索引签名**。可索引接口类型既可以用数字索引，也可以用字符串索引。

- **用数字索引的接口**

  它的含义是用任意的数字去索引 StringArray，都可以得到一个字符串。这就相当于声明了一个字符串类型的数组。

  ```typescript
  interface StringArray {
    [index: number]: string;
  }
  let chars:StringArray = ["A","B"]
  ```

- **用字符串索引的接口**

  它的含义是用任意的字符串与索引 Names，都可以得到一个字符串。

  ```typescript
  interface Names {
    [x: string]: string;
  }
  ```

  这样声明之后，就不能再声明一个number类型的成员了。否则编译器会报错。

  ```typescript
  interface Names {
    [x: string]: string;
    [y: number]: number;
  }
  ======================
  #数字索引类型“number”不能赋给字符串索引类型“string”。
  ```

- **混用两种索引签名**

  这样就既可以用字符串索引，也可以用数字索引。需要注意的是，数字索引的返回值类型必须是字符串索引返回值类型所能兼容的类型。因为 JavaScript 会进行类型转换，把 number 转为 string， 以保持兼容性。

  ```typescript
  interface Names {
    [x: string]: string; 	//any 类型也可以
    [z: number]: string;
  }
  ```

## 函数类型的接口

#### 定义和使用

在 [02类型基础](02类型基础.md) 中提到过用变量定义函数的方式：

```typescript
//定义
let minus: (x: number, y: number) => number;

// 函数具体实现
minus = (a, b) => a - b;
```

此外还有两种定义函数类型的方式：函数类型接口及类型别名。类型别名的意思是为函数取一个具体的名字。

```typescript
// 以接口方式定义
interface Minus {
  (x: number, y: number): number;
}
// 以类型别名方式定义
type Minus = (x: number, y: number) => number;

// 函数具体实现
let minus: Minus = (a, b) => a - b;
```

#### 混合类型接口

混合类型接口：表示一个接口既可以定义一个函数，也可以像对象一样拥有属性和方法。下面用混合类型的方法定义一个类库。

```typescript
//混合类型接口
interface Lib {
  (): void; 			//本身是一个函数
  version: string;		//添加属性
  doSomethind(): void;	//添加方法
}
```

我们在具体实现混合类型接口函数时发现，编译器报错。即使我们为其他两个属性赋值了，也依然没有修复。

```typescript
let lib: Lib = () => {};
lib.version = "v.1.0";
lib.doSomethind = () => {};
=======================
Type '() => void' is missing the following properties from type 'Lib': version, doSomethind
```

这时候我们就需要用到类型断言，告诉编译器我们知道这是一个 Lib 类型。

```typescript
let lib: Lib = (() => {}) as Lib;
```

它的 JavaScript 实现是这样子的：

```javascript
"use strict";
let lib = (() => { });
lib.version = "v.1.0";
lib.doSomethind = () => { };
```

