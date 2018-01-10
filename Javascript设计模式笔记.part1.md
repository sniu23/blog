*《javascript设计模式与开发实践》学习笔记*

# 第一部分：基础知识

# 1. 面向对象的Javascript

**设计模式：从面向对象的角度出发，通过对**封装、继承、多态、组合**等技术的反复利用，提炼出一些可反复使用的对象设计技巧。**

设计模式往往体现了语言的不足、对语言的补充; 如果需要很多设计模式，不如找一门更好的语言。

Javascript语言特征：
- 通过原型委托方式实现对象与对象之间的继承
- 动态类型语言
- 高阶函数

## 1.0 鸭子类型概念

*鸭子类型概念： 如果它走起路来像鸭子，叫起来也像鸭子，那么它就是鸭子。*

**引申：动态类型语言中，当我们调用一个对象的某个方法时，我们不用关心该对象是否被设计为拥有该方法**

动态类型语言优缺：
- 类型不匹配错误在编译时无法发现，变量类型要到程序运行时确定
- 代码更简洁（无需依照强契约编程）

```javascript
// 动态类型语言中，利用鸭子类型类型思想，我们不必借助超类型的帮助，就能轻松实现一个原则：
// “面向接口编程，而不是面向实现编程”
// 静态类型语言中，往往需要抽象类或接口等将对象进行向上转型。

// 如果一个对象有push和pop方法，它就可用被当作栈使用，
// 如果一个对象有length属性，也可以依照下标存取属性，那么它也可用作为数组使用。
var dock = {
    duckSinging: function() {
        console.log('嘎嘎嘎');
    }
}

var chicken = {
    duckSinging: function() {
        console.log('嘎嘎嘎');
    }
}

var choir = [] //合唱团

var joinChoir = function(animal) {
    // 动态类型语言的好处：不需要做类型检查（dock和chichen没有继承自animal）
    if (animal && typeof animal.duckSinging === 'function') {
        choir.push(animal)
        console.log('恭喜加入合唱团')
        console.log('合唱团已有成员数量：'+choir.length)
    }
}

joinChoir(duck)
joinChoir(chicken)
```

## 1.1 多态

**多态的含义：同一操作作用在不同的对象身上，可用产生不同的解释和不同的执行结果。**

**多态背后的思想是将“做什么”和“谁去做以及怎样做”分离开来（不变与变化）。**

```javascript
// 坏的实现
var makeSound  = function(animal) {
    if (animal instanceof Duck) {
        console.log('嘎嘎嘎')
    } else if (animal instanceof Chicken) {
        console.log('咯咯咯')
    }
}
var Duck = function(){}
var Chicken = function(){}

// 好的实现
var makeSound = function(animal){
    animal.sound()
}
var Duck = function(){}
Duck.prototype.sound = function(){
    console.log('嘎嘎嘎')
}
var Chicken = function(){}
Chicken.prototype.sound = function(){
    console.log('咯咯咯')
}

// 执行
makeSound(new Duck())
makeSound(new Chicken())

```

*Martin Fowler《重构：改善既有代码的设计》——
多态的最根本好处在于，你不必再像对象询问“你是什么类型”，而后根据得到的答案调用对象的某个行为——你只管调用该行为就是了，其他的一切多态机制都会微你安排妥当。*


```javascript
var googleMap = {
    show: function() {
        console.log('渲染谷歌地图')
    }
}
var renderMap = function() {
    googleMap.show()
}
renderMap()

// 新的需求加入
var baiduMap = {
    show: function() {
        console.log('渲染百度地图')
    }
}

// 坏的扩展
var renderMap = function(type) {
    if (type === 'google') {
        googleMap.show()
    } else if (type === 'baidu') {
        baiduMap.show()
    } 
}
renderMap('google')
renderMap('baidu')

// 好的扩展
var renderMap = function(map) {
    if (map.show instanceof Function) {
        map.show()
    }
}
renderMap(google)
renderMap(baidu)

```

## 1.2 封装

封装的目的是信息隐藏。

```javascript
// 封装数据
// 很多语言提供private、public、protected等关键字来提供不同的访问权限
// javascript只能依赖变量的作用域来实现封装特性
var myObject = (function(){
    var __name = 'sven';
    return {
        getName: function() {
            return __name;
        }
    }
})();

console.log(myObject.getName()) // sven
console.log(myObject.__name)  //undefined
// 封装不仅仅是隐藏数据，还包括隐藏实现细节等。
// 如果是静态类型语言有时候还需要隐藏对象的类型：工厂方法模式、组合模式等
```

设计模式即“找到变化并封装之”。从意图上讲，设计模式分别被划分为创建型模式、结构型模式、行为型模式。

拿创建型模式讲，要创建一个对象就是一个抽象行为，而据他创建什么对象则是可用变化的。创建型模式的目的就是封装创建对象的变化。而结构型模式封装的是对象之间的组合关系。行为型模式封装的是对象的行为变化。

## 1.3 原型继承

面向对象编程中：以类为中心，类和对象的关系就像铸模和铸件，对象总是从类中创建。

原型编程思想中：类不是必须的，对象也未必需要从类中创建而来，对象可以通过**克隆**另一个对象得到的。

```javascript
var Plane = functin() {
    this.blood =100;
    this.attackLevel =1;
    this.defenseLevel = 1;
};

var plane = new Plane();
plane.blood =500;
plane.attackLevel =10;
plane.defenseLevel = 7;

// 原型模式为创建对象提供了另一种方式，通过克隆对象，我们就不用关系对象的具体类型名称。
// 在javascript这种类型模糊的语言中，创建对象非常容易，也不存在类型耦合的问题。
var clone = Object.create(plane);
console.log(plane.blood) //500
console.log(plane.attackLevel) //10
console.log(plane.defenseLevel) //7

// 在不支持 Object.create方法的浏览器中，可用使用下面代码：
Object.create = Object.create || function(obj) {
    var F = function(){};
    F.prototype = obj;
    return new F();
}
```

原型编程的一些基本规则：
- 所有的数据都是对象
- 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型克隆它
- 对象会记住它的原型
- 如果对象无法响应某个请求，它会把这个请求委托给它的原型

对于Javascript来讲，我们一直说的“对象的原型”，只能讲“对象的构造器有原型”。

```javascript
// Javascript 中的根对象是： Object.prototype对象。 
// Object.prototype是一个空对象。
// Javascript 中的每一个对象实际都是从 Object.prototype克隆而来，也就是他们的原型。
var obj1 = new Object();
var obj2 = {};
console.log(Object.getPrototypeOf(obj1) === Object.prototype); //true
console.log(Object.getPrototypeOf(obj2) === Object.prototype); //true

// Javascript 中没有类的概念
// Person不是类，而是函数构造器。（看起来像个类）
// Javascript 中函数既可用被作为普通函数使用，还可用作为构造器被调用。当使用 new来调用时，此时函数就是构造器。
// 用 new来创建对象的过程，实践也是先克隆一个Object.prototype对象，再进行其他操作。
function Person(name) {
    this.name = name;
};
Person.prototype.getName = function() {
    return this.name;
}
var a = new Person('sven');

// 我们一直说的“对象的原型”，对于 Javascript来并不能这么讲，而只能讲“对象的构造器有原型”。
// Javascript 提供给对象一个叫__proto__ 的隐藏属性，它默认会指向对象的构造器的原型对象。
// 即 {Constructor}.prototype
var a = new Object();
console.log(a.__proto__ === Object.prototype)
```

```javascript
// 常见用法1：
// 虽然Javascript的对象最初都是从Object.prototype克隆而来，但对象构造器的原型不局限于Object.prototype。
// 当对象a需要借用对象b的能力时，可用有选择性的把对象a的构造器的原型指向对象b，从而达到继承的效果。
var obj = {name:'sven'};

var A = function(){};
A.prototype = obj;

var a = new A();
console.log(a.name); //sven

// 常见用法2：
//当我们期望得到一个“类”继承自另外一个“类”的效果时：
var A = function(){};
A.prototype = {name: 'sven'};

var B = function(){};
B.prototype = new A();

var b = new B();
console.log(b.name); //sven
```

# 2. this、call和apply

## 2.1 this

Javascript中的this通常指向一个对象，具体指向哪个对象是在运行时基于函数的执行环境动态绑定的，而非函数被申明时的环境。

```javascript
// 1. 作为对象的方法被调用时，this指向该对象。
var obj = {
    a: 1,
    getA: function() {
        alert(this === obj) //true
        alert(this.a) //1
    }
};
obj.getA();

// 2. 作为普通函数调用，通常指向全局对象。
window.name = 'globalName';
var getName = function() {
    return this.name;
};
console.log(getName()); //globalName

// 2.1 内部变量保存this的引用

// 3. 作为构造器调用，通常指向返回的对象
// 构造器和普通函数一模一样，区别在于被调用的方式（new运算符）
var Myclass = function() {
    this.name = 'sven';
};
var obj = new Myclass();
alert(obj.name); //sven

// 3.1 构造器显式返回object类型的对象，那么new的是构造器返回这个对象。
var Myclass = function() {
    this.name = 'sven';
    return {name: 'anne'};
};
var obj = new Myclass();
alert(obj.name); //anne

// 3.2 构造器不显式返回任何数据，或者返回一个非对象类型的数据：
var Myclass = function() {
    this.name = 'sven';
    return 'anne';
};
var obj = new Myclass();
alert(obj.name); //sven

// 4. Function.prototype.call 或 Function.prototype.apply 调用
var obj1 = {
    name: 'sven',
    getName: function() {
        return this.name;
    }
};
var obj2 = {
    name: 'anne'
};
console.log(obj1.getName()); //sven
console.log(obj1.getName.call(obj2)); //anne

// 5. 丢失了的this
var obj1 = {
    name: 'sven',
    getName: function() {
        return this.name;
    }
};
console.log(obj1.getName()); //sven

var getName2 = obj1.getName;
console.log(getName2); //undefined
```

## 2.2 call、apply

Function.prototype.call 或 Function.prototype.apply 的作用是一模一样的。
- apply接收2个参数（函数体内this对象的指向，导入函数体的参数集合）
- call接收参数不固定（第一个：函数体内this对象的指向，第二个开始：导入函数体的参数）
- javascript的参数在内部就是用一个数组表示，故apply要比call的执行效率高

```javascript
// 1. 改变this的指向
var obj1 = {
    name: 'sven'
};
var obj2 = {
    name: 'anne'
};
window.name = 'window';
var getName = function() {
    alert(this.name);
};
getName(); //window
getName.call(obj1); //sven
getName.call(obj2); //anne

// 2. Function.prototype.bind
Function.prototype.bind = function(context) {
    var self = this; //保存原函数
    return function() { //返回新的函数
        // 执行时，会把之前传入的context当作返回函数体内的this
        return self.apply(context,arguments);
    }
};
var obj = {
    name: 'sven'
};
var func = function() {
    alert(this.name) //sven
}.bind(obj);  //TMD！！！
func();

// 3. 借用其他对象的方法
var a = {};
Array.prototype.push.call(a, 'first');
alert(a.length); 1
alert(a[0]); //first

// 3.1 借用构造函数
var A = function(name) {
    this.name = name;
};
var B = function() {
    A.apply(this, arguments);
};
B.prototype.getName = function() {
    return this.name;
};
var b = new B('sven');
console.log(b.getName()); //sven
```

# 3. 闭包和高阶函数

每一个语言都有它自己的语言特征。javascript中，很多设计模式都是通过闭包或者高阶函数实现的。相对于模式的实现过程，我们更关注的是模式可以帮助我们完成什么。

## 3.1 闭包

对于javascript来说，闭包非常难懂。它的形成跟变量的作用域以及变量的生命周期密切相关。

```javascript
// 变量的作用域：
// 如果在函数中声明变量没有带上关键字var，这个变量就会形成全局变量。这样很容易造成冲突。
// 在函数中用var声明变量，此时即为局部变量，只能在函数体内访问到。
var a = 1;
var fun1 = function() {
    var b = 2;
    var fun2 = function() {
        var c = 3;
        alert(b); //2
        alert(a); //1
    };
    func2();
    alert(c); // Uncaught ReferenceError: c is not defined
};
fun1();

// 作用1: 封闭变量
var mult = (function(){
    var cache = {};
    var calculate = function(){
        var a =1;
        for (var i=0, l=arguments.length; i<l; i++){
            a = a * arguments[i];
        };
        return a;
    };
    return function(){
        var args = Array.prototype.join.call(arguments,',');
        if (args in cache) {
            return cache[args];
        }
        return cache[args] = calculate.apply(null, arguments);
    }
})() 
```

```javascript
// 变量的生命周期：
// 对于函数内用var声明的局部变量来说，他们会随着函数调用的结束而被销毁。
var func = function() {
    var a = 1; //函数调用后a将被销毁。
    alert(a);
};
func(); 

// 作用2: 延续局部变量的寿命
var func = function() {
    var a = 1;
    return function() {
        a++;
        alert(a);
    };
};
// f返回了一个匿名函数的引用，它可用访问到func()被调用时产生的环境，而局部变量a就一直处在这个环境中。
var f = func();  // 非常危险！！！
f(); //2
f(); //3
f(); //4
```

**过程与数据的结合是形容面向对象中的“对象”经常使用的表达。对象以方法的形式包含了过程，而闭包则是在过程中以环境的形式包含了数据。通常用面向对象思想能实现的功能，用闭包也能实现。反之亦然。**

```javascript
// 实例1：闭包的写法
var extent = function() {
    var value = 0;
    return {
        call: function() {
            value++;
            console.log(value);
        }
    }
};
var ext = extent();
ext.call() //1
ext.call() //2
ext.call() //3

// 实例2：面向对象的写法
var extent = {
    value: 0,
    call: function() {
        this.value++;
        console.log(this.value);
    }
};
extent.call() //1
extent.call() //2
extent.call() //3

// 实例3：面向对象的另一种写法
var Extent = function(){
    this.value = 0;
};
Extent.prototype.call = function() {
    this.value++;
    console.log(this.value);
};
var extent = new Extent();
extent.call() //1
extent.call() //2
extent.call() //3
```

局部变量本应该在函数退出时被解除引用，但如果局部变量被封闭在闭包形成的环境中，那么这个局部变量就会一直生存下去。因为有的变量可能在以后还需要使用，把这些变量放在闭包中或者全局中，对内存的影响是一样的。

闭包和内存泄漏相关的是，使用闭包的同时很容易形成循环引用，如果闭包的作用域中保存着一些DOM节点，就有可能造成内存泄漏（在IE中，由于BOM或DOM中的对象是使用C++以COM对象的形式实现的，COM对象垃圾收集机制采用的是引用计数策略，如果两个对象之间形成了循环引用，那么两个对象都无法回收），把循环引用的变量设置为null即可解决。

## 3.2 高阶函数

高阶函数具备以下条件之一即是：
- 函数作为参数被传递
- 函数作为返回值被输出

**函数式编程：代表运算过程是可延续的，分离业务中的变化与不变的部分**

### 3.2.1 函数作为参数传递

```javascript
// 1. 回调函数
var appendDiv = function(callback) {
    for (var i=0; i<100; i++) {
        var div = document.createElement('div');
        div.innerHTML = i;
        document.body.appendChild(div);
        if (typeof callback === 'function') {
            callback(div);
        }
    }
};
appendDiv(function(node){
    node.style.display = 'none';
});

// 2. Array.prototype.sort
[1,4,3].sort(function(a,b){
    return a - b;
}); //[1,3,4] 
[1,4,3].sort(function(a,b){
    return b - a;
}); //[4,3,1] 
```

### 3.2.2 函数作为返回值被输出

```javascript
// 1. 判断数据的类型
var isType = function(type){
    return function(obj){
        return Object.prototype.toString.call(obj) === '[Object ]'+type+']';
    }
};
var isString = isType('String');
var isArray = isType('Array');
var isNumber = isType('Number');
console.log(isArray([1,2,3])); //true

// 上面例子再次封装
var Type = {};
for (var i=0, type; type=['String','Array','Number'][i++];){
    (function(type){
        Type['is'+type] = function(obj){
            return Object.prototype.toString.call(obj) === '[Object ]'+type+']';
        }
    })(type)
};
Type.isArray([]); //true
Type.isString('str'); //true

// 2. getSingle
var getSingle = function(fn){
    var ret;
    return function(){
        return ret || (ret = fn.apply(this, arguments));
    }
}
```

### 3.2.3 高阶函数实现AOP

**AOP面向切面编程的主要作用是把一些和核心业务逻辑无关的功能抽离出来，包括日志统计、安全控制、异常处理等，然后在通过“动态植入”的办法渗入业务逻辑模块中去。这样既保证了核心业务逻辑的纯净和高内聚性，又可以方便的复用日志统计等功能模块。**

在JAVA等语言中要实现AOP需要通过反射和动态代理机制等，而在javascript这种动态类型的语言中就相对简单、实现技术很多。

```javascript
// 通过扩展 Function.prototype来实现AOP
Function.prototype.before = function(beforfn){
    var __self = this; // 保存原函数的引用
    return function(){ // 返回包含了原函数和新函数的代理函数
        beforefn.apply(this,arguments); // 执行新函数，修正this
        return __relf.apply(this, arguments); //执行老函数
    }
};
Function.prototype.after = function(afterfn){
    var __self = this; 
    return function(){
        var ret = __self.apply(this, arguments);
        afterfn.apply(this,arguments); 
        return ret; 
    }
};
var func = function(){
    console.log(2);
};
func = func.before(function(){
    console.log(1)
}).after(function(){
    console.log(3)
});
func(); // 1 2 3
```

### 3.2.4 高阶函数的其他应用方面

```javascript
// 1. currying 函数柯里化
//（又称部分求值）
var currying = function(fn){
    var args = []
    return function(){
        if (arguments.length === 0) {
            return fn.apply(this.args)
        } else {
            [].push.apply(args,arguments)
            return arguments.callee
        }
    }
};
var cost = (function(){
    var money = 0
    retrun function(){
        for (var i=0, l=arguments.length; i<l; i++){
            money += arguments[i]
        }
        return money
    }
})();
var curryCost = curring(cost);
curryCost(100); // 未真正求值
curryCost(200); // 未真正求值
curryCost(300); // 未真正求值
alert(curryCost()); //600

// 2. uncurrying
// 在javascript中，当我们调用一个对象的某个方法时，其实不用关心该对象原本是否被设计是否拥有此方法。同理，一个对象也未必只能使用它自身的方法。call和apply就可以完成借用。
// 比如，我们经常让类数组借用Array.prototype的方法。
(function(){
    Array.prototype.push.call(arguments,4); //arguments借用Array.prototype.push
    console.log(arguments);
})(1,2,3) // [1,2,3,4]
// 以上实例，使用call和apply可以把任意对象当作this传入某个方法。
// 把泛化this的过程提取出来，就是uncurrying。
Function.prototype.uncurrying = function(){
    var self = this; // Array.prototype.push
    return function() {
        var obj = Array.prototype.shift.call(arguments);
        return self.apply(obj, arguments);
    };
};
var push = Array.prototype.push.uncurrying();
var obj = {
    "length":1,
    "0":1
};
push(obj,2);
cosole.log(obj); // {0:1, 1:2, length:2}

// 3. 函数节流
// 有些场景中，函数被触发调用的频率太高了。比如：
// window.onresize, mousemove, 上传进度等

// 4. 分时函数

// 5. 惰性加载函数
// 屏蔽浏览器之间实现的差异，通用的事件绑定函数

```

