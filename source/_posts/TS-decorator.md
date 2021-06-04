---
title: TS-decorator
date: 2021-06-04 11:45:45
tags:
---


### 概念
```
ts 文档
With the introduction of Classes in TypeScript and ES6, there now exist certain scenarios 
that require additional features to support annotating or modifying classes and class members.

Decorators provide a way to add both annotations and a meta-programming syntax 
for class declarations and members
```
翻译： ts和es6 引入class之后，有一些场景需要的支持注解或者修改class和class的members。Decorators 为class和class member 提供了注解和元编程的语法；
装饰器的目的是改变类、方法、属性的默认行为，并不属于运行时，作用于prototype上，所以设计上，就不会随着不同实例而改变，以上仅个人理解，不足参考
（只能用于class自身的行为，与运行上下文无关）

### 语法
#### class Decorator
```
A Class Decorator is declared just before a class declaration. 
The class decorator is applied to the constructor of the class and
can be used to observe, modify, or replace a class definition.
```
翻译：class decorator 作用于 class的constructor, 可以监听、修改、或者替换 class 的定义

语法
```typescript
function classDecorator(constructor: Function) {
  // your operate
}
```


示例1 - 修改class
```typescript
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}
@sealed
class BugReport {
    type = "report";
    title: string;

    constructor(t: string) {
      this.title = t;
    }
}
```
示例2 - 修改构造函数
```typescript
function reportableClassDecorator<T extends { new (...args: any[]): {} }>(constructor: T) {
    return class extends constructor {
      reportingURL = "http://www...";
    };
}
 
 @reportableClassDecorator
class BugReport {
    type = "report";
    title: string;

    constructor(t: string) {
      this.title = t;
    }
}
const bug = new BugReport("Needs dark mode");
console.log(bug.title); // Prints "Needs dark mode"
console.log(bug.type); // Prints "report"
// 装饰器并不会修改原本的class 类型，bug.reportingURL，ts会报类型未定义错误；
console.log(bug.reportingURL); // Prints "http://www..."
```

#### method decorator
```
The decorator is applied to the Property Descriptor for the method,
and can be used to observe, modify, or replace a method definition
```
翻译：方法装饰器，调用方法的Property Descriptor，可以监听、修改、或者替换方法的定义


语法
```typescript
function methodDecorator(...args) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // your code
  };
}
```

示例
```typescript
function enumerable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }

  @enumerable(false)
  greet() {
    return "Hello, " + this.greeting;
  }
}
```

#### 访问器装饰 accessor
```
The accessor decorator is applied to the Property Descriptor 
for the accessor and can be used to observe, modify, or replace an accessor’s definitions. 
```
accessor装饰器，调用accessor的 Property Descriptor，可以监听、修改或者替换 accessor的定义， 注意不能同时修改同一个属性的setter 和getter方法


语法
```typescript
function accessorDecorator(...args) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // your code
  };
}
```

示例
```typescript
function configurable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.configurable = value;
  };
}
class Point {
  private _x: number;
  private _y: number;
  constructor(x: number, y: number) {
    this._x = x;
    this._y = y;
  }

  @configurable(false)
  get x() {
    return this._x;
  }

  @configurable(false)
  get y() {
    return this._y;
  }
}
```

#### 属性装饰器 Property Decorators
```
The expression for the property decorator will be called as a function at runtime, with the following two arguments:

  1.Either the constructor function of the class for a static member, or the prototype of the class for an instance member.
  2. The name of the member.
```
语法
```typescript
// 可配合reflect-metadata使用, 因为属性的使用一般是在其他的方法中，装饰器是作用域原型链中，因此想要获取修改的属性，需要先存起来，再获取
function PropertyDecorator (target, propertyKey) { 
  // your code
}
```

示例
```typescript
import "reflect-metadata";
const formatMetadataKey = Symbol("format");
function format(formatString: string) {
  return Reflect.metadata(formatMetadataKey, formatString);
}
function getFormat(target: any, propertyKey: string) {
  return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
}

class Greeter {
  @format("Hello, %s")
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
  greet() {
    let formatString = getFormat(this, "greeting");
    return formatString.replace("%s", this.greeting);
  }
}
```

#### Parameter Decorators
语法
```typescript
// 可配合reflect-metadata使用, 装饰器是作用域原型链中，因此想要获取修改的参数值，需要先存起来，再获取
function paramDecorator(target: any, propertyKey: sting | symbol, parameterIndex: number) {
   // your code
}
```

例子
```typescript
import "reflect-metadata";
const requiredMetadataKey = Symbol("required");

function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  let existingRequiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
  existingRequiredParameters.push(parameterIndex);
  Reflect.defineMetadata( requiredMetadataKey, existingRequiredParameters, target, propertyKey);
}

function validate(target: any, propertyName: string, descriptor: TypedPropertyDescriptor<Function>) {
  let method = descriptor.value!;

  descriptor.value = function () {
    let requiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyName);
    if (requiredParameters) {
      for (let parameterIndex of requiredParameters) {
        if (parameterIndex >= arguments.length || arguments[parameterIndex] === undefined) {
          throw new Error("Missing required argument.");
        }
      }
    }
    return method.apply(this, arguments);
  };
}

class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }

  @validate
  print(@required verbose: boolean) {
    if (verbose) {
      return `type: ${this.type}\ntitle: ${this.title}`;
    } else {
     return this.title; 
    }
  }
}

```


#### ts decorator 语法糖代码
```javascript
"use strict";
var __decorate = function (decorators, target, key, desc) {
    // 参数个数
    var c = arguments.length;
    // 参数个数小于3 class decorator r=target; 否则 属性装饰器（function、property、param）都属于属性装饰器，获取属性描述desc
    var r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc;
    var d;
    // 如果用了 reflect-metadata 这个库
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") {
        r = Reflect.decorate(decorators, target, key, desc);
    } else {
        for (var i = decorators.length - 1; i >= 0; i--) {
            if (d = decorators[i]) {
                // class 执行 d(constructor); 其他 （target key descriptor）
                r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
            }
        }
    }
    // 不是类装饰器，覆盖或写入key，类 返回 包裹后的 r
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
var __metadata = (this && this.__metadata) || function (k, v) {
    if (typeof Reflect === "object" && typeof Reflect.metadata === "function") return Reflect.metadata(k, v);
};
var __param = (this && this.__param) || function (paramIndex, decorator) {
    return function (target, key) { decorator(target, key, paramIndex); }
};
```

### 如何开启
tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true,
    // 如果引入了 reflect-metadata
    "emitDecoratorMetadata": true,
    "types": ["reflect-metadata"]
  },
}


```