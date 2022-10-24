[↑ Back to TypeScript Notes](Contents.md)  
[← Home](/README.md)

# TypeScript for Java/C# Programmers

A "5-minute" overview of TypeScript for Java/C# Programmers  
[Link to overview](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-oop.html)  

## Co-learning JavaScript

### If Familiar with JavaScript

This helps explain some of the common misconceptions and pitfalls other Java/C# developers are susceptible to. Such as - TypeScript vs Java/C# models differences are important to keep in mind. 

### If New to JavaScript

It's reccomended to learn **a little bit** of JavaScript without types first to understand it's runtime behaviors.

TypeScript uses the same runtime as JavaScript, so it's good to not limit to **only** TypeScript-specific resources. 

## Rethinking the Class

Java/C# approach to classes is mainly code orginization, and a container for all data and behavior at runtime. 

With TypeScript functionality and data doesn't always need to be forced into a class.

### Free Functions and Data

With JavaScript functions can live anywhere and data can be passed around without a pre-defined [`class`](https://www.w3schools.com/java/java_classes.asp) or [`struct`](https://www.w3schools.com/c/c_structs.php). "Free" funcitons tend to be preferred model for JavaScript. 

### Static Classes 

Java/C# constructs such as [singletons and static classes](https://www.w3schools.blog/differences-static-class-vs-singleton-patterns) are unnecessary in TypeScript. 

## OOP in TypeScript

TypeScript still supports common `OOP` patterns such as interfaces, inheritance, and static methods. This is due to some problems being well-suited to utilize traditional OOP hierarchy. 

## Rethinking Types

TypeScript type ≠ Java/C# type

### Nominal Reified Type Systems 

Java/C# value or objects has one exact type - `null`, primative, or a known class type. Methods like `value.GetType()` or `value.getClass()` to find exact type at runtime. Due to the type residing in a class somewhere, this prevents using a different class with a similar "shape" unless there's explicit inheritance or an interface.

**TL;DR** - types wrote in code are present at runtime, and the types are related via their declarations, not their structures. 

### Types as Sets

**Java/C#** uses a 1:1 correspondence between runtime types and compile-time declarations. 

**TypeScript** types are a **set of values** that share something in common. This allows a value to belong to **many** sets at the same time. 

Looking and utilizing types as sets allows for a easier approach to handling a value that could fall under being a `string` or `int`. With TypeScript it belongs to the union of those sets: `string | number`. 

TypeScript out of the box has a number of tools to work with types in a set-theoretic way. 

### Erased Structural Types

TypeScript's type syste is structural, not nominal. 

In the example below `obj` is passed into function `logPoint()` and `logName()` which both accept different types, but the same `obj` value is provided. 

Example

```TypeScript 
interface Pointlike {
  x: number;
  y: number;
}
interface Named {
  name: string;
}

function logPoint(point: Pointlike) {
  console.log("x = " + point.x + ", y = " + point.y);
}

function logName(x: Named) {
  console.log("Hello, " + x.name);
}

const obj = {
  x: 0,
  y: 0,
  name: "Origin",
};

logPoint(obj);
logName(obj);
```

Example Output

```Text
[LOG]: "x = 0, y = 0" 
--------------------------
[LOG]: "Hello, Origin" 
```

This demonstrates that TypeScript's type system **is not reified**, and demonstrates **types as sets**.  
`obj` is a member of both the `PointLike` and `Named` set of values. 

### Consequences of Structural Typing

Two common aspects that surprise OOP developers

#### **Empty Types**

In the example below we can see a class `Empty` and a function `fn` that accepts an argument that's `Empty`.  
Then the `fn` function get's called with an argument of  `{k: 10}`.

The goal of this example is to see if this results in an error.

```TypeScript
class Empty {}

function fn(arg: Empty) {
  // do something?
}

// No error, but this isn't an 'Empty' ?
fn({ k: 10 });
```
This **does not** result in an error due to TypeScript views the argument having **all** the properties `Empty` does, due to `Empty` not having any arguments. 

**The point** - A subclass cannot **remove** a property of its base class, doing so would destroy the relationship between the derived class adn it's base.

#### **Identical Types**

In this example we have two classes `Car` and `Golfer` that both contain `drive()`. Then we see a variable `w` of type `Car` creating a new `Golfer`. 

The goal of this example is to see if this results in an error. 

```TypeScript
class Car {
  drive() {
    // hit the gas
  }
}
class Golfer {
  drive() {
    // hit the ball far
  }
}
// No error?
let w: Car = new Golfer();
```

This **also** does not result in an error due to the **structure** of these classes are the same.

Identical classes that shouldn't be related, like the example, are not common.


