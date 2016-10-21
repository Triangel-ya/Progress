<!-- 
    Author:Triangel
    Email:linjinxia_ya@163.com
    Time:2016/10/08
 -->
# JavaScript高级程序设计
### 第五章 引用类型
#### Object
##### Object的命名：
* var obj = new Object();  //构造函数法
* var obj = {}; //对象字面量表示法
##### Object共有的方法
* toString()
* toLocalString()
* valueOf()
#### Array
##### Array的命名
* var arr = new Array();
* var arr = [];
##### Array的检测
* 在ES中，可以使用 arr instaceof Array来判断，但是这是假定只有一个全局执行环境的情况下的，如果网页包含多个框架，就含有不痛同的全局执行环境，假设上述的arr是在其他iframe引入的，那么ar instaceof Array = false。所以就出现了ES5中的另一个方法。
* 在ES5中，通过 Array.isArray(arr)来判断；（兼容性：IE9+、Firefox4+、Safari5+、Opera10.5+、Chrome.）
tip:兼容低版本的检测方法：
```javascript
function isArray(arr){
    return Object.prototype.toString.call(arr) == "[object Array]";
}
```
同理：判断原生函数：
```javascript
function isFunction(fun){
    return Object.prototype.toString.call(fun) == "[object Function]";
}
```
判断正则表达式：
```javascript
function isRegExp(val){
    return Object.prototype.toString.call(val) == "[object RegExp]";
}
```
##### 栈方法
* arr.push() //尾进
* arr.pop() //尾出
* arr.shift() //头出
* arr.unshift() //头进
##### 数组排序
* arr.reverse() //反转（会改变原数组）
* arr.sort() //升序排序（会改变原数组）（备注：sort方法会调用toString方法，比较得到的每个字符串，所以即使是数组也会返回字符串然后再进行比较）
如果想要自定义比较方法，可以这么写：
```javascript
<script type="text/javascript">
    function compare(a,b){
        return b-a;
    }
    var arr = [2,1,3];
    arr.sort(compare);
    console.log(arr);
</script>
```
##### 数组操作
* arr1.concat(arr2); //拼接(不会改变原数组)
* arr.slice(开始位置,结束位置(可选)); //截断(不会改变原数组)
  (如果第二项忽略，就会返回从开始确定的位置到尾的数组；如果为负，就负+数组长度之后的数)
* arr.splice() //删除、插入、替换(会改变原数组)
  例如：var arr = [1,2,3,4,5];
  删除：arr.splice(要删除的第一个项的位置,要删除的项数)
  arr.splice(0,2); //删除从0到2的三个数组项，返回[删除了的三个项组成的数组[1,2]，arr变成[3,4,5]；
  插入/替换：arr.slice(要插入的位置,项数(若插入就是0项,若替换就是具体的数字),...(这是要插入/替换的值));
  arr.splice(5,0,6,7,8,9); //arr=[1,2,3,4,5,6,7,8,9]
##### 位置查找
* indexOf(要查找的项,起点索引(可选)) //从头开始查找
* lastIndexOf() //从尾开始查找
##### 迭代方法
* arr.every(function(item,index,array){}); //对所有项都返回true,才会返回true
* arr.some(...) //对任一项返回true,就返回true
* arr.filter(...) //返回true想组成的数组
* arr.forEach(...) //运行该函数,无返回值
* arr.map(...) //调用结果后返回的数组
(兼容性：IE9+、Firefox2+、Safari3+、Opera9.5+、Chrome.)
##### 归并方法
*这两种方法都是将上一次的结果返回作为下一次的参数*
* reduce(前一个值,当前值,项的索引,数组对象); //从头到尾
* reduceRight(前一个值,当前值,项的索引,数组对象); //从尾到头
#### Date
##### Object的命名：