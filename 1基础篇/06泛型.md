# 泛型

## 什么是泛型？

很多时候我们希望一个函数或者一个类可以支持多种数据类型，提高灵活性。下面有一个例子：

```typescript
function log(value: string): string {
    console.log(value)
    return value;
}
```

如果希望 log 函数能够接收一个字符串数组，该怎么实现呢？结合之前的内容，我们会想到函数重载：

```typescript
function log(value:string):string;
function log(value:string[]):string[];
function log(value:any){
    console.log(value)
    return value;
}
```

或者联合类型：

```typescript
function log(value: string | string[]): string | string[]{
    console.log(value)
    return value;
}
```

现在我们希望更进一步，log 函数能够接收任何类型的参数，我们很自然地想到 any 类型：

```typescript
function log(value: any) {
    console.log(value)
    return value;
}
```

到此为止，这个函数似乎满足了我们的所有需求。但是这产生了另外一个问题，就是 any 类型丢失了一些信息，也就是类型之间的约束关系，它忽略了参数类型和返回值类型必须是一致的。当调用者看到 any 的时候，他无法获知这种约束关系。这时候，就需要用到 **泛型 **了。

- 那么什么时泛型呢？

>  泛型：不预先确定的数据类型，具体的类型在使用的时候才能确定。

## 泛型函数

现在用泛型改造上面的函数。

```typescript
function log<T>(value: T): T {
  return value;
}
```

改造之后，类型 T 不需要预先指定，就相当于 any 类型。另一方面可以保证输入参数和返回值的类型是一致的。定义泛型函数后，有两种方式调用泛型函数：

1. 直接指明泛型函数的类型，那么输入参数就必须和该类型相同

   ```typescript
   log<string[]>(["a", "b", "c"]);
   ```

2. 利用 TS 的类型推断，省略泛型类型参数

   ```typescript
   log(["a", "b", "c"])
   ```

我们不仅可以使用泛型定义一个函数，还可以定义一个类型。例如定义一个泛型函数类型:

```typescript
type Log = <T>(value: T) => T;

//泛型函数的具体实现
let myLog: Log = value => value;
```

## 泛型接口

泛型同样可以用在接口中。我们依然使用上面的例子。它和类型别名的方式是完全等价的。

```typescript
interface LogInterface {
  <T>(value: T): T;
}
```

在上面的例子中，泛型仅仅用来约束了一个函数，我们也可以用泛型来约束整个接口的所有成员。这里需要注意一点，当泛型变量约束了整个接口之后，我们必须指定一个类型。

```typescript
interface LogInterface<T> {
  (value: T): T;
}
//在具体实现时指定类型
let myLog2: LogInterface<number> = value => value;
```

或者在定义时使用默认类型

```typescript
interface LogInterface<T = number> {
  (value: T): T;
}
let myLog2: LogInterface = value => value;
```

## 泛型类

与泛型接口非常类似，泛型也可以约束类。下面定义了一个泛型类。同样，我们把泛型变量放在类名后面，就可以约束整个类。

```typescript
class Car<T> {
  run(value: T) {
    return value;
  }
}
```

注意，泛型不能应用于类的静态成员。

```typescript
class Car<T> {
  static run(value: T) {
    return value;
  }
}
==========================
#静态成员不能引用类类型参数。
```

在实例化泛型类的时候，我们可以显式地指定 T 的类型。那么实例的方法就会受到泛型的约束。run() 的参数只能是 string 类型，否则编辑器会报错。

```typescript
let car =  new Car<string>()
car.run('fast'); // √

car.run(1);
===========
#类型“1”的参数不能赋给类型“string”的参数。
```

此外也可以不指定类型参数。这个时候该实例就不受泛型的约束，value 可以为任意类型。

```typescript
let car2 = new Car();

car2.run("fast");	// √
car2.run(1);		// √
```

## 泛型约束

现在我们再定义 log 函数，但返回的是参数的属性。编译器会报错，提示 T 类型不存在该属性。

```typescript
function log2<T>(value: T): T {
  console.log(value.length)
  return value;
}
===============================
#类型“T”上不存在属性“length”。
```

此时就需要 **泛型约束** 这个概念。我们先预定义一个含有 length 属性的接口，然后让 T 继承这个接口，T 就拥有了 length 属性，从而可以通过类型检查。

```typescript
interface Length {
  length: number;
}
function log2<T extends Length>(value: T): T {
  console.log(value.length);
  return value;
}
```

T 继承了 length 接口，就表示 T 受到了约束，不再是可以接受任何类型。输入的参数不管是什么类型，但必须具有 length 属性。例如数组、字符串、或含有 length 属性的对象。

```typescript
log2([1]);			 //√
log2("a");			 //√
log2({ length: 1 }); //√
```

## 小结

泛型的好处：

1. 函数和类可以轻松地支持多种类型，增强程序的扩展性
2. 不必谢多条函数重载，冗长的联合类型声明，增强代码可读性
3. 灵活控制类型之间的约束 



