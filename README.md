# <你不知道的javascript(上)>读书笔记  (第一部分)


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
