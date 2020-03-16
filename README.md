# 面试考点

# 写 React / Vue 项目时为什么要在列表组件中写 key，其作用是什么
  key是给每一个vnode(虚拟Dom)的唯一id,可以依靠key,更准确, 更快的拿到oldVnode中对应的vnode节点。利用key的唯一性生成map对象来获取对应节点，比遍历方式更快
# 防抖：动作绑定事件，动作发生后一定时间后触发事件，在这段时间内，如果该动作又发生，则重新等待一定时间再触发事件。
      function debounce(fn) {
      let timeout = null; // 创建一个标记用来存放定时器的返回值
      return function () {
        clearTimeout(timeout); // 每当用户输入的时候把前一个 setTimeout clear 掉
        timeout = setTimeout(() => { // 然后又创建一个新的 setTimeout, 这样就能保证输入字符后的 interval 间隔内如果还有字符输入的话，就不会执行 fn 函数
          fn.apply(this, arguments);
        }, 500);
      };
    }
    function sayHi() {
      console.log('防抖成功');
    }

    var inp = document.getElementById('inp');
    inp.addEventListener('input', debounce(sayHi)); // 防抖
 # 节流： 动作绑定事件，动作发生后一段时间后触发事件，在这段时间内，如果动作又发生，则无视该动作，直到事件执行完后，才能重新触发
    function throttle(fn) {
      let canRun = true; // 通过闭包保存一个标记
      return function () {
        if (!canRun) return; // 在函数开头判断标记是否为true，不为true则return
        canRun = false; // 立即设置为false
        setTimeout(() => { // 将外部传入的函数的执行放在setTimeout中
          fn.apply(this, arguments);
          // 最后在setTimeout执行完毕后再把标记设置为true(关键)表示可以执行下一次循环了。当定时器没有执行的时候标记永远是false，在开头被return掉
          canRun = true;
        }, 500);
      };
    }
    function sayHi(e) {
      console.log(e.target.innerWidth, e.target.innerHeight);
    }
    window.addEventListener('resize', throttle(sayHi));
    
 #  手写简单promise

    const PENDING = 'pending';
    const RESOLVED = 'resolved';
    const REJECTED = 'rejected';

    function MyPromise(fn) {
        const _this = this;
        _this.state = PENDING;
        _this.value = null;
        _this.resolvedCallbacks = [];
        _this.rejectedCallbacks = [];


    // resolve函数
    function resolve(value) {
        if (_this.state === PENDING) {
            _this.state = RESOLVED;
            _this.value = value;
            _this.resolvedCallbacks.map(cb => cb(_this.value));
        }
    }

    // rejected函数
    function reject(value) {
        if (_this.state === PENDING) {
            _this.state = REJECTED;
            _this.value = value;
            _this.rejectedCallbacks.map(cb => cb(_this.value));
        }
    }

    // 当创建对象的时候，执行传进来的执行器函数
    // 并且传递resolve和reject函数
    try {
        fn(resolve, reject);
    } catch (e) {
        reject(e);
    }
}

    // 为Promise原型链上添加then函数
    MyPromise.prototype.then = function (onFulfilled, onRejected) {
        const _this = this;
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v;
        onRejected = typeof onRejected === 'function' ? onRejected : r => {
            throw r;
        }
        if (_this.state === PENDING) {
            _this.resolvedCallbacks.push(onFulfilled);
            _this.rejectedCallbacks.push(onRejected);
        }
        if (_this.state === RESOLVED) {
            onFulfilled(_this.value);
        }
        if (_this.state === REJECTED) {
            onRejected(_this.value);
        }
        return _this;
    }



    // 测试
    new MyPromise(function (resolve, reject) {
        setTimeout(() => {
            resolve('hello');
        }, 2000);
    }).then(v => {
        console.log(v);
    }).then(v => {
        console.log(v + "1");
    })

# 节流2
   
    function conduct(fn, delay, masExac) {

        var timer;
        
        var lastTime = new Date();
        
        return function (arg) {
        
            var now = new Date();
            
            clearTimeout(timer)
            
            if (now - lastTime < masExac) {
            
                timer = setTimeout(() => {//只在最后事件停止出发的时候执行过
                
                    fn(arg);
                    lastTime = now;
                    
                }, delay)
                
            } else {          //间隔masExac时间执行一次
            
                fn(arg);
                lastTime = now;
                
            }
        }
    }

    function Fn(data){
        console.log(data)
    }
    var throttle = conduct(Fn,1000,2000)
    $(document).scroll(function () {
        throttle('1')
    })
  # 防抖2
  
    var isThrottle = true; //初始化无需走节流
   
      var setTimer = null; //初始化定时器

      var throttle = function(fn, time) {
          clearInterval(setTimer)
          var time = time || 300;
          if (isThrottle) { //注意：在这里第一次滚动时候我们是需要立马执行函数的，真正的防抖是从第二次开始
              fn();
              isThrottle = false;
              return false;
              /*细节：这里如果你不return false阻断代码往下执行，那么在第一次时候实际还会走下面的else，也就是触发了两次,（因为滚轮触发的事件频率很高，第一次节流（else中）的事件会紧接着初始化第一次未节流事件去执行了）*/
          } else {
              console.log('高频触发滚动事件中')
              setTimer = setTimeout(function() {
                  fn();
                  isThrottle = true;
                  console.log('上一次打印是节流执行。。。')
              }, time)
          }
      }

      function conduct() {
          console.log("1")
      }
      $(document).scroll(function() {
          throttle(conduct, 1000);
      })
# ES6的set用于数组去重
    // 去重数组的重复对象
    let arr = [1, 2, 3, 2, 1, 1]
    [... new Set(arr)]	// [1, 2, 3]
# 数组扁平化
    是指将一个多维数组变为一维数组
    [1, [2, 3, [4, 5]]]  ------>    [1, 2, 3, 4, 5]
# js异步函数的发展历程
  1. callback(回调函数）---回调地狱
  2.promise---------------then方法的链式调用
  3.async/await 
# TCP3次或4次握手
     4次握手出现于服务器需要多给客户端传输一次

     3次的情况
     顾客：老板，还不上菜
     老板：客官，菜来了

     4次的情况
      顾客：老板，还不上菜
     老板：稍等一下，菜马上出锅
     老板：客官，菜来了
# setState的同步、异步情况
    生命周期中的 setState 处于一个大的 transaction 中，此时的 isBatchingUpdate 为 true，执行 setState 只
    会让 dirtyComponents 数组 push 当前组件而不会进一步处理，此时 log 来看的话 state 还是没有变的。而如果
    在 transaction 之外，例如 setTimeout 里 setState，此时 isBatchingUpdate 为 false，会一路直接执行下来
    更改 state，所以此时 log 出来 state 是被立刻改变了的。因此 setState 不保证是同步，而不是说它一定是异步
# 重绘
    由于节点的几何属性发生改变或者由于样式发生改变而不会影响布局的，称为重绘，例如outline, visibility, color、background-color等，重绘的代价是高昂的，因为浏览器必须验证DOM树上其他节点元素的可见性
# 重排（回流/feflow)
    元素几何属性改变后影响布局的，会触发重排。  重排必然带来重绘，但是重绘未必带来重排
# 模块化主要是用来抽离公共代码，隔离作用域，避免变量冲突等
# 去掉antd的input数字输入框含有上下箭头的默认样式
    //火狐浏览器
    input[type="number"] {
      -moz-appearance: textfield;
    }
    //谷歌浏览器
    input[type="number"]::-webkit-inner-spin-button,
    input[type="number"]::-webkit-outer-spin-button {
      -webkit-appearance: none;
      margin: 0;
    }
# ["A1", "A2", "B1", "B2", "C1", "C2", "D1", "D2", "A3", "B3", "C3", "D3"].sort()输出
    //["A1", "A2", "A3", "B1", "B2", "B3", "C1", "C2", "C3", "D1", "D2", "D3"]
# let的某个特性
    for(var i=0;i<10;i++){
      setTimeout(function(){
        console.log(i)                     // 9 9 9 9 9 9 9 9 9 9
      },1000)
    }
    
    for(let i=0;i<10;i++){
      setTimeout(function(){
        console.log(i)                     // 0 1 2 3 4 5 6 7 8 9
      },1000)
    }
    
    //setTimeout 函数的第三个参数，会作为回调函数的第一个参数传入
    for(var i=0;i<10;i++){
      setTimeout(function(i){
         consolelog(i)                     // 0 1 2 3 4 5 6 7 8 9
      },1000,i)
    }
  # 引用类型在比较运算符时候,隐式转换会调用本类型toString或valueOf方法
        var a = {num:0};
      a.valueOf = function(){
        return ++a.num
      }
      if(a == 1 && a == 2 && a == 3){
        console.log(1);
      }
# [3, 15, 8, 29, 102, 22].sort() 
    //[102, 15, 22, 29, 3, 8]
    根据MDN上对Array.sort()的解释，默认的排序方法会将数组元素转换为字符串，然后比较字符串中
    字符的UTF-16编码顺序来进行排序。所以'102' 会排在 '15' 前面
# 为什么前端监控通常使用1x1的GIF图片进行上报
    1.没有跨域问题  //打点请求域名一般是不同于当前项目域名，解决为什么不用get/post等方法直接请求
    2.不会阻塞页面加载，影响用户体验   // 解决 js/css等都需要插入页面DOM后，才会触发请求
    3.在png/jpg/gif等图片格式中，GIF所占体积最小
# js执行顺序
    var a = {n: 1};
    var b = a;
    a.x = a = {n: 2};
    console.log(a.x) //undefined	
    console.log(b.x)  //{n:2}
    
    why？ 重点在于a.x=a={n:2}  此语句执行顺序为从左向右，等号左边是给a指针指向的对象{n:1}添加一个key属性x,即a指向的对象变成
    了{n:1,x:undefined},此时看等号右边，是一个表达式，等待表达式执行完，a被重新赋值，也就是a指针指向变了，此时a=={n:2}。
    那{n:1,x:undefined}怎么办，被垃圾回收？NO,b指针还在用着它，so，明朗了吧
#  冒泡排序使数组按照顺序排列
    function bubbleSort(arr){
      for(var i=0;i<arr.length;i++){//第一层循环可以取到i值，由i值确定二级循环的length;数组排序完成时不一定是全部循环完成，中途可能就完成
        for(var j=0;j<arr.length-i-1;j++){  //每一次循环结束的最后一项都是最大值，所以下一次循环就不需要再比较最后一项
          if(arr[j]>arr[j+1]){
             let temp = arr[j+1];
             arr[j+1] = arr[j];
             arr[j]=temp;
          }
        }
      }
      console.log(arr);
    }
# ‘1'.toString()为社么可以调用？
     其实这个语句运行过程中做了这样几件事
     var s =new Object('1')  //为什么不是new String,因为Sybmbos类型的出现，对它调用new会报错，目前ES6规范不建议用new创建基本类型的包装类
     s.toString()
     s=null;     //执行完销毁这个实例
# [] == ![]结果是什么？为什么？
     true
     “==” 中，左右两边都需要转换为数字然后进行比较, "[]"转换数字为 0 ，"![]"首先转换为布尔值，由于[]作为一个引用类型转换为布尔值为true,因此![]为false,进而转换位数字0，  “0==0” 结果为true
# 如何让if(a==1&&a==2)条件成立？
    var a ={
      value:0,
      valueOf:function(){
        this.value++;
        return this.value
      }
    }
    判断a==1的时候，执行了a.valueOf()==1,以此类推
# 如果父元素的font-size也是采用em表示,那么子元素的font-size怎么计算；
	如果父元素设置为2rem;真是font-size为32px;
	子元素也设置为2rem;那么子元素的font-size为64px;
# 下面通过代码阐述instanceof的内部机制,假设现在有 x instanceof y 一条语句
	while(x.__proto__!==null) {
	    if(x.__proto__===y.prototype) {
		return true;
	    }
	    x.__proto__ = x.__proto__.proto__;
	}
	if(x.__proto__==null) {return false;}
	
	x会一直沿着隐式原型链__proto__向上查找直到x.__proto__.__proto__......===y.prototype为止，如果找到则返回true，也就是x为y的一个实例。否则返回false，x不是y的实例。
#  react-router 和  react-router-dom 
	不同之处就是后者比前者多出了 <Link> <BrowserRouter> 这样的 DOM 类组件。
	因此我们只需引用 react-router-dom 这个包就行了
#  刷新页面，js 请求一般会有哪些地方有缓存处理？
	DNS缓存：短时间内多次访问某个网站，在限定时间内，不用多次访问DNS服务器。
	CDN缓存：内容分发网络（人们可以在就近的代售点取火车票了，不用非得到火车站去排队）
	浏览器缓存：浏览器在用户磁盘上，对最新请求过的文档进行了存储。
	服务器缓存：将需要频繁访问的Web页面和对象保存在离用户更近的系统中，当再次访问这些对象的时候加快了速度。
# 可以通过哪些方法优化css3 animation渲染
	尽可能多的利用硬件能力，如使用3D变形来开启GPU加速：
	尽可能少的使用box-shadows与gradients
	在CSS动画中使用webkit-transform: translateX(3em)的方案代替使用left: 3em，因为left会额外触发layout与pain
#  CSS Hack (用一些奇怪的手段达到兼容的方式！）
	<!–-[if IE 7]>
		<link rel="stylesheet" href="ie7.css" type="text/css" />
	<![endif]–->
# （值1，值2）等于啥
	var d = a = 3, b = 4
   	console.log(d)  //3
	
	var d = (a = 3, b = 4)
    	console.log(d)  //4
#  !!隐式转换
	var x = !!"Hello" + (!"world", !!"from here!!");    console.log(x)   //2
   	var x2 = 1 + (0,1);                                 console.log(x2)  //2
   	var x3= true + (false,true);                        console.log(x3)  //2
# a+++b运算过程
    var a=1;
    console.log( a+++3 ); //4      分为2部分运算，第一个首先a要先加上3，即1+3得出4，然后a++;即a此时==2；
    console.log( a+++1 );//3       分为2部分运算，第一个首先a要先加上1，即2+1得出3，然后a++;即a此时==3；
    console.log( a+++8 );//11      分为2部分运算，第一个首先a要先加上8，即3+8得出11，然后a++;即a此时==4；
# 判断值的类型
	typeof可以用来判断除null以外的其他原始类型的数据，typeof null == object;
	instanceof 用来判断除function以外的其他复杂类型的数据， typeof 函数fn == function。
# 函数传参中，参数为原始类型的，函数内不管如何改变参数的值，对函数外部的真实值不产生影响.若参数为复杂类型即对象时，函数内若更改参数，那么函数外部的值也将由影响
# js的常用内置对象？（new + 对象）
	Arguments 函数参数集合
	Array 数组
	Boolean 布尔对象
	Date 日期时间
	Error 异常对象
	Function 函数构造器
	Math 数学对象
	Number 数值对象
	Object 基础对象
	RegExp 正则表达式对象
	String 字符串对象
# 谈谈垃圾回收机制的方式及内存管理
      方式：
      	    1.标记清除（首先标记所有变量，还有引用价值的变量去掉标记，剩下有标记的就全部被垃圾回收机制清除）
	    2.引用计数（跟踪每个变量，该变量被引用一次计数就加1，该变量的值变成另一个计数就减1，当该变量计数为0时就被垃圾回收清除）
# [1,2,3].map(parseInt)
      设计到两方面知识
      1.map函数的参数arguments
      2.二进制转换规则
      首先，map函数接收一个function,这个function默认传参为（item,index),也就是说parseInt(item,index);
	parseInt(‘1’, 0); // radix为0时，使用默认的10进制，返回1。
	parseInt(‘2’, 1); // radix值在2-36，无法解析，返回NaN。
	parseInt(‘3’, 2); // 基数为2，2进制数表示的数中，最大值小于3，无法解析，返回NaN。
# ES5和ES6如何取数组的最大值：
	// ES5 的写法
	Math.max.apply(null, [14, 3, 77, 30]);

	// ES6 的写法
	Math.max(...[14, 3, 77, 30]);
# eval()的作用？
	把字符串参数解析成JS代码并运行，并返回执行的结果；
	
	例如：
	eval("2+3");//执行加运算，并返回运算值。
	eval("var age=10");//声明一个age变量
	
	eval的作用域取决于调用它的this的作用域
# DOM2级事件处理程序
	DOM2支持同一dom元素注册多个同种事件，事件发生的顺序按照添加的顺序依次触发（IE是相反的）。
	DOM2事件通过addEventListener和removeEventListener管理
	oDom.addEventListener(eventName,handlers,boolean）中，boolean为true时，则节点事件在捕获时执行，反之则在冒泡时执行。
# 手写ajax的get与post函数
	 function goAjax(url,method,param,callBack){
		 var xmlHttp=null;
		 if(window.xmlHttpRequest)
		 {
		   xmlHttp=new XmlHttpRequest(); 
		 }else{
		  xmlHttp=new ActiveXObject("Microsoft.XMLHTTP");
		 }
		 if(method=="get"||method=="GET")
		 {
		  if(param.length>0)
		  {
		   url=url+"?"+param;
		  }
		  xmlHttp.open("get",url,true);
		  xmlHttp.setRequestHeader("If-Modified-Since","0");//不缓存
		  xmlHttp.send(null);
		 }else{
		  xmlHttp.open("post",url,true);
		  xmlHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
		  xmlHttp.setRequestHeader("If-Modified-Since","0");//不缓存
		  xmlHttp.send(param);
		 }
		 xmlHttp.onreadystatechange=function(){
		  if(xmlHttp.status==200&&xmlHttp.readyState==4)
		  {
		   callBack(xmlHttp. reponseText);
		  }else{
		   alert(xmlHttp.statusText);
		  }
		 }
	}
# JSONP的基本原理
	动态添加一个<script>标签，而script标签的src属性是没有跨域的限制的。
# session的原理
	session是依赖Cookie实现的。session是服务器端对象
	当用户第一次使用session时（表示第一次请求服务器），服务器会创建session，并创建一个Cookie，在Cookie中保存了session的id，发送给客户端。这样客户端就有了自己session的id了。但这个Cookie只在浏览器内存中存在，也就是说，在关闭浏览器窗口后，Cookie就会丢失，也就丢失了sessionId
# $.extend({},obj1,obj2,...)
	var obj1={name:"Tom",age:21}
	1:
	var result=$.extend({},obj1,{name:"Jerry",sex:"Boy"})
 	result=={name:"Jerry",age:21,sex:"Boy"}
	obj1={name:"Tom",age:21}
	
	2:
	var result=$.extend(obj1,{name:"Jerry",sex:"Boy"})
 	result=={name:"Jerry",age:21,sex:"Boy"}
	obj1=={name:"Jerry",age:21,sex:"Boy"}
# 算法之快速排序（贼简单）
	let quickSort = (arr)=>{

	    //停止条件，这部分本应是最后写的，可以跳过之后再看
	    if(arr.length<2){
		return false;
		return arr;
	    }

	    //定义一个基准值，理论上可以是数组中的任意值，但为了形象，取中间index的；
	    let midIndex = Math.floor(arr.length/2);
	    let midNum = arr[midIndex];

	    //定义两个数组，分别存放小于基准值及大于基准值的项
	    let leftAarr = [] ;
	    let rightArr = [] ;

	    //遍历数组，进行排序
	    for(var i = 0 ;i<arr.length ; i++){
		if(arr[i]<midNum){
		    leftAarr.push(arr[i])
		}else if(arr[i]>midNum){
		    rightAarr.push(arr[i])
		}
	    }

	    //递归
	    return quickSort(leftAarr).concat(midNum,quickSort(rightArr));//写到此处要在函数中判断停止条件

	}
# js是单线程的，为什么？ 只知道一方面是避免DOM渲染的冲突。如何解决？引入了同步异步概念
# 异步的实现方式有哪些喃？
	ES6之前：callback、eventloop、Promise
	
	ES6：Generator   ：
	
		function* gen(x){
		  var y = yield x + 2;
		  return y;
		}
		var g = gen(1);
		g.next() // { value: 3, done: false }
		g.next() // { value: undefined, done: true }
		
	ES7:Async/Await
