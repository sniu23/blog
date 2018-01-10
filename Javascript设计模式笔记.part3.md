# 第三部分：设计原则与编程技巧

# 18. 单一职责原则SPA

单一职责原则更多被运用在对象或方法级别上。**一个对象（或方法）只做一件事情。**

单一职责原则被定义为“引起变化的原因”。如果我们有两个动机去改写方法，那么这个方法就有两个职责。

SPA在很多设计模式中都有应用，如代理模式、迭代器模式、单例模式、装饰者模式。

正确的分离职责并不是件容易的事情。

当我们按照职责把对象分解成更小的粒度时，实际上也增加了这些对象之间相互关联的难度。

# 19. 最少知识原则LKP（迪米特法则）

最少知识原则说的是一个软件实体应尽可能少地与其他实体发生相互关系。这里的软件不仅仅包括系统、类、模块、函数、变量等。

*一个将军需要挖掘一些散兵坑，下面是完成任务发的一种方式：将军通知上校让它叫来少校，然后少校找来上尉，并让上尉通知一个军士，最后军士找来一个士兵，然后命令士兵挖掘一些散兵坑。* 

对象与对象耦合在一起，有可能会降低他们的可复用性。

如果两个对象之间不必彼此直接通讯，那么这两个对象就不要发生直接的相互联系。常见的做法是引入一个第三者对象，来承担这些对象之间的通讯作用。

LKP在设计模式中的体现有：中介者模式、外观模式。

# 20. 开放-封闭原则

在面向对象程序设计中，开放-封闭原则是**最重要**的一条原则。

**Bertrand Meyer在《Object-Oriented Software Construction》提出：「软件实体（类、模块、函数）等应该是可用扩展的，但是不可用修改。」**

*肥皂的故事： 有一家生产肥皂的企业，从欧洲巨资引进了一条生产线，这条产线可用自动完成从原材料加工到包装成箱的整个流程，但美中不足的是，生产出来的肥皂有一定的空盒率。于是，老板从欧洲找来一支专家团队，又花费巨资改造这条生产线，终于解决了生产出空盒肥皂的问题。另外一家企业也引入了这条生产线，同样也遇到了空盒的问题。但他们的解决办法很简单，用一个大风扇在产线旁边吹，把空盒吹跑。*

通过封装的办法，可以把系统中稳定不变的部分和容易变化的部分隔离开来。

办法：
- **用对象的多态性消除条件分支**
- **放置挂钩**
- **使用回调函数**

设计模式中的开放-封闭原则：发布-订阅模式、模板方法模式、策略模式、代理模式、职责链模式。（模板方法模式、策略模式是一对竞争者）

没有人可以“未卜先知”，接收第一次的愚弄。

# 21. 接口和面向接口编程

**接口是对象能响应的请求的集合。**

从过程来看，“面向接口编程”其实是“面向超类型编程”。

静态类型语言中：
接口是某些提供的关键字，如JAVA中的interface。interface关键字可用产生完全抽象的类，这个完全抽象的类用来表示一种契约，专门负责建立类与类之间的联系。抽象类有2个作用：
- 向上转型
- 建立一些契约
- 相对与单继承的抽象类，一个类可以实现多个interface.

javascript中：
因为javascript是动态类型语言，类型本来就是个模糊的概念，所以不需要利用抽象类或interface给对象进行“向上转型”。很少人在javascript中关心对象的真正类型。*（在一些静态类型语言中，对象类型之间的解藕非常重要，甚至有一些设计模式是专门隐藏对象的真正类型）*，如此以来，接口在javascript中的最大作用就退化到了检查代码的规则性。比如：检查某个对象是否实现了某个方法、检查是否给函数传入了预期类型的参数。

```javascript
function show(obj){
    obj.show();
};
var myObject = {}; // 没有show方法
// 或者
var myObject = {
    show: 1 // show不是方法 
};
show(myObject);

// 增加防御性代码
function show(obj){
    if (obj && typeof obj.show === 'function') {
        obj.show();
    }
}
// 实现2
function show(obj){
    try {
        obj.show()
    } catch (e) {

    }
}

// 鸭子类型
Object.prototype.toString.call([]) === '[Object Array]'
```

## 21.1 鸭子类型：

*如果它走起路来像鸭子，叫起来也像鸭子，那么它就是鸭子。*

利用鸭子类型的思想，不必借助超类型的帮助，就能在动态类型语言中轻松实现本章提到的设计原则：面向接口编程，而不是面向实现编程！！！

Q：使用TypeScript编写？

# 22. 代码重构

一些不可忽略的细节建议。

1. 提炼函数，避免&分解出现超大的函数。独立的函数有利于代码复用或覆写
2. 函数拥有一个良好命名，它本身就起到一个注释的作用
3. 把条件分支语句提炼成函数
4. 提前让函数退出代替嵌套条件分支
5. 传递对象参数代替过长的参数列表
6. 尽量减少参数的数量，不传冗余或无用的参数
7. 少用三目运算符（代码可读性差）
8. 合理使用链式调用（链式难以调试，还是少用吧）
9. 退出多重循环时，尽量使用return，而不是循环标记

```javascript
// 1. 需要独立的函数
var gatUserInfo = function(){
    ajax('http://xx.com/userInfo',function(data){
        console.log('userId:'+data.userId);
        console.log('userName:'+data.userName);
    })
};
// 独立后：
var gatUserInfo = function(){
    ajax('http://xx.com/userInfo',function(data){
        printDetail(data);
    })
};
var printDetail = function(data){
    console.log('userId:'+data.userId);
    console.log('userName:'+data.userName);
};

// 3. 把条件分支语句提炼成函数
var getPrice = function(price){
    var date = new Date();
    if (date.getMonth()>=6 && date.getMonth()<=9){
        return price * 0.8;
    };
    return price;
};
// 提炼分支
var isSummer = function(){
    var date = new Date();
    return date.getMonth()>=6 && date.getMonth()<=9;
}

// 4. 提前让函数退出代替嵌套条件分支
var del = function(obj){
    var ret;
    if (!obj.isReadOnly){
        if (obj.isFolder){
            ret = deleteFolder(obj);
        } else if (obj.isFile){
            ret = deleteFile(obj);
        }
    }
    return ret;
};
// 提前退出代替嵌套条件
var del = function(obj){
    var ret;
    if (!obj.isReadOnly){
        return;    
    }
    if (obj.isFolder){
        return ret = deleteFolder(obj);
    }
    if (obj.isFile){
        return ret = deleteFile(obj);
    }
};

// 6. 尽量减少参数的数量，不传冗余或无用的参数
var draw = function(width, height, square){};
// 消除冗余参数 square可以算出
var draw = function(width, height){};

// 9. 退出多重循环时，尽量使用return，而不是循环标记
var func = function(){
    var flag = false; //循环标记
    for (var i=0; i<10; i++){
        for (var j=0; j<10; j++){
            if (i*j>30){
                flag = true;
                break;
            }
        }
        if (flag === true){
            break;
        }
    }
}
// 重构
var print = function(i){
    console.log(i);
};
var func = function(){
    for (var i=0; i<10; i++){
        for (var j=0; j<10; j++){
            if (i*j>30){
                flag = true;
                return print(i);
            }
        }
    }
};
```