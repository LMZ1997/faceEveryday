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
  
