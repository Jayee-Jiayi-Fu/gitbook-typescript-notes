# 类

ES6 中引入了 class 关键字，这使得前端开发人员终于可以像传统面向对象语言那样创建类了。总体而言，TypeScript 中的类覆盖了 ES6 中的类，同时引入了一些其他特性。使用的时候需要注意两者区别。

## 类的基本实现

```typescript
class Dog {
  constructor(name: string) {
    this.name = name;
  }
  name: string;
  run() {}
}
```

需要注意两点：

- 无论是 ES6   还是 TS，类的成员属性都是实例属性，类的成员方法都是原型方法

- 与 ES6 不同的是，类的属性必须具有初始值，或者在构造函数中被初始化

  ```typescript
  class Dog {
    constructor(name: string) {}
    name: string;
    run() {}
  }
  ============================
  #属性“name”没有初始化表达式，且未在构造函数中明确赋值。
  ```

  如果类属性没有在构造函数中初始化，编译器会报错。当然解决办法还可以是给属性添加初始值：

  ```typescript
  name: string = 'Puppy'; // √
  ```

  或声明为可选属性：

  ```typescript
  name?: string;			// √
  ```

## 继承

实现类的继承主要以下几个步骤：

1. 子类使用 extends 关键字指定继承的父类。

2. 子类的构造函数必须包含 \"super()" 调用，这是 ES6 中强制规定。

3. super() 代表父类的实例。父类中构造函数需要传入 name 参数，super() 中也一定要有。

4. 同样子类的属性也需要初始化，并且 this 调用必须在 super() 之后。

   ```typescript
   class Husky extends Dog {
     constructor(name: string, color: string) {
       super(name);
       this.color = color;
     }
     color: string;
   }
   ```

## 成员修饰符

类的成员修饰符是 TS 对 ES6 的一种扩展。包括以下几种：

### public

公有成员，类的所有属性默认都是 public，它的含义是对所有人都是可见的

### private

私有成员，只能在类的本身被调用，而不能被类的实例或子类调用。

```typescript
class Dog {
  constructor(name: string) {
    this.name = name;
  }
  private pri() {} 
}
```

在实例中调用，编译器会报错。

```typescript
let dog = new Dog("wangwang");
dog.pri();
===================================
#属性“pri”为私有属性，只能在类“Dog”中访问。
```

子类调用父类的私有成员，编译器也会报错。

```typescript
class Husky extends Dog {
  constructor(name: string) {
    super(name);
    this.pri();
  }
}
==============================
#属性“pri”为私有属性，只能在类“Dog”中访问。
```

当把类的构造函数标记为私有成员，那么该类就不能被实例化和继承。

 ```typescript
class Dog {
  private constructor(name: string) {...}
}

let dog = new Dog("wangwang");
================================
#类“Dog”的构造函数是私有的，仅可在类声明中访问。

class Husky extends Dog {...}
===============================
#无法扩展类“Dog”。类构造函数标记为私有。
 ```

### protected

受保护成员，成员只能在类本身或子类中被调用。

```typescript
class Dog {
  constructor(name: string) {
    this.name = name;
  }
  protected pro() {}
}
```

在实例中被调用，编译器会报错。

```typescript
let dog = new Dog("wangwang");
dog.pro();
===================================
#属性“pro”受保护，只能在类“Dog”及其子类中访问。
```

在子类中调用父类的受保护成员，没问题。

```typescript
class Husky extends Dog {
  constructor(name: string, color: string) {
    super(name);
    this.pro()
  }
}
```

构造函数也能被声明为 protected。这样该类就不能被实例化，但可以被继承，就相当于声明了一个基类。

```typescript
class Dog {
  protected constructor(name: string) {...}
}

let dog = new Dog("wangwang");
==============================
#类“Dog”的构造函数是受保护的，仅可在类声明中访问。

class Husky extends Dog {...} // √
```

### readonly

只读属性，能被读取，不能被更改。

 ```typescript
class Dog {
  //...
  readonly age: number = 1;
}

let dog = new Dog("wangwang");
console.log(dog.age);	// 1
dog.age = 2;
============
#Cannot assign to 'age' because it is a read-only property.
 ```

### static

静态成员，只能通过类名调用。

```typescript
class Dog {
  //...
  static food: string = "bones";
}

console.log(Dog.food);	// bones

let dog = new Dog("wangwang");
console.log(dog.food)
=====================
Property 'food' is a static member of type 'Dog'
```

当然，类的静态成员也能被继承，并且也只能通过类名调用。

```typescript
class Husky extends Dog {...}

console.log(Husky.food);	// bones

let husky = new Husky("husky", "black", 10);
husky.food;
===========
Property 'food' is a static member of type 'Husky'
```

### 构造函数的参数使用修饰符

除了类的成员可以添加修饰符外，构造函数的参数也可以添加修饰符，它的作用是把参数直接变成了类的属性，这样我们就能省略在类中的定义了。

```typescript
class Husky extends Dog {
  constructor(name: string, color: string, public weight:number) {
    super(name);
    this.color = color;
    gbthis.weight = weight;
  }
  color: string;
  //weight:number;//因为使用了修饰符，可以省略
}
```

## 抽象类

ES6 中并没有引入抽象类的概念，这是 TS 对 ES 的又一次扩展。所谓抽象类就是：只能被继承而不能被实例化的类。如果实例化抽象类，编译器会报错。

```typescript
abstract class Animal{}
let animal = new Animal()
=========================
#无法创建抽象类的实例。
```

在抽象类中，我们可以定义一些具体的实现，这样子类就可以直接继承而无需再次编写，实现了代码的复用。

```typescript
abstract class Animal {
  eat() {
    console.log("eat");
  }
}
class Cat extends Animal {...}
let cat = new Cat();
cat.eat();		// eat
```

在抽象类中，也可以不定义方法的具体实现，这就构成了一个抽象方法。抽象方法的好处是，你明确知道子类有其他的实现，而不必在抽象类中实现。

```typescript
abstract class Animal {
  abstract sleep(): void;
}
class Cat extends Animal {
  //...
  sleep() {
    console.log("Cat sleep");
  }
}
let cat = new Cat();
cat.sleep();	// Cat sleep
```

抽象类的好处是：你可以抽离出一些事物的共性，这样就有利于代码的复用和扩展。另外抽象类也可以实现多态。

## 多态

所谓多态，就是在父类中定义一个抽象方法，在多个子类中对这个抽象方法有不同的实现。在程序运行的时候，会根据不同的对象执行不同的操作。这样就实现了运行时的绑定。

```typescript
abstract class Animal {
  abstract sleep(): void;
}
class Cat extends Animal {
  sleep() {
    console.log("Cat sleep");
  }
}
class Bird extends Animal {
  sleep() {
    console.log("Bird sleep");
  }
}

let cat = new Cat();
let bird = new Bird();
cat.sleep();	// Cat sleep
bird.sleep();	// Bird sleep
```

## this 类型

特殊的 TS 类型。类的成员方法可以直接返回 this，这样就可以很方便地实现链式调用。

```typescript
class WorkFlow {
  step1() {
    return this;
  }
  step2() {
    return this;
  }
}
new WorkFlow().step1().step2();
```

在继承的时候，this 类型也可以表现出多态。这里的多态表示，this 既可以表示父类，也可以表示子类，这样就保持了父类和子类调用的连贯性。

```typescript
class MyFlow extends WorkFlow {
  next() {
    return this;
  }
}
new MyFlow().next().step1().step2();
```



## 小结

对比一下 TS 和 ES 中类的一些差别，你可能发现， TS 把 ES 中缺失的一些特性都补回来了。这样 TS 就更像一门面向对象语言了。



