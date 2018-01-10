# 第二部分：设计模式

# 4. 单例模式

定义：保证一个类仅有一个实例，并提供了一个访问它的全局访问点。

```javascript
// 实现1：
var Animal = function(name){
    this.name = name;
};
Animal.prototype.getName = function(){
    return this.name;
};

Animal.getInstance = function(name){
    if (!this.instance){
        this.instance = new Animal(name);
    };
    return this.instance;
};
var a = Animal.getInstance('a');
var b = Animal.getInstance('b');
alert(a === b);

// 实现2：
var getSingle = function(fn){
    var result;
    return function(){
        return result || (result = fn.apply(this,arguments));
    }
};
var createLoginLayer = function(){
    var div = document.createElement('div');
    div.innnerHTML = 'LOGIN';
    div.style.display = 'none';
    document.body.appendChild(div);
    return div;
};
var singleLoginLayer = getSingle(createLoginLayer);
```

# 5. 策略模式

定义：定义一系列的算法，把他们一个个封装起来，并且使他们可以相互替换。

# 6. 代理模式

不方便访问某个对象时，使用代理来控制对象的访问。

# 7. 迭代器模式

提供一种方法顺序访问聚合中的各个对象

# 8. 发布-订阅模式

定义对象间的一对多的依赖关系，当一个对象状态发生变化时，所有依赖于它的对象都得到通知。

# 9. 命令模式

有时候需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道被请求的操作是什么。消除请求发送者和请求接收者之间的耦合。（回调）

# 10. 组合模式

将对象组合成树状结构，以表示整体-部分之间的结构。
- 组合模式不是父子关系
- 对叶对象操作的一致性

# 11. 模板方法模式

平行的子类，子类之间有一些相同的行为，也有不同的行为。如果相同和不同的行为混合在各个子类的实现中，说明相同行为在各个子类中会重复。
模板方法模式中，将子类实现中相同的部分上移到父类中，将不同的部分留待子类来实现。

# 12. 享元模式

享元模式的核心是运用共享技术来有效支持大量细粒度的对象。

# 13. 职责链模式

使多个对象连成一条链，并沿着这条链传递请求，知道有个对象处理它为止。

# 14. 中介者模式

增加一个中介者对象，所有的相关对象都通过中介者对象来通讯，而不是互相引用。如果一个对象发生改变时，只需要通知中介者对象即可。

# 15. 装饰者模式

在不改变对象自身的情况下，为对象“动态”的添加职责的方式叫装饰者模式。
和继承相比，继承增强了超类和子类间的强耦合性、是子类数量呈爆炸式增长，而装饰者是“即用即付”。

# 16. 状态模式

区分事物内部的状态，事物内部的状态的改变往往会带来事物行为的变化。

# 17. 适配器模式

解决两个软件实体间的接口不兼容的问题。（坏的办法：修改老接口）

Wrapper、Adapter

```javascript
var googleMap = {
    show: function() {
        console.log('渲染谷歌地图')
    }
};
var baiduMap = {
    display: function() {
        console.log('渲染百度地图')
    }
};
var renderMap = function(map) {
    if (map.show instanceof Function) {
        map.show()
    }
};
var baiduAdapter = {
    show: function(){
        return baiduMap.display();
    }
};
renderMap(googleMap);
renderMap(baiduMapAdapter);
```