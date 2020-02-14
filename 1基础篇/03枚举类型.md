# 枚举类型

## 角色判断例子

现有一个角色判断的例子，应用上存在不同的角色，每种角色有不同操作权限及不同的UI界面。那么用户登录的时候我们一般需要做一些初始化的操作，如下面代码所示：

```javascript
function initByRole(role){
    if(role === 1 || role === 2){
        // do sth
    }else if( role === 3 || role == 4){
        // do sth
    }else if( role === 5){
        // do sth
    }else{
        // do sth
    }
}
```

上述代码至少存在两个严重的问题：

1. 可读性差：如果不借助特殊文档，很难一直记住数字的含义
2. 可维护性差：代表角色的数字都被硬编码了，如果某一天代表角色牵一发动全身

怎么解决这个问题？可以使用 TS 的枚举类型。

## 什么是枚举？

枚举：一组**有名字的**常量集合。

![enum](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-typescript-notes/enum.JPG)

### 数字枚举

枚举成员的值默认从0开始，后续成员递增。

```typescript
enum Role {
  Reporter,
  Developer,
  Maintainer
}
console.log(Role.Reporter);		//0
console.log(Role.Developer);	//1
```

也可以指定成员起始值。

```typescript
enum Role2 {
  Reporter = 10,
  Developer,
  Maintainer
}
console.log(Role2.Reporter);	//10
console.log(Role2.Developer);	//11
```

你可能觉得枚举看起来很像一个对象，实际上它编译出来就是一个对象：

```typescript
console.log( Role )
===================
{
    0: "Reporter"
	1: "Developer"
	2: "Maintainer"
	Reporter: 0
	Developer: 1
	Maintainer: 2
}
```

枚举的实现过程如下，枚举成员及成员的值都作为了 key 。这种实现方法叫做**反向映射**。

```javascript
"use strict";
var Role;
(function (Role) {
    Role[Role["Reporter"] = 0] = "Reporter";
    Role[Role["Developer"] = 1] = "Developer";
    Role[Role["Maintainer"] = 2] = "Maintainer";
})(Role || (Role = {}));
```

### 字符串枚举

为枚举成员全都赋值字符串，就是字符串枚举。

```typescript
enum Message {
  Success = "Great!",
  Fail = "Sorry!"
}
```

它的实现过程如下。只有枚举成员作为了 key。也就是说，字符串枚举是不能就进行反向映射的。

```javascript
"use strict";
var Message;
(function (Message) {
    Message["Success"] = "Great!";
    Message["Fail"] = "Sorry!";
})(Message || (Message = {}));
```

### 异构枚举

我们还可以吧数字枚举和字符串枚举混用。

```typescript
enum Answer {
  N,
  Y = "yes"
}
console.log(Answer.N);	// 0
console.log(Answer.Y);	//'yes'
```

它的实现过程如下。但是这用使用容易引起混淆，不太建议使用。

```javascript
"use strict";
var Answer;
(function (Answer) {
    Answer[Answer["N"] = 0] = "N";
    Answer["Y"] = "yes";
})(Answer || (Answer = {}));
```



## 枚举成员的性质

### 修改枚举成员的值

枚举成员是只读类型，如果修改枚举成员的值，编辑器会报错：

```typescript
Role.Reporter = 2;
==================
# Cannot assign to 'Reporter' because it is a read-only property.
```

### 枚举成员的分类

#### 常量枚举成员

常量枚举成员（const member ）有三种情况：

- 使用默认初始值
- 对已有枚举成员的引用
- 常量表达式

常量成员会在编译的时候计算出结果，然后 以常量的形式出现在运行时环境。

####  计算枚举成员

计算枚举成员（computed member）是一些非常量的表达式。这些表达式不会在编译时运行，而会被保留到程序的运行阶段。

```typescript
enum Char {
  a,
  b = Char.a,
  c = 1 + 3,
  d = Math.random(),
  e = "123".length
}
console.log(Char.a);	// 0
console.log(Char.b);	// 0
console.log(Char[0]);	// b
```

它的实现过程如下。

```typescript
"use strict";
var Char;
(function (Char) {
    Char[Char["a"] = 0] = "a";
    Char[Char["b"] = 0] = "b";
    Char[Char["c"] = 4] = "c";
    Char[Char["d"] = Math.random()] = "d";
    Char[Char["e"] = "123".length] = "e";
})(Char || (Char = {}));
```

另外在计算枚举成员后面的成员必须赋初始值，否则编辑器报错：

```typescript
enum Char {
  e = "123".length,
  f
}
=======================
#枚举成员必须具有初始化表达式。
```

### 常量枚举

用 const 声明的枚举就是常量枚举。常量枚举的特性是在编译阶段会被移除，也就是说编译后没有任何代码生成。那么它有什么用呢？当我们不需要一个对象，而需要对象的值的时候就可以使用常量枚举，这样可以减少我们在编译环境的代码。

```typescript
const enum Month{
  Jan,
  Feb,
  Mar
}
let month = [Month.Jan,Month.Feb,Month.Mar]
```

编译后枚举成员被直接替换成了常量。

```javascript
"use strict";
let month = [0 /* Jan */, 1 /* Feb */, 2 /* Mar */];
```

## 枚举类型及枚举成员类型

在某些情况下，枚举和枚举成员都可以作为一种单独的类型存在。我们分数字枚举和字符串枚举两种情况。

### 数字枚举类型

如下我们定义了两个枚举类型 e 和 f，然后可以把任意的 number 类型值赋给 e 和 f 。

```typescript
enum E { a, b }
enum F { a = 0,b = 1 }
let e: E = 3;
let f: F = 3;
console.log(e);		//3
console.log(f);		//3
```

但两种不同类型的枚举是不可以进行比较的，编辑器会报错。因为枚举类型不同，永远都会返回false，没有比较的必要。

```typescript
e === f;
========
#This condition will always return 'false' since the types 'E' and 'F' have no overlap.
```

如果给常量枚举类型赋值非 number 值，编辑器也会报错：

```typescript
let f1: F = "3";
================
#不能将类型“"3"”分配给类型“F”。
```

除了枚举类型变量，同样可以定义枚举成员类型变量:

```typescript
let e1: E.a;
let e2: E.b;
let e3: E.a;
```

尚未赋值的枚举成员类型变量不能用来比较：

```typescript
e1 === e2;
===========
#在赋值前使用了变量“e1”。
#在赋值前使用了变量“e2”。
```

赋值后，不同枚举成员类型的变量也永远不能比较：

```typescript
let e1: E.a = 1;
let e2: E.b = 1;
e1 === e2;
=========
#This condition will always return 'false' since the types 'E.a' and 'E.b' have no overlap.
```

相同枚举成员类型的变量可以比较：

```typescript
let e1: E.a = 1;
let e2: E.a = 1;
e1 === e2;		//true
```

### 字符串枚举类型

字符串枚举类型变量，其取值只能是枚举成员。字符串枚举成员类型变量，其取值只能是自身。

```typescript
enum G {
  a = "apple",
  b = "banana"
}
let g1: G = G.a;
let g2: G.a = G.a;
g1 === g2
```

## 小结

除了学习一种新的概念——枚举之外，我们还需要掌握一种思维方法，就是将程序中不容易记忆的硬编码，或者是在未来中可能会改变的常量抽取出来，定义成枚举类型。这样可以提高程序的可读性和可维护性，枚举可以使你的程序以不变应万变。