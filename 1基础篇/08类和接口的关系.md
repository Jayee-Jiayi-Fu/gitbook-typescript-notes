# 类和接口的关系

## 类类型接口

下面定义一个 Human 接口，Asian 类用 implements 关键字实现了这个接口。

```typescript
interface Human {
  name: string;
  eat(): void;
}

class Asian implements Human {
  constructor(name: string) {
    this.name = name;
  }
  name: string;
  eat() {}
}
```

我们需要注意两点：

1. 类在实现接口的过程中，必须声明接口中所有的属性，否则编译器会报错。但 Asian 类定义自己的属性是可以的。

```typescript
class Asian implements Human {
  constructor(name: string) {
    this.name = name;
  }
  name: string;
  sleep(){}	// √  
  // eat() {}
}
==============================
#类“Asian”错误实现接口“Human”。
#Property 'eat' is missing in type 'Asian' but required in type 'Human'.
```

2. 接口只能约束类的公有成员。

```typescript
class Asian implements Human {
  //...
  private name: string;
}
==============================
#类“Asian”错误实现接口“Human”。
#属性“name”在类型“Asian”中是私有属性，但在类型“Human”中不是。
```

3. 接口也不能约束类的构造函数。例如下面期望在接口中定义构造函数，编译器会报错。

```typescript
interface Human {
  new (name: string): void;
  //...
}
==========================
#类“Asian”错误实现接口“Human”。
#类型“Asian”提供的内容与签名“new (name: string): void”不匹配。
```

## 接口的继承

接口可以像类一样实现继承，并且一个接口可以继承多个接口。例如下面紧接前面的代码，又实现了三个接口。Man 接口继承了 Human 接口，Boy 接口继承了 Man 和 Child 接口。

```typescript
interface Man extends Human {
  run(): void;
}
interface Child {
  cry(): void;
}
interface Boy extends Man, Child {}
```

如果我们想要创建一个 Boy 接口类型的对象，该对象就必须实现 Boy 接口所继承的所有祖先接口定义的属性。

```typescript
let boy: Boy = {
  //from interface Human
  name: "Mark",
  eat() {},
  //from interface Man
  run() {},
  //from interface Child
  cry() {}
};
```

接口的继承可以抽离出可重用的接口，也可以将多个接口合并成一个接口。

## 接口继承类

接口除了可以继承接口外，还可以继承类。这就相当于，接口把类的公有成员都抽象了出来。也就是只有类的成员结构，而没有具体实现。

下面我们定义一个 Auto 类，它只有一个属性 state。然后定义一个接口 AutoInterface 继承了这个类。那么这个接口就隐含了 state 属性。要想实现这个接口，就需要一个类的成员实现这个属性 state。

```typescript
class Auto {
  state = 1;
}

interface AutoInterface extends Auto {}

class C implements AutoInterface {
  state = 2;
}
```

Auto 的子类也可以实现 AutoInterface 这个接口。但在这个子类中，我们不必再定义 state 属性。因为 Auto 的子类自然继承了父类的属性。

```typescript
class Bus extends Auto implements AutoInterface{ }
```

需要注意的是，接口在抽离类成员的时候，不仅抽离了公共成员，还抽离了私有成员和受保护成员。

```typescript
class Auto {
  state = 1;
  private state2 = 2;
}
interface AutoInterface extends Auto {}
class C implements AutoInterface {
  state = 2;
  // 没有实现 Auto 的私有属性 state2
}
=================================
#类“C”错误实现接口“AutoInterface”。
#Property 'state2' is missing in type 'C' but required in type 'AutoInterface'.
```

一图胜千言：

![class-interface](http://q5frcy1n7.bkt.clouddn.com/images/gitbook-typescript-notes/class-interface.JPG)

- 接口之间是可以相互继承的，这样能够实现接口的复用
- 类之间也可以相互继承，这样可以实现方法和属性的复用
- 接口可以通过类来实现，但接口只能约束类的公有属性
- 类可以通过接口抽离出公有成员、私有成员和受保护成员