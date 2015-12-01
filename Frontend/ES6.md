# [ECMAScript 6](http://www.infoq.com/cn/es6-in-depth/)

## 迭代器和for-of循环
+  不要用for-in循环遍历数组，它是用于遍历普通对象的各个属性的key的；
+  for-of循环可以用来遍历数组，没有for-in的缺陷，也没有forEach的缺陷：无法break, continue, return；
+  for-in循环用来遍历对象属性，for-of循环用来遍历数据：例如数组中的值；
+  for-of还支持其他集合的遍历（`Map`, `Set`），也能用于字符串遍历（视其为Unicode字符数组）；

  ```javascript
  for (var value of myArray) {
    console.log(value);
  }
  
  var uniqueWords = new Set(words);
  for (var word of uniqueWords) {
    console.log(word);
  }
  
  for (var [key, value] of phoneBookMap) {
    console.log(key + "'s phone number is: " + value);
  }
  ```

+  for-of循环语句通过方法调用（迭代器方法）来遍历各种集合
+  迭代器对象
  +  拥有`[Symbol.iterator]()`的对象被称为可迭代的
  +  迭代器对象可以是任意具有`.next()`方法的对象
  +  for-of循环将重复调用这个方法，每次循环调用一次
  
  ```javascript
  var zeroesForeverIterator = {
  [Symbol.iterator]: function () {
    return this;
    },
    next: function () {
    return {done: false, value: 0};
  }
  };
  ```

## 生成器 Generators
```javascript
function* quips(name) {
  yield "你好 " + name + "!";
  yield "希望你能喜欢这篇介绍ES6的译文";
  if (name.startsWith("X")) {
    yield "你的名字 " + name + "  首字母是X，这很酷！";
  }
  yield "我们下次再见！";
}
```
+  普通函数使用`function`声明，而生成器函数使用`function*`声明。
+  在生成器函数内部，有一种类似`return`的语法：关键字`yield`。二者的区别是，普通函数只可以`return`一次，而生成器函数可以`yield`多次（当然也可以只`yield`一次）。在生成器的执行过程中，遇到`yield`表达式立即暂停，后续可恢复执行状态。
+  这就是普通函数和生成器函数之间最大的区别，普通函数不能自暂停，生成器函数可以。
+  执行效果：

```javascript
> var iter = quips("jorendorff");
  [object Generator]
> iter.next()
  { value: "你好 jorendorff!", done: false }
> iter.next()
  { value: "希望你能喜欢这篇介绍ES6的译文", done: false }
> iter.next()
  { value: "我们下次再见！", done: false }
> iter.next()
  { value: undefined, done: true }
```
+  调用一个生成器时，它并非立即执行，而是返回一个已暂停的生成器对象，以后对其每调用一次`.next()`方法，函数调用将其自身解冻并一直运行到下一个`yield`表达式，再次暂停。
+  调用最后一个`iter.next()`时，我们最终抵达生成器函数的末尾，所以返回结果中`done`的值为`true`。
+  每当生成器执行`yield`语句，生成器的堆栈结构（本地变量、参数、临时值、生成器内部当前的执行位置）被移出堆栈。然而，生成器对象保留了对这个堆栈结构的引用（备份），所以稍后调用`.next()`可以重新激活堆栈结构并且继续执行。
+  值得特别一提的是，生成器不是线程，当生成器运行时，它和调用者处于同一线程中，拥有确定的连续执行顺序，永不并发。
+  与系统线程不同的是，生成器只有在其函数体内标记为`yield`的点才会暂停。
+  生成器的作用
  +  生成器是迭代器：所有的生成器都有内建`.next()`和`[Symbol.iterator]()`方法的实现。你只须编写循环部分的行为。
  +  使任意对象可迭代。编写生成器函数遍历这个对象，运行时`yield`每一个值。然后将这个生成器函数作为这个对象的`[Symbol.iterator]`方法。
  +  简化数组构建函数。
    +  获取异常尺寸的结果。例如无限长的数组。
    +  重构复杂循环。
    +  构建与迭代相关的工具。
  +  lazy计算。仅当需要时进行计算。
+  `.next`可选参数
+  `.return`
+  `.throw`
+  `yield*`
  
## 模板字符串
```javascript
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      `用户 ${user.name} 未被授权执行 ${action} 操作。`);
  }
}
```
+  反撇号（`）基础知识
  +  模板占位符中的代码可以是任意JavaScript表达式，所以函数调用、算数运算等这些都可以作为占位符使用，你甚至可以在一个模板字符串中嵌套另一个，我称之为模板套构（template inception）。
  +  如果这两个值都不是字符串，可以按照常规将其转换为字符串。例如：如果`action`是一个对象，将会调用它的`.toString()`方法将其转换为字符串值。
  +  如果你需要在模板字符串中书写反撇号、`$`和`{`，你必须使用反斜杠将其转义。
  +  模板字符串可以多行书写，模板字符串中所有的空格、新行、缩进，都会原样输出在生成的字符串中。
+  反撇号的未来
  +  它们不会为你自动转义特殊字符
  +  它们无法很好地与国际化库相配合
  +  它们不能替代模板引擎的地位

## 不定参数和默认参数
+  以前的版本中有“神奇的arguments对象”，ES6引入了与其他语言类似的不定参数语法：

```javascript
function containsAll(haystack, ...needles) {
  for (var needle of needles) {
    if (haystack.indexOf(needle) === -1) {
      return false;
    }
  }
  return true;
}
```
+  如果没有额外的参数，不定参数就是一个空数组，它永远不会是undefined。
+  默认参数也与c++类似，但有几点需要注意
  +  默认值表达式在函数调用时自左向右求值，这一点与Python不同。这也意味着，默认表达式可以使用该参数之前已经填充好的其它参数值。
  +  传递undefined值等效于不传值，将使用定义的默认值
  +  没有默认值的参数隐式默认为undefined
+  停止使用arguments


## 解构（Destructuring）
+  数组与迭代器的解构

  ```javascript
  [ variable1, variable2, ..., variableN ] = array;
  ```

  +  可以对任意深度的嵌套数组进行解构
  +  可以在对应位留空来跳过被解构数组中的某些元素
  
    ```javascript
    var [,,third] = ["foo", "bar", "baz"];
    ```
  +  还可以通过“不定参数”模式捕获数组中的所有尾随元素
  +  当访问空数组或越界访问数组时，对其解构与对其索引的行为一致，最终得到的结果都是：undefined
  +  数组解构赋值的模式同样适用于任意迭代器
  
    ```javascript
    function* fibs() {
      var a = 0;
      var b = 1;
      while (true) {
        yield a;
        [a, b] = [b, a + b];
      }
    }
    var [first, second, third, fourth, fifth, sixth] = fibs();
    console.log(sixth);
    // 5
    ```
    
+  对象的解构

  ```javascript
  var robotA = { name: "Bender" };
  var { name: nameA } = robotA;
  console.log(nameA);
  // "Bender"
  
  var { foo, bar } = { foo: "lorem", bar: "ipsum" };
  console.log(foo);
  // "lorem"
  
  var { missing } = {};
  console.log(missing);
  // undefined
  ```

  +  首先指定被绑定的属性，然后紧跟一个要解构的变量。
  +  当属性名与变量名一致时，可以通过一种实用的句法简写。
  +  可以随意嵌套并进一步组合对象解构
  +  解构一个未定义的属性时，得到的值为undefined
  
+  解构值不是对象、数组或迭代器

  ```javascript
  var {blowUp} = null;
  // TypeError: null has no properties（null没有属性）
  
  var {wtf} = NaN;
  console.log(wtf);
  // undefined
  ```

  +  当你尝试解构null或undefined时，你会得到一个类型错误
  +  可以解构其它原始类型，例如：布尔值、数值、字符串，但是你将得到undefined
  
+  当你要解构的属性未定义时你可以提供一个默认值：

  ```javascript
  var [missing = true] = [];
  console.log(missing);
  // true
  ```

+  解构的实际应用
  +  函数参数定义：函数接收一个对象，将不同的实际参数作为对象属性，以避免让API使用者记住多个参数的使用顺序。
  +  配置对象参数：通过默认值实现
  +  与ES6迭代器协议协同使用：`for (var [key, value] of map) { //... }`
  +  多重返回值
  
## 箭头函数（Arrow Functions）
符号 | 含义
--- | ---
`<!--` | 单行注释
`-->` | “趋向于”操作符
`<=` | 小于等于
`=>` | 这又是什么？

+  箭头函数：`=>`，用于lambda语法
+  使用了块语句的箭头函数不会自动返回值，你需要使用return语句将所需值返回。
+  当使用箭头函数创建普通对象时，你总是需要将对象包裹在小括号里。
  +  `puppy => {}`会被解析为没有任何行为并返回undefined的箭头函数。
+  箭头函数没有它自己的`this`值
  +  通过`object.method()`语法调用的方法使用非箭头函数定义，这些函数需要从调用者的作用域中获取一个有意义的`this`值。
  +  其它情况全都使用箭头函数。
  +  箭头函数不会获取它们自己的arguments对象
  +  ES6的方法语法
  
    ```javascript
    // ES6
    {
      ...
      addAll: function addAll(pieces) {
        _.each(pieces, piece => this.add(piece));
      },
      ...
    }
    
    // ===>>>
    
    // ES6的方法语法
    {
      ...
      addAll(pieces) {
        _.each(pieces, piece => this.add(piece));
      },
      ...
    }
    ```

## Symbols

```javascript
// 创建一个独一无二的symbol
var isMoving = Symbol("isMoving");
...
if (!element[isMoving]) {
  smoothAnimations(element);
}
element[isMoving] = true;
```

+  `Symbol`是JavaScript的第七种原始类型：Undefined 未定义，Null 空值，Boolean 布尔类型，Number 数字类型，String 字符串类型，Object 对象类型
+  以symbol为键的属性属性与数组元素类似，不能被类似obj.name的点号法访问，你必须使用方括号访问这些属性。
+  只有当isMoving在当前作用域中时才会生效
+  JavaScript中最常见的对象检查的特性会忽略symbol键，例如：for-in循环，`Object.keys(obj)`，`Object.getOwnPropertyNames(obj)`
+  `Object.getOwnPropertySymbols(obj)`可以列出对象的symbol键；
+  `Reflect.ownKeys(obj)`，会同时返回字符串键和symbol键
+  symbol被创建后就不可变更，你不能为它设置属性（在严格模式下尝试设置属性会得到TypeError的错误）。他们可以用作属性名称，这些性质与字符串类似。
+  每一个symbol都独一无二，不与其它symbol等同，即使二者有相同的描述也不相等
+  symbol不能被自动转换为字符串，这和语言中的其它类型不同。尝试拼接symbol与字符串将得到TypeError错误。通过String(sym)或sym.toString()可以显示地将symbol转换为一个字符串。
+  获取symbol的三种方法
  +  调用`Symbol()`
  +  调用`Symbol.for(string)`，如果同一个描述的symbol已经存在，将返回已存在的symbol对象
  +  使用标准定义的symbol，例如：`Symbol.iterator`。标准根据一些特殊用途定义了少许的几个symbol。

## 集合
+  已经有了一种类似哈希表的东西：对象（Object）。
+  作为查询表使用的对象，不能既支持方法又保证避免冲突。
+  因而，要么得用`Object.create(null)`而非直接写`{}`，要么得小心地避免把`Object.prototype.toString`之类的内置方法名作为键名来存储数据。
+  对象的键名总是字符串（当然，ES6 中也可以是`Symbol`）而不能是另一个对象。
+  没有有效的获知属性个数的方法。
+  纯粹的对象不可遍历，也就是，它们不能配合`for-of`循环或`...`操作符等语法。
+  集合数据的访问，不能再通过属性方式了，只能通过暴露出来的接口（`get`等）。
+  `Set`, `Map`, `WeakMap`, `WeakSet`

## 代理（Proxy）
+  对象的14种内部方法。
+  `var proxy = new Proxy(target, handler);`
+  代理的行为很简单：将代理的所有内部方法转发至目标。简单来说，如果调用`proxy.[[Enumerate]]()`，就会返回`target.[[Enumerate]]()`。
+  句柄对象的方法可以覆写任意代理的内部方法。

## 类（Class），继承
...
+  实例属性，方法？
+  静态属性，方法？
+  `prototype`？

## `let`和`const`
+  ES6之前，只有两种作用域：全局，函数，没有代码块作用域
+  `var`是函数作用域
+  变量提升（hoisting）
+  ES6引入了新的作用域：代码块作用域；`let`和`const`都是这一作用域；

## 模块
+  import, export, default
+  import as, import _, import *
+  export as, export *
