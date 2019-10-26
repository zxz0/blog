---
layout: post
title:  "【转】JavaScript Closure"
date:   2019-08-05 12:00:00 -0700
categories: Front-End
tags: Front-End JavaScript Closure Function
description: JavaScript函数闭包
---
### 总结自：
- [Learn JavaScript Closures in 6 Minutes](https://www.freecodecamp.org/news/learn-javascript-closures-in-n-minutes/)
- [JavaScript设计模式精讲 - 03 闭包与高阶函数](https://www.imooc.com/read/38/article/478)

JS支持first-class functions：和其他所有value一样对待，比如strings，numbers还有objects。
- 可以作为variables，可以在arrays中，可以被function return

重点：可以被function return！！
- eg. 
```javascript
function getGreeter() {
    return function() {
        return 'Hi, Jerome!';
    };
}
```
- 要call里面的那个函数，就需要：`getGreeter()(); // 第一个call拿到function，第二个call运行`
- 可以放到variable里面：
```javascript
const greetJerome = getGreeter();
greetJerome();
```
-> (写活之后)
```javascript
// We can greet anyone now!
function getGreeter(name) { // outer function takes name
    return function() {
        return `Hi, ${name}!`; // inner function uses it later
    };
}
const greetYazeed = getGreeter('Yazeed');
greetYazeed(); // Hi, Jerome!
```
- 当一个函数返回的时候，它的lifecycle就complete了，无法再做任何操作，并且local variables会被清理。除非！！它返回另外一个函数。如果是这样，那么这个返回的函数仍可以access outer variables，即使after the parent passes on.

好处：
1. Data Privacy: 安全分享代码。外部者无法maliciously manipulate inner variables。像Java、C++就会有private fields，从class外不可access，所以privacy；JS没有，但是有closure。所以我们可以在outer function内定义一些参数，然后在retured的inner function中使用，而不暴露给外界知道。外界也无法直接access它了，只能通过现成的API。
2. Currying. 函数一次拿到一个argument:
```javascript
const add = function(x) {
    return function(y) {
        return x + y;
    }
}
const add10 = add(10);
add10(20); // 30
```
用于：”preload" a function's arguments for easier reuse. 
3. React会用到。比如去年出的hooks, 其中最confuing的，useEffect，依赖于closures。  
eg. 
	```javascript
	function App() {
	  const username = 'yazeedb';
	  React.useEffect(function() {
	      fetch(`https://api.github.com/users/${username}`) //username是在outer function中定义的，但是在这个inner function中用到了
	      .then(res => res.json())
	      .then(user => console.log(user));
	  });
	  
	  // blah blah blah
	}
	```
	
总结：
1. Functions are values, too.
2. Functions can return other functions.
3. An outer function's variables are still accessible to its inner function, even after the outer has passed on.
4. Those variables are also known as state.
5.	Therefore, closures can also be called stateful functions.

----

闭包实现结果缓存（备忘模式）：
```javascript
/* 备忘函数 */
function memorize(fn) {
    var cache = {}
    return function() {
        var args = Array.prototype.slice.call(arguments) // change Arguments to an array
        var key = JSON.stringify(args) // 函数参数序列化成字符串当作索引change to a string like: [arg]
        return cache[key] || (cache[key] = fn.apply(fn, args)) // get value, of run function (with array arguments, and fn as this environment)
    }
}
```

或者使用ES6实现：
```javascript
function memorize(fn) {
    const cache = {}
    return function(...args) {
        const key = JSON.stringify(args)
        return cache[key] || (cache[key] = fn.apply(fn, args))
    }
}
```
使用方法：
```javascript
/* 复杂计算函数 */
function add(a) {
    return a + 1
}
var adder = memorize(add)
adder(1)            // 输出: 2    当前: cache: { '[1]': 2 }
adder(1)            // 输出: 2    当前: cache: { '[1]': 2 }
adder(2)            // 输出: 3    当前: cache: { '[1]': 2, '[2]': 3 }
```

可以有的改进：
- 只存最新的
- 缓存持久化：cookies, localStorage等

注意：不可以是Map：键是使用===比较的，如果引用类型（可变，eg. 数组）则会出错

高阶函数：
- 函数作为参数：回调函数。
	- eg. Vue或者React等框架里面的hook:
```javascript
function foo(callback) {
// some operations
callback();
}
```
- 函数作为返回值：
	- 一个函数内部输出另一个函数
	- 利用闭包保持作用域