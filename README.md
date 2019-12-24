### 一.实例题

#### 1. ['1', '2', '3'].map(parseInt)
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

### 二.原理题
### 1. 函数的防抖和节流
* 防抖
> 触发高频事件后n秒内函数只执行一次，如果n秒内高频事件再次被触发，则重新计算时间
```js
  function debounce(fn, time, immediate) {
      // 创建一个定时器， 在规定时间内再次触发，先清除上次的定时器
      const wait = time || 500;
      let timeout = null;
      return function() {
        timeout && clearTimeout(timeout);
        /**
        * 是否立刻执行函数, 适用场景
        * 是：用户点击按钮增加数量
        * 否：用户输入框输入字符，然后搜索
        */
        if (immediate) {
          // 如果已经执行过，不再执行
          var callNow = !timeout;
          timeout = setTimeout(function(){
            timeout = null;
          }, wait);
          if (callNow) fn.apply(this, arguments)
        } else {
          timeout = setTimeout(() => {
            fn.apply(this, arguments);
          }, wait);
        }
      }
    }
    function say() { console.log('debounce is success'); }
    const inp = document.getElementById('input');
    inp.addEventListener('input', debounce(say, 500, false)); // 防抖
```
* 节流
> 持续触发事件，每隔一段时间，只执行一次事件
```js
function throttle(fn, time) {
  let canRun = true; 
  const wait = time || 1000;
  // 每次触发事件时都判断当前是否有等待执行的延时函数
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
  let args = [...arguments][1];
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
  console.log(deepTraversal2(root2)); // ['1', '2', '1-1', '1-2', '2-1', '2-2', '1-2-1']
```

