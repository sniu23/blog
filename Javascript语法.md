
### Array.sort(sortby)

```javascript
// 默认以字符编码排序
var array1 = [e,a,d,c,b];
array1.sort(); // a,b,c,d,e
var array1 = [0,1,5,10,15];
array1.sort(); // 0,1,10,15,5

// sortby: 指定排序对比函数，可选
// 该函数要比较两个值，然后返回一个说明这两个值相对顺序的数字。
// 比如：两个参数a和b，返回值如下：
// 若：a<b，排序后a出现在b后面，则返回一个小于0的数字;
// 若：a=b，返回0;
// 若：a>b，排序后a出现在b前面，则返回一个大于0的数字;
[1,4,3].sort(function(a,b){
    return a - b;
}); // [1,3,4]
[1,4,3].sort(function(a,b){
    return b - a;
}); // [4,3,1]
```

### 原型继承

```js
// 母亲
var Mother = function(age){
    this.age = age || 60;
    this.hobby = ['runing','TV'];
};
Mother.prototype.showAge = function(){
    console.log(this.age);
};
// 大儿子
var Son = function(name,age){
    this.base = Mother; //写法1
    this.base(age);
    this.name = name;
};
Son.prototype = new Mother;
var s1 = new Son('wa',30);
s1.showAge();
// 二女儿
var Girl = function(name,age){
    Mother.call(this,age);  //写法2,推荐
    this.name = name;
};
Girl.prototype = new Mother;
var g2 = new Girl('nie',18);
g2.showAge();
// 大外孙子
var WaiSun = function(name,age){
	Girl.call(this,name,age);
};
WaiSun.prototype = g2;  //g2 和 new Girl的差别
var w1 = new WaiSun('bao',0.6); 
w1.showAge();
WaiSun.prototype.walk = function(){ //新方法
	console.log('i can walk!');
};
w1.walk();

Mother.prototype.isSexy = true; //添加超类属性，不建议
w1.isSexy;

s1.isHandsome = false;  //添加属性

Girl.prototype.showAge = function(){    //覆写超类方法
	console.log('ya,'+this.age);
};
s1.showAge();
g2.age = 23;
g2.showAge();
w1.showAge();

s1.__proto__ === Son.prototype; //true
s1.__proto__.__proto__ === Mother.prototype; //true
w1.__proto__ === WaiSun.prototype;    //true
w1.__proto__ === g2;    //true
g2.walk();  // 注意： i can walk!

g2.__proto__.name;
g2.__proto__.constructer.name;

w1.__proto__.age;   //23, 不是18
w1.__proto__.__proto__.age; //60

// 必须先有类，再有对象吗？！不用，对象克隆

```


