《javascript启示录》总结
<!-- 
    Author:Triangel
    Email:linjinxia_ya@163.com
    Time:2016/08/01
 -->
第6章  this关键字
1.this是在函数内部使用，用来引用包含函数的对象，而不是函数本身[使用new关键字或call()和apply()的情况例外]。
2.this是基于调用函数的上下文，会根据调用函数所在的上下文而改变。
例子：
    var foo = 'foo';
    var obj = {foo:'I am Triangel'};
    var sayFoo = function(){console.log(this['foo']);} 
    obj.sayFoo = sayFoo(); //此时this指obj,即调用它的函数的上下文，即引用包含该函数的对象obj
    obj.sayFoo(); //I am Triangel
    sayFoo(); //foo
3.当this值的宿主函数被封装在另一个函数内部或者在另一个函数的上下文中被调用时，this值将永远是对head对象的引用。
例子1：当this值的宿主函数被封装在另一个函数内部
    var obj = {
        func1:function(){
            console.log(this); // 输出obj
            var func2 = function(){
                console.log(this); //输出window,从此处开始this指window对象
                var func3 = function(){
                    console.log(this); //输出window
                }
            }
        }
    }
    obj.func1();
例子2：在另一个函数的上下文中被调用时
    var foo = {
        func1:function(bar){
            bar(); //window
            console.log(this); //foo
        }
    }
    function bar(){
        console.log(this);
    }
    foo.func1(bar); //输出window{},obj{}

4.利用call()和apply()，此时this指向为所引用的对象。
例子：
（1）call()-->参数用逗号分隔
    var a = function(p1,p2){
     this.foo = p1;
     this.bar = p2;
     console.log(this); //Object{foo='foo',bar='bar'}--此时指向为obj
    }
    var obj = {};
    a.call(obj,'foo','bar');
    console.log(obj); //Object{foo='foo',bar='bar'}--与上面this的值一样，因为上面this就是指向obj这个对象
（2）apply()-->参数是数组
    var a = function(p1,p2){
     this.foo = p1;
     this.bar = p2;
     console.log(this); //Object{foo='foo',bar='bar'}--此时指向为obj
    }
    var obj = {};
    a.apply(obj,['foo','bar']);
    console.log(obj); //Object{foo='foo',bar='bar'}--与上面this的值一样，因为上面this就是指向obj这个对象
5.构造函数内部使用this关键字，此时this指向被创建的对象。
例子：
（1）如果使用new关键字调用构造函数时，this引用‘即将创建的对象’。
    var Person = function(name){
        this.name = name || 'Triangel'; //this引用所创建的实例，即aa
    }
    var aa = new Person('Jsonya');
    console.log(aa.name);
（2）如果不使用new关键字，this值就是调用Person的上下文
    var Person = function(name){
        this.name = name || 'Triangel'; //this引用所创建的实例，即aa
    }
    var aa = Person('Jsonya');
    console.log(aa.name); //undefined
    console.log(window.name); //Jsonya
6.原型方法中的this也是引用其实例对象，如果实例对象不包含所要查找的属性，则继续在原型上查找。
例子：
    var Person = function(x){
    if(x){this.fullName = x}
}
Person.prototype.whatIsMyFullName = function(){
    return this.fullName;
}
var Jsonya = new Person('Jsonya');
var Triangel = new Person('Triangel');
console.log(Jsonya.whatIsMyFullName()); //Jsonya
console.log(Triangel.whatIsMyFullName()); //Triangel
Person.prototype.fullName = 'a';
var a = new Person();
console.log(a.whatIsMyFullName()); //a

第7章  作用域和闭包
1.概念：作用域是执行代码的上下文。作用域有三种类型：全局作用域，局部作用域和eval作用域。
2.注意：函数定义时确定作用域，而非调用时确定

第8章  函数原型属性
1.概念：prototype属性是JavaScript为每个Function()实例创建的一个对象。它将通过new关键字创建的对象实例链接会创建它们的构造函数.
例子：
    Array.prototype.foo = "foo";
    var test = new Array();
    console.log(test._proto_.foo); //输出foo，因为test._proto_ = Array.prototype
    console.log(test.constructor.prototype.foo); //输出foo
    cosole.log(test.foo); //输出foo
解释：test._proto_和test.constructor.prototype引用Array.prototype
2.原型链的最后是Object.prototype.
注意：任何添加到Object.prototype的内容都会出现在for in 循环中。
3.创建继承链
例子：Chef对象继承Person对象
    Chef.prototype = new Person();
4.运用原型：
例子：
(1)直接调用
Array.prototype.join = function(){
    var that = this; //获取调用该方法的对象
    var str = "";
    for(var i=0,len=that.length; i&lt;len; i++){
        str += that[i] + " ";
    }
    return str;
}
var a = [1,2,3];
a.join();
(2)传参形式
Array.prototype.join = function(a){
 var str = "";
 for(i in a){
  str += a[i]+"/";
 }
 return str;
}
var a = [1,2,3];
a.join(a);
5.性能：
（1）搜索实例成员比字面量或者局部变量中读取数据代价更高，加上遍历原型链带来的开销，会让性能问题更严重。
比如：
location.href比window.location.href要快，后者也比window.location.href.toString()要快，如果这些属性不是对象的实例属性，那么成员解析还需要搜索原型链，这会花更多的时间。
（2）提示：
大多数浏览器通过点表示法和括号表示法操作没有明显区别，但是在Safari中，点符号始终会更快。
（3）性能优化小贴士：
【1】缓存对象成员；同一个函数中没有必要多次读取同一个对象成员。
比如：
优化前代码：
function hasEitherClass(e,className1,className2){
    return e.className == className1 || e.className == className2;
}
优化后代码：
function hasEitherClass(e,className1,className2){
    var currentClassName = e.className;
    return currentClassName == className1 || currentClassName == className2;
}
好处：对对象成员的查找次数减少到1次，将第一次查找后的值存在局部变量中，局部变量读取速度快很多。
【2】javascript命名空间，减少频繁访问嵌套属性。
优化前代码：（成员查找次数共9次）
function toggle(e){
    if(YAHOO.util.Dom.hasClass(e,'selected'){ //3次
        YAHOO.util.Dom.removeClass(e,'selected'); //3次
        return false;
    }
    else{
        YAHOO.util.Dom.addClass(e,'selected'); //3次
        return true;
    }
}
优化后代码：（成员查找次数共5次）
function toggle(e){
    var Dom = YAHOO.util.Dom; //2次
    if(Dom.hasClass(e,'selected'){ //1次
        Dom.removeClass(e,'selected'); //1次
        return false;
    }
    else{
        Dom.addClass(e,'selected'); //1次
        return true;
    }
}

第9章  Array()
实例属性：
    constructor/index/input/length
实例方法：
    pop()：删除数组末尾元素
    push()：添加元素到数组末尾
    reverse()：反转
    shift()：删除数组最前面的元素
    sort()：排序
    splice()：通过索引删除某个元素
    unshift()：添加到数组的前面
    concat()：连接
    join()：用符号连接
    slice()：复制一个数组
    splice()：通过索引删除某个元素

第10章  String()
var string1 = new String('foo');//object,foo{0='f',1='o',2='o'}
var string2 = String('foo'); //string,foo
var string3 = 'foo'; //foo
实例属性：
    constructor
    length
实例方法：
    charAt()
    charCodeAt()
    concat()
    indexOf()
    lastIndexOf()
    localeCompare()
    match()
    quote()
    replace()
    search()
    slice()
    splite()
    substr()
    substring()
    toLocaleLowerCae()
    toLocaleUpperCase()
    toLowerCase()
    toString()
    toUpperCase()
    valueOf()

第11章  Number()
var number1 = new Number(1); //object,1
var number2 = Number(1); //number,1
var number3 = 1; //number,1
实例属性：
    constructor
实例方法：
    toExponential()
    toFixed()
    toLocaleString()
    toPrecision()
    toString()
    valueOf()

第12章
如果一个值是0、-0、null、false、NaN、undefined或空字符串（""）,那么它是false。

第14章  null
1.概念：显示指出对象属性不包含值
2.null与undefined的区别：undefined告知你有东西丢失，null只是指出该变量没值。
3.利用typeof验证值为null的变量的类型都是返回object，所以测试返回值是不是null的时候一般都直接将变量与null比较，就比如：
var a=null;
console.log(a===null);//输出'null'，此时a为真正的null
var obj = null;
console.log(typeof obj);//输出'object'，判断不出是否null
注意：如果用‘==’，无法判断出是null还是undefined;

第15章  undefined
1.两种用法：
（1）声明变量没有指定值。
（2）视图访问的对象属性没有被定义（即没有被命名），并且不存在原型链中
2.undefined是全局作用域中的一个属性。