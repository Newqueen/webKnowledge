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
### apply 和 call ，bind 的原理与区别
> apply和call改变函数运行时的this指向, bind是返回一个改变了上下文后的函数

> 第一个参数都是this， 第二个参数有所区别。apply: 传入带下标的集合，数组或者类数组；call: 其余参数逐个列出
* call
```js
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
  console.log(cheese)
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