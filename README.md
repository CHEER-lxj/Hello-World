# <你不知道的javascript（上）>读书笔记  (第一部分)


# 第一章
## RHS 与 LHS
- RHS查询：即查询某个变量的值，关心谁是赋值操作的源头。
- LHS查询：找到变量的容器本身，关心赋值操作的目标是谁。 
## 作用域
- 作用域是一套规则，用于确定在何处以及如何查找变量。如果查找的目的是对变量进行赋值，那么使用LSH；如果目的是获取变量的值，会使用RSH。
- 赋值操作会导致LSH查询，=操作符或调用函数时传入参数的操作都会导致关联作用域的赋值操作。
- js引擎首先在代码执行前进行编译，例如 let a = 2; 的声明会被分解为两个独立的操作。
 1. let a 在其作用域中声明新变量。这在代码执行前进行。
 2. a = 2 执行LSH查询并对a赋值。
 
- LSH和RSH都从当前作用域中开始，若有需要，就向上级作用域继续查找，到达全局作用域，无论找到与否都会停止。
- **ReferenceError异常**表示作用域判别失败。**TypeError异常**表示对作用域的判别是成功的，但是执行对结果的操作是非法或不合理的。
- 不成功的RSH会导致抛出**ReferenceError异常**。**非严格模式**下，不成功的LSH引用会导致自动隐式的创建一个全局变量；**严格模式下**，会抛出ReferenceError异常。

# 第二章
## 词法作用域
- 词法作用域是在词法化阶段,定义的作用域。
- **with** 和 **eval** 会修改词法作用域，从而使引擎的优化失效，运行变慢，所以程序中不要使用。

# 第三章
## 函数作用域和块作用域
- 区分函数声明和函数表达式,看 function 关键字出现在声明中的位置(整个声明中的位置)。如果 function 是声明中的第一个词，那么就是一个函数声明，否则就是一个表达式。
- 为变量显示声明块作用域，并对变量进行本地绑定是非常有用的。可以让引擎知道是否要销毁变量。

```
function process(data) {  
	//TODO  
}
{
	let someBigData = {...};
	process(someBigData);
}
var btn = document.getElementById('my_btn');
btn.addEventListener('click', function() {
	console.log('btn clicked');
})
```
# 第四章
## 变量提升
- 函数声明会被提升,但是函数表达式并不会被提升。

```
// 函数声明 代码正常执行
foo();
function foo() {
	console.log('a');   // undefined
	var a = 2;
}

// 函数表达式 抛出异常(抛TypeError异常)
foo();
var foo = function() {
	//...
}
```
**注: 函数表达式抛出的是TypeError异常而不是ReferenceError！函数声明和变量声明都会被提升，但是函数声明提升后会赋值，变量声明不会。且函数会首先被提升**

- 尽量避免在块内声明函数，也不要在一个作用域内重复定义。出现在后面的函数声明是可以覆盖前面的。

# 第五章
## 作用域闭包
- 函数在当前词法作用域之外执行，但是仍可以记住并访问所在的词法作用域，这就是闭包。

```
function foo() {
    var a = 2;
    function bar() {
    	console.log(a);
    }
    return bar;
}
var baz = foo();
baz() // 2
```
**foo()执行后,其返回值(即内部的bar()函数)赋值给变量baz并调用baz()。bar（）可以被正常的执行，但是他是在自己的作用域以外的地方被执行的。foo（）的内部作用域被bar（）占用，所以不会被垃圾回收机制所回收。**

## 模块
模块模式需要具备两个必要条件。
1. 必须有外部的封闭函数，该函数必须至少被调用一次（每次调用都会创建一个新的模块实例）。
2. 封闭函数必须返回至少一个内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

```
// 模块示例
const coolMoudle = () => {
    let someThing = "cool";
    let another = [1, 2, 3];
    const doSomeThing = () => {
    	console.log(something);
    };
    const doAnother = () => {
    	console.log(another.join("!"));
    };
    return {
    	doSomeThing: doSomeThing,
	doAnother: doAnother
    };
}
const foo = coolMoudle();

foo.doSomeThing(); // cool
foo.doAnother(); // 1!2!3
```

# <你不知道的javascript(上)>读书笔记  (第二部分)


# 第一章
## 关于this

关于this需要澄清的两处: 1. this 并不指向函数自身; 2. this并不指向函数的词法作用域。
**this是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。**

例1
```
function foo(num) {
    console.log("foo:" + num);
    this.count++;
}
foo.count = 0;
var i;
for(i = 0; i < 10; i ++) {
    if(i > 5) {
        foo.call(this, i);
    }
}

// foo:6
// foo:7
// foo:8
// foo:9

console.log(foo.count); // 0
console.log(this.count); // NaN
```

例2
```
function foo(num) {
    console.log("foo:" + num);
    this.count++;
}
foo.count = 0;
var i;
for(i = 0; i < 10; i ++) {
    if(i > 5) {
        foo.call(foo, i);
    }
}

// foo:6
// foo:7
// foo:8
// foo:9

console.log(foo.count); // 0
```
# 第二章
## this全面解析
**这一章应该是第二部分最重要的一章,也是最难的一章,需要反复阅读理解。**
### 理解调用位置
![调用位置及调用栈示例](https://github.com/CHEER-lxj/You-do-not-know-JS/raw/master/img/diaoyongweizhi.png)
this是在函数调用时发生绑定,所以需要先理解一下调用位置和调用栈。如上图示例,借助浏览器调试工具，在被调用函数第一行打断点或者添加“debugger;”,则执行时,可以在右侧面板中看到调用栈。调用栈中的第二个元素，就是真正的调用位置。
### 绑定规则
- 默认绑定
无法应用其他规则时的默认规则。**只有当函数运行在非 strict mode 模式时,默认绑定才能绑定到全局对象;在严格模式下,this会绑定到 undefined。在引入第三方库时，要注意兼容此类细节。**
- 隐式绑定
考虑调用位置是否有上下文对象。即函数在调用时如果作为属性被对象obj所包含，隐式绑定规则会把函数调用中的this绑定到这个上下文对象中。但是在传入回调函数时，被隐式绑定的函数会丢失绑定对象。
- 显示绑定
借助call（...）和apply(...)方法实现强制绑定。他们的第一个参数都是一个对象，在使用时，把函数的this强制绑定到这个对象。显示绑定并不能解决this绑定丢失的问题，但是硬绑定可以。
```
function foo(something) {
    console.log(this.a, something);
    return this.a + something;
}
// 辅助函数
function bind(fn, obj) {
    return function() {
        return fn.apply(obj, arguments);
    };
}
let obj = {a: 2};
let bar = bind(foo, obj);
let b = bar(3); // 2 3
console.log(b); // 5
```
**注意此处不要使用箭头函数做示例,箭头函数会改变this绑定。硬绑定的思路是创建一个包裹函数或者一个辅助函数，在创建的函数中，借助apply或者call手动进行一次this绑定，则此后无论怎么调用函数，总会手动进行一次绑定，因此this不会再丢失。ES5内置的bind函数已经实现了硬绑定，可以直接使用。**
- new绑定
在js中，构造函数只是在使用new操作符时被创建的函数，他们并不属于某个类，也不会实例化某个类。使用new调用函数时，会执行下面操作：
1. 创建（构造）一个全新的对象；
2. 新对象会被执行原型连接；
3. 新对象会绑定到函数调用的this；
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。
### 优先级以及this判断
1. 若函数在new中调用,则this绑定这个新创建的对象;
2. 若函数通过call、apply（显示绑定）或者硬绑定，this绑定的是指定的对象。var bar = foo.call(obj2)；
3. 函数在某个上下文中被调用（隐式绑定），this绑定的是那个上下文对象；
4. 若都不是，则使用默认绑定规则。在严格模式下，被绑定到udnefined，否则绑定到全局对象。var bar = foo()。
**注：对于默认绑定来说，决定this绑定对象的并不是调用位置是否处于严格模式，而是函数整体是否处于严格模式。**
### 特例（需要忽略this的情形）
使用情景：当需要使用apply来展开数组时，或者使用bind(...)对函数进行柯里化。通常需要忽略this，此时传入null即可。
```
function foo(a, b) {
    console.log("a:" + a + ",b:" + b);
}
// 把数组"展开"成参数
foo.apply(null, [2, 3])

// 使用bind进行柯里化
var bar = foo.bind(null, 2);
bar(3); // a: 2,b: 3
```
**注: es6中可以使用[...]来展开数组,可以避免不必要的this绑定。但是es6中没有柯里化。**
#### 更安全的使用this
1. 若总是传入null，若代码里引入了第三方库，有可能会把this绑定到全局，从而修改全局变量。**因此，应该传一个空对象，来实现更安全的绑定。**
```
// 新建空对象
var o = Object.create(null);
var bar = foo.bind(o, 2);
bar(3); // a: 2,b: 3
```
2. 编码过程中,会有意无意的构造出函数的"间接引用",此时会使用默认绑定规则。要注意this会被绑定到全局变量。
```
function foo() {
    console.log(this.a);
}
var a = 2;
var o = {a: 3, foo: foo};
var p = {a: 4};

o.foo(); // 3
(p.foo = o.foo)(); // 2

p.foo = o.foo 的返回值是目标函数的引用,因此调用位置是foo()而不是 p.foo() 或者 o.foo(),此时会使用默认绑定。（不要看调用位置是否处于严格模式，而要看函数整体是否处于严格模式）
```
3. 建议让this默认绑定一个除了全局对象和undefined以外的值，既可以实现硬绑定相同的效果，也可以保留隐式绑定或者显示绑定修改this的能力。
### this词法
es6中箭头函数不使用this的四种绑定规则，而是依据外层作用域来决定this。箭头函数会继承外层函数调用的this绑定（无论this绑定到什么）

#### 建议
1. 只使用词法作用域并且完全抛弃错误的this风格的代码；
2. 完全采用this风格，必要时使用bind(...),尽量避免使用 self = this 和箭头函数。

# 第三章 对象

对象有两种形式，声明（文字）形式和构造形式。

文字语法：
```
var myObj = {
    key: value,
    key2: value2
    //...
};
```
构造形式:
```
var myObj = new Object();
myObj.key = value;
```
两种形式生成的对象是一样的。区别：在文字声明中可以添加多个键值对，在构造形式中只能逐一添加。

## 类型
主要类型:
- string
- number
- boolean
- null
- undefined
- object

**注: 简单基本类型(string、boolean、number、null、undefined)本身并不是对象。数组和函数都是对象的一种子类型。**

## javaScript 内置对象
- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error

## es6可计算属性名
在文字形式（对象字面量）中，使用[]包裹一个表达式当属性名。
```
var prefix = "foo";
var myObject = {
    [prefix + "bar"]: "hello",
    [prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```
## 复制对象
潜拷贝只复制一层对象的属性，而深拷贝会复制所有层。潜拷贝，若原对象有引用类型，则复制后的对象与原对象有相同的引用，修改新对象会影响原对象；深拷贝就不会。
潜拷贝更常用，深拷贝可使用JSON来巧妙复制。

```
var newObj = JSON.parse(JSON.stringify(someObj));
```

深拷贝:
```
var obj = { a:1, arr: [2,3] };
var shallowObj = shallowCopy(obj);

function shallowCopy(src) {
  var dst = {};
  for (var prop in src) {
    if (src.hasOwnProperty(prop)) {
      dst[prop] = src[prop];
    }
  }
  return dst;
}
```
## 属性描述符
属性描述符是从es5开始增加的。属性描述符，即直接检测属性特性的方法，比如判断属性是否是只读。创建一个普通对象时，属性描述符会使用默认值。

```
var myObject = {a: 2};
Object.getOwnPropertyDescriptor(myObject, "a");
{
   value: 2,
   writable: true,
   enumerable: true,
   configurable: true
}
```
## 属性存在性与枚举

使用in验证存在性,但是会检验对象及其原型链;使用hasOwnProperty只检验对象是否有这个属性。
```
var myObject = {};
Object.defineProperty(
    myObject,
    "a",
    {enumerable: true, value: 2}
);
Object.defineProperty(
    myObject,
    "b",
    {enumerable: false, value: 3}
);

myObject.propertyIsEnumerable("a"); // true
myObject.propertyIsEnumerable("b"); // false

Object.keys(myObject); // ["a"]
Object.getOwnPropertyNames(myObject); // ["a", "b"]
```

使用in可以判断属性是否存在，但是for...in...循环却不能显示不可枚举的属性值。

**propertyIsEnumerable检查只存在对象上,且满足enumerable的属性;
Object.keys返回数组，包含可枚举属性；
Object.getOwnPropertyNames返回数组，包含所有属性；
Object.keys与Object.getOwnPropertyNames只查找对象直接包含的属性；
in查原型链，hasOwnProperty不查。**

## 遍历

for...in 循环遍历的是对象中的所有可枚举属性。并不是全部。遍历数组下标时采用的是数字顺序，但是**遍历对象属性时的顺序是不确定的，在不同的javaScript引擎中可能不一样。**如果要直接遍历属性值而不是下标，可以使用for...of循环。

# 第四章 混合对象 “类”

## 类理论
面向对象编程强调的是数据和操作数据的行为本质上是相互关联的，因此好的设计模式就是把数据以及他的相关行为打包（封装）起来。在计算机科学中，称为数据结构。
类是一种设计模式。然而在javaScript中，并不存在真正的类。javaScript中的类跟其他语言中的并不一样。

类实际意味着复制。javaScript里，就是对象引用的复制。

传统的类被实例化时，他们的行为会被复制到实例中。类被继承时，行为也会被复制到子类中。

多态（在继承链的不同层次名称相同但是功能不同的函数）看起来似乎是从子类引用父类，但是本质上引用的其实是复制的结果。

javaScript并不会（像类那样）自动创建对象的副本。

混入模式可用来模拟类的复制行为，但是通常使得代码难以维护。

因此，在javaScript中模拟类是得不偿失的。

## 类和实例的关系
“类” 和 “实例” 的关系来源于房屋建造。
类相当于建筑蓝图，实例就是建筑的物理实物。为了获得一个真正可以交互的对象，我们必须按照类来建造（或者说实例化）一个东西，这就是实例。

# 原型
## 属性设置和屏蔽

给对象设置属性并不是添加一个新的属性或者修改现有属性值。如果现有对象中包含这个普通数据访问属性，那么就会修改已有的属性值。如果不包含，就会遍历对象的原型链。若原型链上没有这个属性，就把这个属性添加到对象上；若有，则可能会发生屏蔽。屏蔽，即对象中包含的属性会屏蔽原型链上层的属性。

```
myObj.foo = "bar"
```
屏蔽的三种可能：
1. 在[[Prototype]]链上层存在要赋值的属性并且没有被标记为只读（writable：true），那就会在对象myObj中添加一个新属性，他是屏蔽属性。
2. 如果在[[Prototype]]链上存在要赋值的属性，但是被标记为只读（writable：false），则无法修改已有属性值或者在myObj上创建屏蔽属性。若运行在严格模式下，则会抛出错误。否则，这条赋值语句会被忽略。（不会发生屏蔽）
3. 如果在[[Propotype]]链上层存在foo并且他是一个setter，那就一定会调用这个setter。foo不会被添加到myObj，也不会重新定义foo的这个setter。

**只读属性会阻止[[Prototype]]链下层隐式创建同名属性。且这个限制只存在=赋值中，使用Object.defineProperty就不受影响。使用屏蔽得不偿失，应当尽量避免使用。**

## 原型继承

```
function Foo(name) {
    this.name = name;
}
Foo.prototype.myName = function() {
    return this.name;
}
function Bar(name, label) {
    Foo.call(this, name);
    this.label = label;
}

Bar.prototype = Object.create(Foo.prototype);

Bar.prototype.myLabel = function() {
    return this.label;
}

var a = new Bar("a", "obj a");

a.myName(); // "a"
a.myLabel(); // "obj a"
```

使用 Object.create() 做原型继承，而不使用 Bar.prototype = Foo.prototype 或者 Bar.prototype = new Foo() 是因为这两种都会与预期有出入。使用Object.create()的缺点是，需要抛弃默认的Bar.prototype对象。es6中可以直接修改现有的Bar.prototype对象。使用方法：**Object.setPrototypeOf(Bar.prototype, Foo.prototype);**

1. Bar.prototype = Foo.prototype 并不会创建一个关联到Bar.prototype的新对象,只是让Bar.prototype直接引用新对象。因此当执行类似于Bar.prototype.mylabel = ... 时,会直接修改Foo.prototype对象本身。
2. Bar.prototype = new Foo()的确会创建一个关联到Bar.prototype的新对象。但是它使用了Foo(...)的"构造函数调用"，如果函数Foo有一些副作用，就会影响到Bar()的"后代"。

## Object.create()
Object.create用来做对象关联。他会创建一个新对象并把它关联到指定的对象，而不会引起一些副作用。Object.create(null)会创建一个拥有空[[Prototype]]链接的对象，这个对象无法进行委托，没有原型链。通常被称作“字典”，非常适合用来存储数据。

# 第六章 行为委托

javaScript倡导对象之间的委托，这样不限于垂直的父与子类的关系，而是对象之间可以随意的进行关联。这种设计模式更强大，更灵活。


# 你不知道的JavaScript（中）
# 第一部分 第一章 类型

javaScript是一种弱类型语言,但是会进行强制类型转换。

## 内置类型
javaScript有7种常见类型
- null
- undefined
- boolean
- number
- string
- object
- symbol（ES6新增）

除对象外，统称为基本类型。

使用typeof检验null是不成立的。这是javaScript中已经存在了20多年的一个bug，不会修复了。
typeof null === 'object'; //true

因此需要使用复合类型来检验null: 
```
var a = null;
(!a && typeof a === 'object'); // true
```
**另: 函数是对象的一个子类型,数组也是对象的一种子类型。**
```
typeof function a(){} === "function"; // true
typeof [1,2,3] === 'object'; // true
```

## 值和类型
javaScript中的变量没有类型，值有类型，但是引擎不需要变量总是持有与其初始值相同的类型。

未持有值的变量（未被初始化），会赋给undefined，实际应该为undeclared。javaScript对二者不做区分。

编码时，可以通过typeof或者全局变量（window）来检查未定义的变量，防止报错。但是代码有可能不是运行在浏览器中，比如node.js，所以typeof是个不错的选择。

```
function doSomeThing() {
    var helper = 
        (typeof Feature !== "undefined") ? Feature : function() {/*.. default feature ..*/};
    var val = helper();
    //..
}
```

# 第二章  值

javaScript中的值是有类型的。进行操作时通常有一些小技巧。

- 字符串是一种类数组的类型，没有数组的一些原生方法，但是有需要时，可以转换为数组进行操作。

```
let arrString = 'food'
// 将字符串转化为数组 翻转 再拼接为字符串
arrString = arrString.split('').reverse().join('')
console.log(arrString)
```
在使用es6之前的方法拼接字符串模板时,也可以考虑使用这种方法,会比使用"+"进行拼接要清晰很多。如果要进行很多类数组操作,可以考虑直接初始化为数组

