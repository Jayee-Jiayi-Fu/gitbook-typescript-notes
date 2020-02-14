# 函数相关知识点梳理

## 如何定义一个函数？

1. 通过 function 关键字定义函数

   需要明确指出函数参数的类型，而返回值的类型可以通过 TS 的类型推断省去。

```typescript
function add1(x: number, y: number) {
  return x + y;
}
```

2. 通过变量定义函数类型

```typescript
let add2: (x: number, y: number) => number;
```

3. 通过 type 类型别名定义函数类型

```typescript
type add3 = (x: number, y: number) => number;
```

4. 通过类型接口定义函数类型

```typescript
interface add4 {
  (x: number, y: number): number;
}
```

注意后三种只是函数类型的定义，并没有具体的函数实现。真正需要调用的话，需要书写函数体。

## 函数的参数

### 参数个数

JavaScript 中对函数参数的个数没有限制。TypeScript 中的形参和实参都必须一一对应，多一个或少一个都不行。

```typescript
add1(1);
========
#应有 2 个参数，但获得 1 个。
```

```typescript
add1(1,2,3);
============
#应有 2 个参数，但获得 3 个。
```

### 参数可选

有时候需要函数的参数是可选的，可传可不传。这就用到可选参数。

```typescript
function add5(x: number, y?: number) {
  return y ? x + y : x;
}
add5(1);	// 1
add5(1,2);	// 3
```

必须要注意的是，可选参数必须位于必选参数之后，否则编译器会报错。

```typescript
function add5(x: number, y?: number, z: number) {
  return y ? x + y : x;
}
=================================================
#必选参数不能位于可选参数后。
```

### 参数默认值

可以像 ES6 那样，为参数定义默认值。

```typescript
function add6(x: number, y = 1, z: number, q = 1) {
  return x + y + z + q;
}
```

调用的时候，我们需要注意一下：在必选参数前的默认参数是不能省略的，必须明确地传入 undefined 来获取默认值。但非必选参数前的默认参数，这里是参数 q ，是可以省略的。

```typescript
add6(1, undefined, 1); // 4
```

### 剩余参数

当参数个数不确定的时候，可以使用剩余参数。剩余参数的类型是以数组形式存在。

```typescript
function add7(x: number, ...rest: number[]) {
  return x + rest.reduce((pre, cur) => pre + cur);
}
add7(1, 1, 1);	//3
```

## 函数重载

在静态类型语言中，如C、C++、Java都有函数重载的概念。它的含义是：两个函数，如果他们的函数名相同，参数个数或参数类型不同，那么就实现了函数重载。函数重载的好处是无需为功能相似的函数选用不同的函数名称，这样增强了函数的可读性。

在TypeScript 中，函数重载的定义方式有些不同。它要求我们先定义一个函数名相同但参数不同的声明列表，然后在一个类型最宽泛的版本中，实现函数重载。

```typescript
//声明列表
function add8(...rest: number[]): number;
function add8(...rest: stringp[]): string;
//函数重载
function add8(...rest: any[]): any {
  let first = rest[0];
  if (typeof first === "string") {
    return rest.join("");
  }
  if (typeof first === "number") {
    return rest.reduce((pre, cur) => pre + cur);
  }
}
add8(1, 2, 3);			// 6
add8("a", "b", "c");	// abc
```

TypeScript 编译器在编译重载的时候，会去查找声明列表，并按照列表顺序尝试各个定义。如果匹配的话，就使用该定义，否则继续往下查找。所以应该把最容易匹配的放在最前面。

