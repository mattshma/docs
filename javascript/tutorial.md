JavaScript
===

对象
---
对象是 JavaScript 的基本数据类型。它是属性的无序集合，每个属性都是一个Key/Value对。对象除了可以有自有属性(own property)外，还可以从原型对象继承属性(inherited property)。JavaScript对象的属性可以新增修改或删除。 

JavaScript 的属性有一些与之相关的值，称为 property attribute：

- writable attribute  
 是否可以设置该属性的值
 
- enumerable attribute  
 是否可以通过for/in循环返回该属性

- configurable attribute  
 是否可以删除或者修改该属性

除属性外，每个对象还拥有三个相关的 object attribute：

- prototype  
 指向另一个对象。对象的属性继承自它的原型对象
 
- class  
 标识对象类型
 
- extensible flag  
 是否可以向对象添加新属性

JavaScript 使用 **引用** 来操作对象。

### 创建对象

可以通过如下几种方式创建对象。

#### 对象直接量

对象直接量是由若干 Key/Value对组成的映射表。如下：

```
var empty = {};
var point = { x: 0, y: 1};
```

#### new

使用 `new` 后跟随一个构造函数的方法来创建一个对象。如：

```
var o = new Object();
```

#### Object.create()
在说 `Object.create()` 方法之前，先说下原型。每个JavaScript对象（null除外）都和原型对象相关联。所有通过对象直接量创建的对象都具有同一个原型对象，可以使用 `Object.prototype` 来获得对原型对象的引用。通过 `new` 和构造函数创建的对象的原型就是构造函数的 prototype 属性值。

`Object.create()` 的提供两个参数，第一个是对象的原型，第二个是可选的，用来对对象的属性进行进一步的描述。如下：

```
var o1 = Object.create({x:1, y:2});        // o1 继承属性x和y
var o2 = Object.create(Object.prototype);  // 与{}和new Object()一样。
```

原型
---
JavaScript 中类的实现不是基于类继承方式，而是基于原型继承。如果两个实例都是从同一个原型对象上继承属性，则它们是同一个类的实例（duck-typing）。

对于如下代码：
```
var x = 2, b = 1;

function add(x, y) {
    return x + y
}

function sub(x, y) {
    return x - y
}
```

对于上述实现方法，没有做到对于重用代码和模块开发，上述代码显得不够优雅。使用如下方式来美化一下：

```
function Calculator(x, y){
    this.x = x;
    this.y = y;
}

Calculator.prototype = {
    add: function(x, y) {
        return x + y;
    }

    sub: function(x, y) {
        return x - y;
    } 
}
```
以后可以通过new得到一个Calculator对象，然后调用其方法。

### 原型链
对象的原型链由对象的原型继承关系组成。当查找一个对象的属性时，JavaScript会向上遍历原型链，直到找到给定名称的属性为止。若查找到原型链的顶部--Object.prototype 仍没有找到指定的属性，则返回undifined。

以下是mollypages关于原型的一张图：

![JavaScript原型链](../img/js_prototype_chain.jpg)

下面是一个测试代码：

```
function Foo() {}
var foo=new Foo();
alert(foo instanceof Foo);//true
alert(foo instanceof Object);//true
alert(foo instanceof Function);//false
alert(Foo instanceof Function);//true
alert(Foo instanceof Object);//true
```

### \_\_proto\_\_ 与 prototype

`__proto__` 指向对象的内部原型，`prototype` 指向构造器原型。

对于大多数JavaScript实现而言，每一个对象都有一个 `__proto__` 属性，原型链正是基于 `__proto__` 属性来实现的。不过 `__proto__` 不是一个标准的做法，部分浏览器不能直接访问它，不应该出现在代码中。

每个JavaScript函数（除 `Function.bind()` 返回的函数外）都自动拥有一个 `prototype` 属性。该属性的值是一个对象，这个对象包含唯一一个不可枚举的属性`constructor`, `constructor`属性的值是一个函数对象。

```
var F = function(){};           //F是一个函数对象
var p = F.prototype;            //p是F相关联的原型对象
var c = p.constructor;          //c是原型相关联的函数
c === F                         // 即对任意函数有 F.prototype.constructor == F
```

构造函数的原型中存在预先定义好的 `constructor` 属性，这意味着对象继承的`constructor` 属性均指代它们的构造函数。

```
var o = new F();       // 创建类F的一个对象
o.constructor === F;   // true, 对象的constructor属性指代这个类。
```

通过`new`和构造函数创建的对象原型就是构造函数的prototype属性。关于`new`操作符，先看如下代码：

```
function Foo(){}
var foo = new Foo();
```

JavaScript在new的过程中，做了如下操作：

1. 创建类实例  
 这里将一个空对象的 `__proto__` 属性设置为 `F.prototype`
2. 初始化这个实例  
 调用带参的函数F，并将`this` 指定为该实例。
3. 返回实例  

即`var foo = new Foo()`相当于如下代码：

```
var foo = { __proto__: Foo.prototype};
Foo.apply(foo, agruments);
return foo;
```

从上面可以看出，**所有对象的 `__proto__` 都指向其构造函数的 `prototype` 属性**。在JavaScript中函数也是对象，且所有function的基类都是`Function`，因此所有构造函数（包括`Object`）的 `__proto__` 都指向 `Function.prototype`（原型对象）。对象（包括`Function`）都继承自`Object`，由于所有对象的 `__proto__` 属性都指向其构造函数的`prototype` 属性，因此原型对象的 `__proto__` 都指向`Object.prototype`。`Object.prototype`的 `__proto__` 指向 `null` 。

可以看到，`Object` 和 `Function` 相互影响， `Function` 继承自 `Object`，而 `Object.__proto__ === Function.prototype`。

Execution Context
---

每次代码的执行都需要相应的上下文（即运行环境），这个上下文的执行过程分为以下两个阶段：

1. 建立阶段：
 - 建立variable object(VO)
 - 建立Scope Chain
 - 确定this 

2. 代码执行阶段：
 执行函数中的代码

在函数中，VO被表示为活动对象（activation object）

### Scope Chain
一个作用域链由该函数的 activation object 及父类的 variable object 组成。`with`等可以对作用域链产生影响。

注意：**函数对象的作用域链是在定义函数时决定的，而不是在调用时**。 eg：

```
var x = 1;
function echo() {
    console.log(x)
}

function env() {
    var x = 2;
    echo();
}

env();
```

运行结果为`1`. echo()的上下文中name始终为`1`。

根据以上说明，当定义一个函数时，会保存一个作用域链；当调用这个函数时，会创建一个活动对象，并将调用该函数时的每个**形参**都做为该活动对象的属性。再将这个活动对象加到定义时的作用域链最顶端，做为调用该函数时的作用域链。在查找变量x的值时，会从作用域链中的第一个对象开始查找，如果没有找到，则继续查找链上下一个对象，依此类推。

闭包
---

只要外部函数的变量都存在，那么从内部函数访问这些外部数据没什么特别的；那什么是闭包呢？简单来说闭包是外部函数执行完后其资源不被释放，因为其内部函数的执行仍需要引用外部函数的资源。因此闭包有两个条件：1. 在一个函数内定义另外一个函数，2. 返回（return）内部函数。

对闭包更深入的解释如下：

> 1. 有外部函数a和内部函数b，当定义函数a的时候，js解释器会将函数a的作用域链(scope chain)设置为定义a时a所在的“环境”，如果a是一个全局函数，则scope chain中只有window对象。
> 2. 当函数a执行的时候，a会进入相应的执行环境(excution context)。
> 3. 在创建执行环境的过程中，首先会为a添加一个scope属性，即a的作用域，其值就为第1步中的scope chain。即a.scope=a的作用域链。
> 4. 然后执行环境会创建一个活动对象(call object)。活动对象也是一个拥有属性的对象，但它不具有原型而且不能通过JavaScript代码直接访问。创建完活动对象后，把活动对象添加到a的作用域链的最顶端。此时a的作用域链包含了两个对象：a的活动对象和window对象。
> 5. 下一步是在活动对象上添加一个arguments属性，它保存着调用函数a时所传递的参数。
> 6. 最后把所有函数a的形参和内部的函数b的引用也添加到a的活动对象上。在这一步中，完成了函数b的的定义，因此如同第3步，函数b的作用域链被设置为b所被定义的环境，即a的作用域。

如果内部函数没有被返回，即内部函数和外部函数互相引用，在执行完外部函数后，两个函数都会被GC回收。

### 闭包的应用

闭包可以用于以下场景：

- 隐藏内部变量

```
counter = (function(){
    var count = 0
    return function(){
        console.log(count += 1)
    }
})()
counter()          // 1
counter()          // 2
```

- curry
 有时候对于某些函数，仅仅需要传递部分参数，如下：

```
function add(x, y) {
return x + y;
}

add(5, 4);             // 9

// step 1 -- 传递部分参数
function add(5, y) {  // SyntaxError: missing formal parameter
    return 5 + y;     
}
// step 2 -- 传递参数
function add(5, 4) {  // SyntaxError: missing formal parameter
    return 5 + 4;
}  
```

这里可以使用嵌套函数，将参数分部分传入这些函数中，如下

```
function add(x, y) {
    var oldx = x, oldy = y;
    if (typeof oldy === "undefined") { // partial
        return function (newy) {
            return oldx + newy;
        };
    }
    // full application
    return x + y;
}  
```

这样的处理方式叫做curry化。

上面这个例子可以写作

```
var add = function(a) {
    return function(b) {
        return a + b;
    }
}

add(1)(2);   //3
```

Reference
---

- [JavaScript Object Layout](http://www.mollypages.org/misc/js.mp)
- [How prototypal inheritance really works](http://blog.vjeux.com/2011/javascript/how-prototypal-inheritance-really-works.html)
- [JavaScript核心](http://www.cnblogs.com/TomXu/archive/2012/01/12/2308594.html)
- [Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype)
- [JavaScript curry](http://blog.csdn.net/qq838419230/article/details/8535978)
