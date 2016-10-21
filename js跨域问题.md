<!-- 
    Author:Triangel
    Email:linjinxia_ya@163.com
    Time:2016/09/01
 -->
一、js跨域问题
（1）概念：js在不同域之间进行数据传输或者通信，只要协议、域名、端口有任何一个不同，就被当做是不同的域。出现的主要原因是因为安全限制（同源策略，即js或者cookie只能访问同域下的内容）
注意：
第一，如果是协议和端口造成的跨域问题“前台”是无能为力的，
第二：在跨域问题上，域仅仅是通过“URL的首部”来识别而不会去尝试判断相同的ip地址对应着两个域或两个域是否在同一个ip上。
“URL的首部”指window.location.protocol + window.location.host，也可以理解为“Domains, protocols and ports must match”。
（2）解决方法：
[1]通过jsonp跨域
参考资料：http://www.cnblogs.com/chopper/archive/2012/03/24/2403945.html
原理：在js中，我们直接用XMLHttpRequest请求不同域上的数据时，是不可以的。但是，**在页面上引入不同域上的js脚本文件却是可以的，jsonp就是利用了这个特征来实现的。**
具体实现方法：
-jquery方法：利用jquery封装好的方法$.getJSON(url,[data],[callback])
如：
```javascript
<script type="text/javascript">
    $.getJSON('http://localhost:8080/test.ashx?callback=?',function(data){
            alert(data.name + " is a good student.");
        });
</script>
```
注意：callback参数后面的问号是内部自动生成的一个回调函数名，用来指代后面的匿名函数。
如果我们想要自己指定回调函数，可以使用$.ajax()方法
如：
```javascript
<script type="text/javascript">
    $.ajax({
        url:'http://localhost:20002/MyService.ashx?callback=?',
        dataType:'jsonp', //关键之处
        //jsonp:'callback', //传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(一般默认为:callback)
        jsonpCallback:'person', //自定义的jsonp回调函数名称，默认为jQuery自动生成的随机函数名，也可以写"?"，jQuery会自动为你处理数据
        //jsonpCallback就是可以指定我们自己的回调方法名person，远程服务接受callback参数的值就不再是自动生成的回调名，而是person。dataType是指定按照JSOPN方式访问远程服务
        success:function(data){
            alert(data.name + ' is a good student.');
        }
    });
</script>
```
-js方法：
```javascript
<script type="text/javascript">
    function addScriptTag(src){
        var script = document.createElement('script');
        script.setAttribute("type","text/javascript");
        script.src = src;
        document.body.appendChild(script);
    }
    window.onload = function(){
        addScriptTag("http://localhost:8080/test.ashx?callback=person");
    }
    function person(data){
        alert(data.name + ' is a good student.');
    }
</script>
```
注意：json和jsonp的区别
参考资料：http://kb.cnblogs.com/page/139725/
JSON是一种数据交换格式；JSONP是一种非官方跨域数据交互协议。
    JSONP出现的原因：跨域或无线访问问题；而Web页面上调用js文件则是不受跨域的影响(凡是有src这个属性的标签都拥有跨域的能力，比如```<script>、<img>、<iframe>```);为了解决跨域等问题，就可以利用jsonp，该协议的一个要点就是允许用户传递一个callback参数给服务器端，然后服务端返回数据时将这个callback参数作为函数名包裹住JSON数据，

[2]利用cors(跨域资源共享，Cross-Origin Resource Sharing)
含义：它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。
参考资料：http://www.ruanyifeng.com/blog/2016/04/cors.html
          http://www.cnblogs.com/Darren_code/p/cors.html
原理：利用http中的Access-Control-Allow-Origin.
运用：在服务器端使用header('Access-Control-Allow-Origin:*');//允许任何域向我们的服务端提交请求
header('Access-Control-Allow-Origin:http://www.test1.com');//允许来自http://www.test1.com这个域名的请求
**CORS与JSONP的比较**
JSONP只支持GET请求，CORS支持所有类型的HTTP请求，JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。
[3]document.domain+iframe的设置
参考资料：http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html
          http://www.cnblogs.com/rainman/archive/2011/02/21/1960044.html
使用范围：主域相同而子域不同的例子，比如：http://www.a.com/a.html和http://script.a.com/b.html，它们的主域名是一样的，都是a.com。如果你想要把script.a.com设置为baidu.com是不行的。
扩展：顶级域名：.com、.net等
      一级域名：baidu.com等
      二级域名：www.baidu.com等
上面中的两个网址www.a.com和script.a.com都是二级域名
例子：
```script
在a.html页面中的js代码如下：
<script type="text/javascript">
    document.domain = 'a.com';
    var ifr = document.createElement('iframe');
    ifr.src = 'http://script.a.com/b.html';
    ifr.style.display = 'none';
    document.body.appendChild(ifr);
    ifr.onload = function(){
        var doc = ifr.contentDocument || ifr.contentWindow.document;
        // 在这里操纵b.html
        alert(doc.getElementsByTagName("h1")[0].childNodes[0].nodeValue);
    };
</script>
在b.html页面中的代码如下:
<script type="text/javascript">
    document.domain = 'a.com';
</script>
```
以上就是先利用document.domain设置两个主域相同的页面处于同一主域中，然后再用iframe去调用另一个页面。
问题：安全性，当一个站点（b.a.com）被攻击后，另一个站点（c.a.com）会引起安全漏洞。
如果一个页面中引入多个iframe，要想能够操作所有iframe，必须都得设置相同domain。
[4]window.name实现跨域数据传输
参考资料：http://www.cnblogs.com/rainman/archive/2011/02/21/1960044.html
关键点：iframe的src属性由外域转向本地域，跨域数据即由iframe的window.name从外域传递到本地域。这个就巧妙地绕过了浏览器的跨域访问限制，但同时它又是安全操作。
[5]HTML5的postMessage
参考资料：http://www.cnblogs.com/dolphinX/p/3464056.html




