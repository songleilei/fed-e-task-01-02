# fed-e-task-01-02

函数式编程与 JavaScript 性能优化

#### 简单题

1. ##### 描述引用计数的工作原理和优缺点

   工作原理：给对象添加一个计数器，被引用则计数器加 1；引用失效，计数器减 1。当计数器为 0 时， 即不被引用，则回收该对象。

   优点：

   - 即时处理垃圾对象
   - 最大限度减少程序卡顿

   缺点：

   - 无法解决循环引用的问题
   - 资源开销大

2) ##### 描述标记整理算法的工作流程

   工作流程：

   1. 标记对象：遍历所有对象标记活动对象
   2. 清除对象：先将活动对象和非活动对象进行整理，移动对象位置，然后进行清除
   3. 回收空间

3. ##### 描述 V8 中新生代存储区垃圾回收的流程

   流程：程序主线程执行完毕进行垃圾回收时，先将 From 区的对象进行标记整理，然后将 From 区的活动对象都复制到 To 区。清除 From 区 ，最后将两个区互换。一个对象多次复制过程中，依旧为活动对象，则将该活动对象移动至老生代，即晋升。

4) ##### 描述增量标记算法在何时使用，及工作原理

   何时使用：V8 老生代垃圾回收会使用

   工作原理：垃圾回收的时候，遍历对象进行标记，此标记过程与程序执行交替进行。

#### 代码题 1

##### 基于以下代码完成下面的四个练习

```javascript
const fp = require("lodash/fp");

// 数据
// horsepower 马力， dollar_value 价格， in_stock 库存
const cars = [
  {
    name: "Ferrari FF",
    horsepower: 600,
    dollar_value: 700000,
    in_stock: true,
  },
  {
    name: "Spyker C12",
    horsepower: 650,
    dollar_value: 648000,
    in_stock: false,
  },
  {
    name: "JAGUAR xkr-s",
    horsepower: 550,
    dollar_value: 132000,
    in_stock: false,
  },
  {
    name: "Audi R8",
    horsepower: 525,
    dollar_value: 114200,
    in_stock: false,
  },
  {
    name: "Aston Martin Onr-77",
    horsepower: 750,
    dollar_value: 185000,
    in_stock: true,
  },
  {
    name: "Pagani Huayra",
    horsepower: 700,
    dollar_value: 130000,
    in_stock: false,
  },
];
```

##### 练习 1：

使用函数组合 fp.flowRight() 重新实现下面这个函数

```javascript
/*
let isLastInStock = function (cars) {
  let last_car = fp.last(cars);
  return fp.prop("in_stock", last_car);
};
*/

const isLastInStock = fp.flowRight(fp.prop("in_stock"), fp.last);
```

##### 练习 2：

使用 fp.flowRight()、fp.prop() 和 fp.first() 获取第一个 car 的 name

```javascript
const isFirstInStock = fp.flowRight(fp.prop("name"), fp.first);
```

##### 练习 3：

使用帮助函数 \_average 重构 averageDollarValue，使用函数组合的方式实现

```javascript
let _average = function (xs) {
  return fp.reduce(fp.add, 0, xs) / xs.length;
};

/*
let averageDollarValue = function (cars) {
  let dollar_values = fp.map(function (cars) {
    return cars.dollar_value;
  }, cars);
  return _average(dollar_values);
};
*/

const averageDollarValue = fp.flowRight(
  _average,
  fp.map((v) => v.dollar_value)
);
```

##### 练习 4：

使用 flowRight 写一个 sanitizeNames() 函数，返回一个下划线连接的小写字符串，把数组中的 name 转换为这种形式：例如：sanitizeNames(['Hello World']) => ["hello_world"]

```javascript
let _underscore = fp.replace(/\W+/g, "_");

const sanitizeNames = fp.map(fp.flowRight(_underscore, fp.toLower));

const arr = fp.map((v) => v.name, cars);

console.log(sanitizeNames(arr));

/**
[
  "ferrari_ff",
  "spyker_c12",
  "jaguar_xkr_s",
  "audi_r8",
  "aston_martin_onr_77",
  "pagani_huayra",
];
*/
```

#### 代码题 2

##### 基于下面提供的代码，完成后续的四个练习

```javascript
class Container {
  static of(value) {
    return new Container(value);
  }

  constructor(value) {
    this._value = value;
  }

  map(fn) {
    return Container.of(fn(this._value));
  }
}

class Maybe {
  static of(x) {
    return new Maybe(x);
  }

  isNothing() {
    return this._value === null || this._value === undefined;
  }

  constructor(x) {
    this._value = x;
  }

  map(fn) {
    return this.isNothing() ? this : Maybe.of(fn(this._value));
  }
}

module.exports = {
  Maybe,
  Container,
};
```

##### 练习 1：

使用 fp.add(x, y) 和 fp.map(f, x) 创建一个能让 functor 里的值增加的函数 ex1

```javascript
const fp = require("lodash/fp");

const { Maybe, Container } = require("./work");

let maybe = Maybe.of([5, 6, 1]);

let ex1 = maybe.map(fp.map(fp.add(1)));
```

##### 练习 2：

实现一个函数 ex2，能够使用 fp.first 获取列表的第一个元素

```javascript
const fp = require("lodash/fp");

const { Maybe, Container } = require("./work");

let xs = Container.of(["do", "ray", "me", "fa", "so", "la", "ti", "do"]);

let ex2 = xs.map(fp.first);
```

##### 练习 3：

实现一个函数 ex3，使用 safeProp 和 fp.first 找到 user 的名字的首字母

```javascript
const fp = require("lodash/fp");

const { Maybe, Container } = require("./work");

let safeProp = fp.curry(function (x, o) {
  return Maybe.of(o[x]);
});

let user = { id: 2, name: "Albert" };

let ex3 = safeProp("name")(user).map(fp.first);
```

##### 练习 4：

使用 Maybe 重写 ex4，不要有 if 语句

```javascript
const fp = require("lodash/fp");

const { Maybe, Container } = require("./work");

/*
let ex4 = function (n) {
  if (n) {
    return parseInt(n);
  }
};
*/

const ex4 = function (value) {
  return Maybe.of(value).map(parseInt);
};
```
