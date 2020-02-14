# 数据类型

## 类型对比

ES6 和 TypeScript 两者的数据类型对比：

| ES6的数据类型 | TypeScript的数据类型 | TypeScript新增类型 |
| ------------- | :------------------- | ------------------ |
| Boolean       | Boolean              | void               |
| Number        | Number               | any                |
| String        | String               | never              |
| Array         | Array                | 元组               |
| Function      | Function             | 枚举               |
| Object        | Object               | 高级类型           |
| Symbol        | Symbol               |                    |
| undefined     | undefined            |                    |
| null          | null                 |                    |

## 类型注解

作用：相当于强类型语言中的类型声明，可以起到约束变量的作用

语法：(变量/函数)：type

## 基础类型

在上一节工程的 src 目录下新建 datatype.ts 文件，并引入到 index.ts 文件中

```javascript
//in index.ts
import "./datatype"
...
```

在 datatype.ts 文件中我们定义了各种类型的变量，下面进行逐一讲解。

### 原始类型

```typescript
let bool: boolean = true;
let num: number = 123;
let str: string = "abc";
```

如果把 string 赋值给一个 number 类型的变量，编辑器会报错，提示：

```typescript
num = 'abc'
==========
#不能将类型“"abc"”分配给类型“number | null | undefined”。
```

此处就可以很直观地体验  JavaScript 和 TypeScript 的区别，就是 TypeScript 的数据类型是不能够改变的。

### 数组

定义数组类型有两种方式：

```typescript
let arr1: number[] = [1, 2, 3];
let arr2: Array<number> = [1, 2, 3];
```

这里的 Array 是 TypeScript 为我们预定义的一个泛型接口。两种定义方式是等价的，都表示数组的类型只能是 number 类型。

如果为 number 类型的数组添加字符串成员，编辑器会报错：

```typescript
let arr3: number[] = [1, 2, 3, '4'];
==================================
#不能将类型“string”分配给类型“number”。
```

如果想为数组的成员定义不同的数据类型，可以这么做：

```typescript
let arr4: Array<number | string> = [1, 2, 3, "4"];
```

### 元组

元组是一种特殊的数组，它限定了数组元素的类型和个数。

```typescript
let tuple: [number, string] = [1, "a"];
```

如果我们修改了元组的类型和个数，编辑器会报错：

```typescript
let tuple2: [number, string] = ['1', "a"];
==========================================
#不能将类型“string”分配给类型“number”。

let tuple3: [number, string] = [1, "a", "c"];
=============================================
#不能将类型“[number, string, string]”分配给类型“[number, string]”。属性“length”的类型不兼容。不能将类型“3”分配给类型“2”。
```

- **元组的越界问题**

  元组也有 push 方法，我们可以给元组 push 元素，编辑器不会报错。打印元组也确实可以看到新元素。可见 TypeScript 允许我们想元组插入新元素。

  ```typescript
  tuple.push(2);
  console.log(tuple); //(3) [1, "a", 2]
  ```

  但是却不能访问这个新元素，编辑器会报错：

  ```typescript
  console.log(tuple[2])
  =====================
  #Tuple type '[number, string]' of length '2' has no element at index '2'.
  ```

  实际开发过程中，强烈建议不要这样使用。

### 函数

如果我们按照 ES6 方式定义一个带参函数如下，编辑器会报错：

```typescript
let add = (x,y) => x + y;
=========================
#参数“x”隐式具有“any”类型。
#参数“y”隐式具有“any”类型。
```

我们要为函数参数提供类型注解，就没有问题了。此外还可以加上函数返回值的类型，不过通常可以省略，因为利用了 TS 的类型推断功能。

```typescript
let add = (x: number, y: number) => x + y;
let add = (x: number, y: number): number => x + y;
```

下面定义一个函数类型，它没有具体的实现，但已清楚表示可传入两个 number 类型的参数，并返回 一个number 类型值。

```typescript
let computed: (x: number, y: number) => number
```

然后再具体实现该函数，此时编辑器可以为我们自动提示，参数名也无需与定义时相同，也不必定义具体的类型。

```typescript
computed = (a, b) => a + b;
```

### 对象	

此处定义一个 object 类型的对象。在 JS 中，我们可以任意修改对象的属性。但在 TS 中，编辑器会报错：

```typescript
let obj: object = { x: 1, y: 2 };
obj.x = 2
==========
#类型“object”上不存在属性“x”。
```

因为我们只是简单地指定 obj 的类型是 object，并没有具体定义它应该包含哪些属性。正确的定义方式应该是：

```typescript
let obj: { x: number; y: number } = { x: 1, y: 2 };
obj.x = 2; // √
```

### symbol

symbol 表示唯一的标识。

```typescript
let s1: symbol = Symbol();
let s2 = Symbol();
```

### undefined 和 null

我们可以为变量赋值为 undefined 和 null。

```typescript
let un: undefined = undefined;
let nu: null = null;
```

undefined 和 null 类型的变量将不能再赋其他的值。

```typescript
un = 1;
=======
#不能将类型“1”分配给类型“undefined”。
```

反过来，其他类型的变量也不能赋值为 undefined 和 null

```typescript
let num2:number = 1;
num = undefined;
================
#不能将类型“undefined”分配给类型“number”。
num = null;
================
#不能将类型“null”分配给类型“number”。
```

除非我们给变量定义了联合类型：

```typescript
let num3: number | undefined | null = 1;
num3 = undefined;	//√
num3 = null;		// √
```

但是 TS 认为 undefined 和 null 是所有类型的子类型，如果我们希望两者可以自由地赋值给其他类型，则需要修改 tsconfig.json 配置文件。反之如果我们想要使用更为严谨的语法，只要注释掉，并使用联合类型就可以了。

```json
{	"strictNullChecks": false    }
```

### void

在 JS 中，void 是一种操作符，它可以让任何表达式返回 undefined。比如，想得到 undefined 最便捷的办法是：

```javascript
void 0 // 返回 undefined
```

为什么要有这种设置呢？因为在 JS 中，undefined 并不是一个保留字。它可以被赋为其他任何值。我们可以自定义一个 undefined 来覆盖全局的 undefined。

在 TS 中，void 类型表示没有任何返回值的类型。比如没有返回值的函数，它的类型就是 void 类型。

```typescript
let noReturn = () => {};
```

### any

在 TS 中，如果不指定一个变量的类型，那么它默认就是 any 类型。这和 JS 中就没有任何区别了，可以任意给它赋值。如果不是特殊情况，不建议使用 any 类型。

```typescript
let x;
x = 1;
x = "a";
```

### never

never 类型表示永远不会有返回值的类型。比如抛出 error 的函数和死循环函数，它们就永远不会有返回值，所以这两个函数的类型是 never 类型。

```typescript
let error = () => {
  throw new Error("error");
};
let endless = () => {
  while (true) {}
};
```

## 小结

TS 的基本数据类型完全覆盖了 JS 中的类型，并且通过 any 类型实现了对 JS 类型的兼容。为变量或者函数声明类型，是一个非常好的编程习惯，







