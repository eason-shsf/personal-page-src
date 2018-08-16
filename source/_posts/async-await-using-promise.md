---
title: 通过promise自己实现一套async-await来学习async-await
date: 2018-05-15 15:14:03
tags:
---
昨天学习koa，koa很依赖Async/Await，但是自己只知道点皮毛，所以研究了一下。    

本来想研究一下Async/Await实现原理，但是没有找到官方实现原理，但是找到了基于Promise实现Async/Await的代码，也颇受启发，根据找到的源码，写一下自己对Async/Await实现原理和目的的一点分析。    
```javascript
async function() {
    let abc = await someAsync()
}
```
首先根据这段代码介绍一点基础知识，对Async/Await做一些介绍。Async/Await为了解决异步编程写法复杂、代码可读性差等问题而设计，目标是让异步编程像同步编程一样简单。    

但是如上所述，someAsync这一句可以是异步语句，也可以是同步语句，但是await标识添加后能够等到整个语句完全执行完并拿到结果。但是我们都知道，someAsync如果是异步函数，结果必然在某个回调函数中（即便用promise其实也算是在then指定的回调函数中），按照同步语句的思路不可能让abc得到返回值，那么async/await是如何实现同步写法得到异步返回值的呢？    

答案是Async/Await要求awati后面的语句返回值必须是Promise，Promise有一个重要的特性就是状态可以从pending变成resolved，这样abc首先拿到了一个pending的Promise，最终会变成resolved并得到value，使整个过程变得可行，所以我们Async/Await作为一个语法糖，很以后可能是用Promise实现的。    

当然这里不探讨那么深，只是学习笔记，只是列出一种可行的Async/Await语法糖的实现。    

下面进入正题：    
```javascript
function asyncFunc(value) {
	return new Promise((resolve, reject) => {
		setTimeout(() => {
			resolve(value * 2)
		}, 3000);
	})
}

async function main1(value) {
	let d = new Date().getTime();
	console.log('starting...');
	let result1 = await asyncFunc(value);
	let result2 = await asyncFunc(result1);
	console.log('ending... ' + result2)
	console.log(new Date().getTime() - d)
}

function main2(value) {
	let d = new Date().getTime()
	console.log('starting...')
	asyncFunc(value).then(res => {
		let result1 = res;
		asyncFunc(result1).then(res => {
			let result2 = res;
			console.log('ending... ' + result2)
			console.log(new Date().getTime() - d)
		})
	})
}
```
首先看结果，两者都能够在6000ms后打印出value的4倍值来，都实现了Async/Await的意图。    

再来看实现，第一种标准的Async/Await实现，没有什么好说的。第二种则嵌套式的使用了Promise.then,可以看出main2（value）函数运行之后得到了这个函数第4行的then返回的那个Promise，这个Promise状态是pending，3000ms后第一个asyncFunc的resolve函数被调用，进入then的这个promise中，然后开始运行第二哥asyncFunc，然后进入第二个then，最终异步执行了两个console.log。可以看出每一个await就代表着一层Promise.then，如果await的结果需要传值，则在生成的then的resolve函数中赋值，从而层层嵌套，实现Async/Await的功能。    

在这里我还看到有的推荐实现方式是如下这样：    
```javascript
function main2(value) {
	let d = new Date().getTime()
	console.log('starting...')
	asyncFunc(value).then(res => {
		let result1 = res;
		return asyncFunc(result1)
	}).then(res => {
		let result2 = res;
		console.log('ending... ' + result2)
		console.log(new Date().getTime() - d)
	})
}
```
这种方式不需要层层嵌套，更符合Promise的用法，在本例中同样实现了这个功能，但是如果最终的结果需要闭包，比如需要打印result1+result2的值，这种方法将无法实现。   

综上所述，Async/Await的实现肯定比上述要复杂很多，会结合上述两种的有点，并使用Promise的诸多特性，以后有机会再做学习。   

    
参考资料：    

1. jayphelps/sweet-async-await
1. Async/Await 原理分析 - CSDN博客