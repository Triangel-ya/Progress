《高性能JavaScript》总结 
<!-- 
    Author:Triangel
    Email:linjinxia_ya@163.com
    Time:2016/08/01
 -->
第一章	加载和执行
1. <script></script>标签每次出现都霸道地让页面等待脚本的解析和执行，无论当前的JavaScript代码是内嵌还是包含在外链文件中，页面的下载和渲染都必须停下来等待脚本**执行**完成。这是页面生存周期中必要环节，因为脚本的执行过程中可能会修改页面内容。
2. 浏览器在解析到<body>标签之前，不会渲染页面的任何部分。把脚本放在页面顶部将会导致明显的延迟，通常表现为显示空白页面，用户无法浏览内容，也无法与页面进行交互。
3. IE8，Firefox3.5，Safari4和Chrome2都允许并行加载JavaScript文件，只是他们在加载的时候还是会阻塞其他资源的下载。
    4. 结论：所以推荐奖所有的<script></script>标签尽可能放在<body>标签的底部，以尽量减少对整个页面下载的影响。
    5. HTML4中为<script></script>标签定义了一个扩展属性：defer，指明本元素所包含的脚本不会修改DOM，因此代码能安全地延迟执行，但是还是与没有加defer一样正常下载，只是还不执行而已。出来Opera Mini不支持外，其他主流浏览器都支持。
6. 几种方法能减少JavaScript对性能的影响：
    （1）<body>闭合标签之前，将所有的<script></script>标签放在页面底部，这能确保在脚本执行前页面已经完成了渲染。
    （2）合并脚本。页面中的<script></script>标签也少，加载也就越快，响应也更迅速，因为减少了http请求。无论是外链文件还是内嵌脚本都是如此。
（3）有多种无阻塞下载JavaScript的方法：
    -- 使用<script></script>标签的defer属性
    -- 使用动态创建<script></script>元素来下载并执行代码（如LazyLoad类库）
推荐的无阻塞模式的代码：（可以将loadScript()函数直接嵌入页面，从而避免多产生一次http请求）
<script type="text/javascript">
    function loadScript(url,callback){
        var script = document.createElement('script');
        script.type = "text/javascript";
        if(script.readyState){//IE
            script.onreadystatechange = function(){
                if(script.readyState == "loaded" || script.readyState == "complete"){
                    script.onreadystatechange = null;
                    callback();
                }
            };
        }
        else{//其他浏览器
            script.onload = function(){
                callback();
            };
        }
        script.src = url;
        document.getElementsByTagName("head")[0].appendChild(script);
    }

    loadScript("aa.js",function(){//调用函数加载解析js文件，最好将loadScript()函数直接嵌入页面，从而避免多产生一次http请求
        Application.init();
    });
</script>
-- 使用xhr对象下载JavaScript代码并注入页面中。
//XMLHttpRequest脚本注入
var xhr = new XMLHttpRequest();
xhr.open("get","aa.js",true);
xhr.onreadystatechange = function(){
    if(xhr.readyState == 4){
        if(xhr.status >= 200 && xhr.status < 300 || xhr.status == 304){
            var script = "text/javascript";
            script.text = xhr.responseText;
            document.body.appendChild(script);
        }
    }
}
xhr.send(null);

第二章	数据存储
1.	通常的建议：如果在乎运行速度，那么尽量使用字面量或者局部变量，减少数组项和对象成员的使用。
例如：平时使用到的document是一个全局对象，所以如果在函数中要经常使用到它，可以定义一个局部变量来存document，这样每次使用的时候就不用搜索整个作用域，知道最后在全局变量对象中找到它，用局部变量代替全局变量，减少性能的影响。
2.	作用域的原理：
作用域有三种类型：全局作用域（只有一个），局部作用域（也称“函数作用域”，有多个）和eval作用域（有多个）。
3.	全局变量总是存在执行环境（也称“执行上下文”）作用域链的做末端，所以读写一个全局变量的性能开销是最大的。所以如果一个全局变量在函数内部被访问多次，可以先在函数内部定义一个局部变量来存储全局变量，之后就不用一直通过作用域链一步步寻找全局变量，这样就减少了性能开销。
4.	动态作用域（就是改变之前的执行环境的作用域链）：主要有with语句，try-catch语句的catch子句，或者是包括eval()的函数。（只有在确定有必要时才推荐使用动态作用域）
5.  闭包
（1）概念：闭包是允许函数访问局部作用域以外的数据，即使外部函数已经退出，外部函数的变量仍可以被内部函数访问到。
（2）闭包的实现需要三个条件：
a.内部函数实用了外部函数的变量
b.外部函数已经退出，但是其活动对象由于闭包的存在而没有销毁
c.内部函数可以访问
（3）闭包最需要关注的性能点：在频繁访问跨作用域的标识符时，每次访问都会带来性能问题。闭包存在性能问题，就是当包含闭包的活动对象在闭包销毁之前无法销毁。
（4）解决闭包性能问题的方法：将常用的跨作用域的变量存储在局部变量中，然后直接访问局部变量。
(以下[1][2]来源：http://hao.jser.com/archive/8655/)
[1]如上面的描述，当执行闭包函数后，父函数所保留下来的活动对象并不是在闭包函数的作用域链的首位（首位存放的是闭包的活动对象），当频繁的访问跨作用域的标识符时候，每次都会造成性能的损失，我们仍然可以将常用的跨作用域变量存储在局部变量中，直接访问该局部变量。
由于闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部删除。
如下代码：
function add(a,b){
    var sum = a + b;
    return sum;
}
var t = add(1,2);
![](http://7xaw6e.com1.z0.glb.clouddn.com/wp-content/uploads/2015/11/31/20151031_563514716fc81.png)
![](http://7xaw6e.com1.z0.glb.clouddn.com/wp-content/uploads/2015/11/31/20151031_56351471a322c.png)
[2]实用闭包所造成的内存泄露问题（IE9以下）：
IE9及以下的版本使用的是引用计数的内存回收机制，当引用计数为0的时候将会回收，但有一种循环引用的情况
window.onload = function(){
    var el = document.getElementById("id");
    el.onclick = function(){
        alert(el.id);
    }
}
这段代码执行时，将匿名函数对象赋值给el的onclick属性；然后匿名函数内部又引用了el对象，存在循环引用，所以不能被回收；
（javascript 高级程序设计（第三版））
解决方法：
window.onload = function(){
    var el = document.getElementById("id");
    var id = el.id; //解除了循环引用
    el.onclick = function(){
        alert(id); //并没有出现循环引用
    }
    el = null; // 将闭包引用的外部活动对象清除
}

相关练习：[慕课网](http://www.imooc.com/article/10979)
[1]函数里面的函数的作用域是window
var name = "The Window"; 　　
var object = { 　　　　
    name : "My Object", 　　　　
    getNameFunc : function(){ 　　　　　　
        return function(){ 　　　//函数里面的函数的作用域是window　　　　　
            return this.name; 　　　　　
        }; 　　　　
    } 
}; 
alert(object.getNameFunc()()); //The Window

[2]h的作用域链为：h的活动对象->f的活动对象->window对象。
function f(x) { 
    var g = function () { 
        return x; 
    } 
    return g;
} 
var x = 2;
var h = f(1); 
alert(h()); //1


第三章	


