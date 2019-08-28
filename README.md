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
# setState的同步、异步情况
    生命周期中的 setState 处于一个大的 transaction 中，此时的 isBatchingUpdate 为 true，执行 setState 只会让 dirtyComponents 数组 push 当前组件而不会进一步处理，此时 log 来看的话 state 还是没有变的。而如果在 transaction 之外，例如 setTimeout 里 setState，此时 isBatchingUpdate 为 false，会一路直接执行下来更改 state，所以此时 log 出来 state 是被立刻改变了的。因此 setState 不保证是同步，而不是说它一定是异步
