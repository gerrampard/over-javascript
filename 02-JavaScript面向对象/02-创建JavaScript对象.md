# 02-JavaScript 中的面向对象

## 一 创建对象方式

### 1.0 对象创建方式总结

常见的对象创建方式：

- 直接量方式（也即字面量）
- 使用基类 Object 创建
- 工厂模式
- 构造函数

### 1.1 直接量方式

直接量是最简单的实例创建方式：

```js
// 创建实例：
let obj = {
  name: '张学友',
  age: 13,
  run: function () {
    console.log(this.name + '唱歌...')
  },
}

// 操作实例：
p.run()
```

方法也可以简写为：

```js
// 创建实例：
let obj = {
  name: '张学友',
  age: 13,
  run() {
    console.log(this.name + '唱歌...')
  },
}
```

### 1.2 使用 Object 基类方式

```js
let person = new Object()

person.name = 'Nicholas'
person.age = 29
person.job = 'Software Engineer'
person.sayName = function () {
  console.log(this.name)
}

person.sayName()
```

这种方式写法较为繁琐，往往直接使用字面量方式即可。

ES5 的 Object 对象还提供了 create()方法用来创建对象：

```js
let obj = Object.create({
  name: 'lisi',
  age: 13,
})
console.log(obj.name)
```

注意：该方法传入参数为 null 时，会创建一个没有原型的新对象，不会继承任何东西，甚至不能使用`toString()`这样的基础方法，所以创建空对象的方式是：

```js
let obj = Object.create(Object.prototype)
```

### 1.3 工厂模式创建对象

工厂模式可以对要创建的对象实现更多的可定制：

```js
function createPerson(name, age, job) {
  let o = new Object()
  o.name = name
  o.age = age
  o.job = job
  o.sayName = function () {
    console.log(this.name)
  }
  return o
}

let person1 = createPerson('Nicholas', 29, 'Software Engineer')
let person2 = createPerson('Greg', 27, 'Doctor')
```

工厂模式并不是真正创建对象的方式，内部仍然是使用 new 或者直接量等。只是利用设计模式思想，让开发更方便（世界更美好）。

## 二 使用构造函数与原型创建对象

### 2.1 使用构造函数

传统的面向对象语言，如 Java，对象都是通过构造函数进行构造，其作用很类似工厂函数：

```js
// 构造函数首字母要大写
function Man(name, age) {
  this.name = name
  this.age = age
  this.sex = '男' // 无需使用参数的成员
  this.sayName = function () {
    console.log(this.name)
  }
}

let person1 = new Person('Nicholas', 29, 'Software Engineer')
let person2 = new Person('Greg', 27, 'Doctor')
person1.sayName() // Nicholas
person2.sayName() // Greg
```

构造函数与工厂函数区别是：构造函数没有显式的创建一个对象，属性和方法直接都赋值给了 this，也不需要 return。

构造函数与普通函数没有本质上的区别，其调用方式采用 new。其实只要是使用 new 方式调用的函数即构造函数！

构造函数创建对象有个**弊端**：当我们无限制的 new 对象时，对象引用的函数对象就会越来越多！，逻辑上这些函数都应该是同一个函数（做同一个事情！），反复创建浪费了空间：

```js
function Person(name, age) {
  this.name = name
  this.age = age
  this.say = function () {
    console.log(`My name is ${this.name}`)
  }
}

let p1 = new Person('lisi', 18)
let p2 = new Person('zs', 22)
console.log(p1.say == p2.say) // false
```

### 2.2 使用原型

为了解决构造函数问题，我们可以成员方法转义到构造函数外部：

```js
function Person(name, age) {
  this.name = name
  this.age = age
  this.say = say
}

function say() {
  console.log(`My name is ${this.name}`)
}

let p1 = new Person('lisi', 18)
let p2 = new Person('zs', 22)
console.log(p1.say == p2.say) // true
```

上述方式解决了构造函数问题，但是再次出现了新的问题：全局声明的方法 say 污染了作用于，他本该只属于对象专属调用的，大量的方法更会造成结构的混乱，为此 JS 在创建之初采用了原型的方式解决该问题。

每个函数都有一个 prototype 属性，用来包含所有实例**共享**的属性、方法！

```js
function Man(name, age) {
  this.name = name
  this.age = age
}

// 共享的两个成员
Man.prototype.sex = '男'
Man.prototype.say = function () {
  console.log(`My name is ${this.name}`)
}

let m1 = new Man('lisi', 18)
let m2 = new Man('zs', 22)
m1.say() // My name is lisi
m2.say() // My name is zs
console.log(m1.sex) // 男
console.log(m2.sex) // 男

console.log(m1.say == m2.say) // true
```

注意：原型内适合存放共享数据，像 name、age 这样的成员应该在构造函数内，非共享数据写在原型中，会造成数据混乱：

```js
function Person() {}
Person.prototype = {
  constructor: Person,
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  friends: ['Shelby', 'Court'],
  sayName() {
    console.log(this.name)
  },
}

let person1 = new Person()
let person2 = new Person()
person1.friends.push('Van')
console.log(person1.friends) // "Shelby,Court,Van"
console.log(person2.friends) // "Shelby,Court,Van"
console.log(person1.friends === person2.friends) // true
```

### 2.3 写法优化

采用构造函数与原型创建对象的方式中，prototype 上的每个方法都要写一次`构造函数.prototype.方法名=`，既然 prototype 是个对象，其实可以使用字面量形式直接书写。

```js
function Person() {}

Person.prototype = {
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  sayName() {
    console.log(this.name)
  },
}
```

这里会出现一个问题：

```js
let friend = new Person()
console.log(friend.constructor == Person) // false
```

解决方案：

```js
function Person() {}

Person.prototype = {
  constructor: Person,
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  sayName() {
    console.log(this.name)
  },
}
```

这里仍然有个问题：上述的 constructor 属性会创建一个 `[[Enumerable]]` 为 true 的属性，而原生的 constructor 属性默认是不可枚举的，如果要照顾兼容性，则可以使用下面的方式：

```js
function Person() {}
Person.prototype = {
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  sayName() {
    console.log(this.name)
  },
}

// 恢复 constructor 属性
Object.defineProperty(Person.prototype, 'constructor', {
  enumerable: false,
  value: Person,
})
```

## 四 理解

### 4.2 总结

**一般将属性放在构造函数中，方法放在原型中！**，因为：

- 构造函数内部的函数，在每次创建实例的时候都会在内存中新建一次该函数，比如一个 Person()类 new10 次就会产生 10 个 run 方法，会严重造成资源浪费。但是在 protoype 属性上挂载的方法则不会，因为原型本身其实也是一个对象，所有的实例都会引用这一个对象上的方法。
- 原型上书写属性字段，会造成 new 出的实例的该字段值都是一样的，无法定制化。比如上述实例 new 出的对象其 sex 属性都为 ”男“

方法也是对象内部数据的一部分，既然构造函数用来描述对象的内部的数据，为什么不把方法也写在构造函数内部，而是要额外写在原型中呢？

```js
function Person(name) {
  this.name = name
  // 直接写在构造函数中的实例方法
  this.speak = function () {
    console.log(this.name + '在说话...')
  }
}

Person.prototype.sing = function () {
  console.log(this.name + '在唱歌...')
}

let p1 = new Person('张三')
p1.sing() // 张三在唱歌...
p1.speak() // 张三在说话...

let p2 = new Person('李四')
p2.sing() // 李四在唱歌...
p2.speak() // 李四在说话...

console.log(p1.sing == p2.sing) // true
console.log(p1.speak == p2.speak) // false
```

我们发现实例方法如果直接写在构造函数后，不同的实例方法在判断相等时候，结果是 false。这是因为每次通过 new 创建一个全新的实例时，都等于重新申请了新的内存，并将构造函数的数据拷贝进该内存，这也造成了 p1 和 p2 的 speak 方法不相等，也导致了无故多创建了一个函数内存！

而原型不会被拷贝，因为他是构造函数自身的属性，不是实例上的属性，为了统一处理这些不会被拷贝的数据，JS 规定将这些数据都存放在构造函数的 prototype 属性上。

由上也可以看出，原型上也是可以存储非函数的普通变量数据的，但是这种数据无法被实例改变，只能像静态方法那样，通过类名修改。一般不推荐在面向对象思想中这样做。

## 三 new 的过程

new 创建对象的过程：

- 1、在内存中开辟空间，创建一个新的空对象
- 2、将构造函数内部的 this 指向新对象
- 3、新对象的 Prototype 特性被赋值为构造函数的 prototype 属性
- 4、执行构造函数内部的代码，即给新对象添加属性！
- 5、如果构造函数返回空对象，则返回该对象，否则返回刚创建的新对象

伪代码演示 new 过程：

```js
this = {} // 1 2
this.__proto__ = 构造函数.prototype // 3
this.age = 18 // 4
return this // 5
```

原型的动态性：因为从原型上搜索值的过程是动态的，所以即使实例在修改原型之前已经存在，任何时候对原型对
象所做的修改也会在实例上反映出来。

```js
let friend = new Person()
Person.prototype.sayHi = function () {
  console.log('hi')
}
friend.sayHi() // "hi"，没问题！
```

虽然随时能给原型添加属性和方法，并能够立即反映在所有对象实例上，但这跟重写整个原型是两回事。实例的[[Prototype]]指针是在调用构造函数时自动赋值的，这个指针即使把原型修改为不同的对象也不会变。重写整个原型会切断最初原型与构造函数的联系，但实例引用的仍然是最初的原型。记住，实例只有指向原型的指针，没有指向构造函数的指针。

```js
function Person() {}
let friend = new Person()
Person.prototype = {
  constructor: Person,
  name: 'Nicholas',
  age: 29,
  job: 'Software Engineer',
  sayName() {
    console.log(this.name)
  },
}
friend.sayName() // 错误
```
