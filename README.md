### 一. 基础知识
### 1. 从 URL 输入到页面展现到底发生什么？
  * DNS域名解析， 解析为IP地址
    > 查找过程：
    >  * 浏览器缓存
    >  * 系统缓存
    >  * 路由器缓存
    >  * ISP的DNS服务器
    >  * 根服务器递归查询

  * TCP三次握手

    > **为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误**
    > * 客户端（浏览器）发送数据包到服务器端口
    > * 服务端返回响应包传达确认信息
    > * 客户端回传数据包， “握手”结束

  * 发送HTTP请求

    > 1. 请求行
    >    * 请求方式：get, post, put, delete, ~~patch, head, options, trace~~
    >    * url
    >    * 请求协议， http版本号
    > 2. 请求头
    > 3. 请求体 (请求参数： query, params, body)

  * 客户端接受请求并返回HTTP报文
    > 返回协议版本，状态码，状态码描述，数据
  * 浏览器解析渲染页面
    > * 解析DOM树
    > * 解析CSS树 
    > * 结合DOM 和 CSS树， 生成渲染树
    > * 计算渲染树节点信息
    > * 绘制页面
    >> * <i>重绘</i>：不影响元素周围或内部布局的属性(颜色)发生改变
    >> * <i>回流</i>：元素尺寸改变，重新渲染
  * 断开连接：TCP第四次握手

### 2. HTTP状态码
  * 2xx: 成功
  * 3XX：重定向
    > 301: 永久重定向；302：临时重定向； 304：发送附带条件的请求时，条件不满足时返回，与重定向无关
  * 4xx: 客户端错误
    > 400: 请求信息中存在语法错误；401： 权限； 403：请求的对应资源禁止被访问； 405: 不支持此方法请求； 408： timeout
  * 5xx: 服务端报错
    > 500： 服务器内部错误； 501: 客户端发起的请求超出服务器的能力范围; 502: 无法连接到其他网关； 503： 服务器停机/维护； 504： timeout

### 3. GET 和 POST的区别
|      |  GET   | POST  |
| ---- |  ----  | ----  |
| 请求参数 | url传递  | request body 中 |
| 安全性 | 低（参数暴露在了链接中） |  高 |
| 请求参数长度限制 | yes |  no |
| 被浏览器主动cache | yes |  no（可以手动设置） |
| 浏览器回退 | 无害  | 再次发起请求 |
| 请求参数完整保留在浏览器历史记录 | yes |  no |
| 参数的数据类型 | ASCII字符 |  没限制 |

### 4. HTTP 和 HTTPS 的区别
|      |  HTTP   | HTTPS  |
| ---- |  ----  | ----  |
| 安全性 | 不安全（数据无法加密）  | 安全 |
| 标准端口 | 80 |  443 |
| 证书 | 无需证书 | 需要CA机构wosign的颁发的SSL证书 |

### 5. 宏任务和微任务
* 1. 宏任务：
  > script(整体代码), **setTimeout**, **setInterval**, I/O, UI交互事件, postMessage

  >(macro)task，可以理解是每次执行栈执行的代码就是一个宏任务（包括每次从事件队列中获取一个事件回调并放到执行栈中执行）。<br>
  >浏览器为了能够使得JS内部(macro)task与DOM任务能够有序的执行，会在一个(macro)task执行结束后，在下一个(macro)task 执行开始前，对页面进行重新渲染<br>
  
* 2. 微任务:
  > **Promise.then**, Object.observe

  >  microtask,可以理解是在当前 task 执行结束后立即执行的任务。也就是说，在当前task任务后，下一个task之前，在渲染之前。<br>
  > 所以它的响应速度相比setTimeout（setTimeout是task）会更快，因为无需等渲染。也就是说，在某一个macrotask执行完后，就会将在它执行期间产生的所有microtask都执行完毕（在渲染前）。<br>

* 3. 运行机制
    * 1. 执行一个宏任务
    * 2. 执行过程中如果遇到微任务，就将它添加到微任务的任务队列中
    * 3. 宏任务执行完毕后， 微任务队列中的任务依次执行
    * 4. 宏任务完毕， 检查渲染
    * 5. 渲染完毕， 执行下一个宏任务

 * 4. Promise和async立即执行

 * 5. await
      > await是一个让出线程的标志。**await后面的表达式会先执行一遍，将await后面的代码加入到microtask中**，然后就会跳出整个async函数来执行后面的代码。<br>

    await 本身就是promise+generator的语法糖
    ```js
    async function async1 () {
      console.log(1);
      await async2();
      console.log(3);
    }
    function async2() {
      console.log(2);
    }
    // 等价于
    function async1() {
      console.log(1);
      Promise.resolve(async2()).then(() => {
        console.log(3);
      })
    }
    ```
  * 6. 试题
    ```js
      async function async1() {
        console.log('async1 start');
        await async2();
        console.log('async1 end');
      }
      async function async2() {
        console.log('async2');
      }

      console.log('script start');

      setTimeout(function() {
          console.log('setTimeout');
      }, 0)

      async1();

      new Promise(function(resolve) {
          console.log('promise1');
          resolve();
      }).then(function() {
          console.log('promise2');
      });
      console.log('script end');

      // script start => async1 start => async2 => promise1 => script end => async1 end => promise2 => setTimeout
      ```  
  
### 二.实例题

### 1. ['1', '2', '3'].map(parseInt)
```js
  //结果： [1, NaN, NaN];
  // map语法
  var new_array = arr.map(function callback(currentValue[,index[, array]]) {
   // Return element for new_array
   }[, thisArg]);
  // parseInt 语法
  // radix: 一个介于2和36之间的整数(数学系统的基础)，表示上述字符串的基数
  parseInt(string, radix);
  
  // 所以例题可改写为：
  ['1', '2', '3'].map((item, index) => parseInt(item, index));
  parseInt('1', 0) // 1
  parseInt('2', 1) // NaN
  parseInt('3', 2) // NaN, 3 不是二进制
  
  // 想得到 [1, 2, 3], 可改写为 ['1', '2', '3'].map(Number)
```

### 2. 数组扁平化去重， 排序
  ```js
    var arrs = [ [1, 2, 2], [3, 4, 5, 5], [6, 7, 8, 9, [11, 12, [12, 13, [14] ] ] ], 10];
    Array.from(new Set(arrs.flat(Infinity))).sort((a, b) => a - b);
    // arr.flat 数组扁平化，Infinity： 任意深度
    // new Set 自动去重
    // Array.from 转化为数组
  ```
  

### 三.原理题
### 1. 函数的防抖和节流
* 防抖
> 触发高频事件后n秒内函数只执行一次，如果n秒内高频事件再次被触发，则重新计算时间
```js
  function debounce(fn, time) {
      const wait = time || 500;
      let timeout = null;
      return function() {
        clearTimeout(timeout);  // 每次触发清除上一次设置的timeout
        timeout = setTimeout(() => { // 创建新的timeout保证，重新计算时间
          fn.apply(this, arguments);
        }, wait);
      }
    }
    function say() { console.log('debounce is success'); }
    const inp = document.getElementById('input');
    inp.addEventListener('input', debounce(say, 500)); // 防抖
```
* 节流
> 持续触发事件，每隔一段时间，只执行一次事件
```js
function throttle(fn, time) {
  let canRun = true; 
  const wait = time || 1000;
  // 事件触发多次时，上一次的setTimeout还没执行完成， 此时canRun还是false，直接被return
  return function () {
    if (!canRun) return; 
    canRun = false; 
    setTimeout(() => {
      fn.apply(this, arguments);
      canRun = true;
    }, wait);
  };
}
function sayHi(e) {
  console.log(e.target.innerWidth, e.target.innerHeight);
}
window.addEventListener('resize', throttle(sayHi, 2000));
```

### Set, Map, WeakSet, WeakMap
  * Set 集合
    > 唯一不重复且无序的数据结构， 存放格式：[value, value];
    ```js
      let arr = [1, 2, 3, 2, 1, 1];
      let set = new Set(arr);
      console.log([...set]); // 1,2,3

      // 元素数量用size
      console.log(arr.length); // undefined
      console.log(arr.size); // 3

      // 操作方法： has, add, delete, clear
    ```
  * WeakSet
    > 将弱引用对象储存在一个集合中

    > WeakSet 与 Set 的区别：
    > * WeakSet只能存储对象引用，不能存放值； Set都可以
    > * WeakSet 对象里有多少个成员元素，取决于垃圾回收机制有没有运行，运行前后成员个数可能不一致，遍历结束之后，有的成员可能取不到了（被垃圾回收了），**WeakSet 对象是无法被遍历的**（ES6 规定 WeakSet 不可遍历），也没有办法拿到它包含的所有元素

  * Map
    > 存放[key, value]形式的值  
    ```js
      const data = new Map([['Bob', 3], ['Alex', 16]]);
      data.get('Bob'); // 3
      data.set('Tom', 10); 
    ```

  * WeakMap
    > WeakMap 对象是一组键值对的集合，其中的**键是弱引用对象**，而值可以是任意。

  * **总结**     
    * Set
      * 成员唯一、无序且不重复
      * [value, value]，键值与键名是一致的（或者说只有键值，没有键名）
      * 可以遍历，方法有：add、delete、has、clear
    * WeakSet
      * 成员都是对象
      * **成员都是弱引用，可以被垃圾回收机制回收**，可以用来保存DOM节点，不容易造成内存泄漏
      * **不能遍历**，方法有add、delete、has
    * Map
      * 本质上是键值对的集合，类似集合 , [key, value]
      * 可以遍历，方法很多可以跟各种数据格式转换
    * WeakMap
      * 只接受对象作为键名（null除外），不接受其他类型的值作为键名
      * **键名是弱引用**，键值可以是任意的，键名所指向的对象可以被垃圾回收，此时键名是无效的
      * **不能遍历**，方法有get、set、has、delete

### 2. apply 和 call ，bind 的原理与区别
> apply和call改变函数运行时的this指向, bind是返回一个改变了上下文后的函数

> 第一个参数都是this， 第二个参数有所区别。apply: 传入带下标的集合，数组或者类数组；call: 其余参数逐个列出
* call
```js
// 原理
Function.prototype.myCall = function(context) {
  // context 运行时的上下文
  context = context ? Object(context) : window; 
  context.fn = this;  // this指代函数(Product)， Product.myCall
  let args = [...arguments].slice(1);  //arguments[0]: this(Food)
  let r = context.fn(...args);
  delete context.fn;
  return r;
};

// 继承
function Product(name, price) {
  this.name = name;
  this.price = price;
}
function Food(name, price) {
  Product.call(this, name, price);
  this.category = 'food';
}
var cheese = new Food('feta', 5);
console.log(cheese);
  // Food { name: 'feta', price: 5, category: 'food' }

// 调用匿名函数
var animals = [
    { species: 'Lion', name: 'King' },
    { species: 'Whale', name: 'Fail' }
  ];
    // this, 将每个数组元素作为指定的this值执行了那个匿名函数
  for (var i = 0; i < animals.length; i++) {
    (function(i) {
        console.log('#' + i + ' ' + this.species + ': ' + this.name) }
    ).call(animals[i], i);
  }
```
* apply
```js
// 原理
Function.prototype.myApply = function(context) {
  context = context ? Object(context) : window;
  context.fn = this;
  const args = arguments[1];
  let r;
  if (!args) {
    r = context.fn();
  } else {
    r = context.fn(...args);
  }
  delete context.fn;
  return r;
};

// 将数组添加到另一个数组
const array = ['a', 'b'];
const elements = [0, 1, 2];
array.push.apply(array, elements);
console.log(array); // ["a", "b", 0, 1, 2]
```
* bind
> 当 new 调用绑定函数的时候，this 参数无效。也就是 new 操作符修改 this 指向的优先级更高
```js
Function.prototype.myBind = function(context) {
  if (typeof this !== "function") {
    throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
  }
  const self = this;
  // 因为第1个参数是指定的this,所以只截取第1个之后的参数
  const args = Array.prototype.slice.call(arguments, 1); 
  const fNOP = function () {};			// 创建一个空对象
  const fBound = function() {
    // 这时的arguments是指bind返回的函数传入的参数
    const bindArgs = Array.prototype.slice.call(arguments);
    // 当作为构造函数时，this 指向实例, 当作为普通函数时，this 指向 window, 绑定context
    return self.apply( this instanceof fNOP ? this : context, args.concat(bindArgs) );
  };
  // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
  fNOP.prototype = this.prototype; 	
  fBound.prototype = new fNOP();		
  return fBound;
}
```

### 3. 深度优先遍历和广度优先遍历
* 深度优先遍历(DFS)
> 找到一个节点后，把它的后辈都找出来，然后再去找相邻的下一个节点。最常用递归法
```html
<div id="root">
    <div id="child1">
        <div id="child1-1"></div>
        <div id="child1-2">
            <div id="child1-2-1"></div>
        </div>
    </div>
    <div id="child2">
        <div id="child2-1"></div>
        <div id="child2-2"></div>
    </div>
</div>
```
```js
function deepTraversal(node,nodeList) {  
  if (node) {    
    nodeList.push(node);    
    var children = node.children;    
    for (var i = 0; i < children.length; i++) {
      //每次递归的时候将  需要遍历的节点  和 节点所存储的数组传下去
       deepTraversal(children[i],nodeList);    
    }
  }    
  return nodeList;  
}  

// 非递归
let deepTraversal3 = (node) => {
  let stack = [];
  let nodes = [];
  if (node) {
    // 推入当前处理的node
    stack.push(node);
    while (stack.length) {
      let item = stack.pop();
      let children = item.children;
      nodes.push(item);
      // node = [] stack = [parent]
      // node = [parent] stack = [child3,child2,child1]
      // node = [parent, child1] stack = [child3,child2,child1-2,child1-1]
      // node = [parent, child1-1] stack = [child3,child2,child1-2]
      for (let i = children.length - 1; i >= 0; i--) {
        stack.push(children[i])
      }
    }
  }
  return nodes
}
var root = document.getElementById('root')
console.log(deepTraversal(root,nodeList=[])); // ['1', '1-1', '1-2', '1-2-1', '2', '2-1', '2-2']
```
* 广度优先遍历（BFS）
> 找到一个节点后，把他同级的兄弟节点都找出来放在前边，把孩子放到后边，最常用 while
```js
function deepTraversal2(node) {
    let nodes = [];
    let stack = [];
    if (node) {
      stack.push(node);
      while (stack.length) {
        let item = stack.shift();
        console.log(item);
        let children = item.children;
        nodes.push(item);
        // 队列，先进先出
        for (let i = 0; i < children.length; i++) {
          stack.push(children[i])
        }
      }
    }
    return nodes
  }
  var root2 = document.getElementById('root');
  console.log(deepTraversal2(root2)); // ['root', '1', '2', '1-1', '1-2', '2-1', '2-2', '1-2-1']
```

