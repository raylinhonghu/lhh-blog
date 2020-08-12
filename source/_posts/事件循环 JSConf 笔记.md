---

title: 事件循环 JSConf 笔记

category: JavaScript

date: 2019-02-10

author: 林鸿鹄

---


### The Call Stack 
- one thread == one call stack == one thing at a time 

### Example 

Code  

```
function multiply(a,b){
	return a * b
}

function square(n){
	return mutiply(n, n)
}

function printSquare(n){
	var squared = square(n);
	console.log(squared);
}

printSquare(4);
```

Push Stack Bottom to Top


```
   [   multiply(n,n)  ] <=
  [   square(n) ] <=
 [   printSquare(4)  ] <=
[   main()  ] <=
```

Pop Stack Top to Bottom

```
			[   multiply(n,n)  ] => 
		[   square(n) ] => 
[   printSquare(4)  ]
[   main()  ]
```

Push new Stack

```
  [   console.log(squared) ] <=
[   printSquare(4)  ]
[   main()  ]
```

### Stack Overflow

```
function foo() {
	return foo()
}
foo()
```

### Blocking
- what happens when thing are slow ?

Example : network request 

```
var foo = $.getSync('//foo.com')
var foo = $.getSync('//bar.com')
var foo = $.getSync('//qux.com')

console.log(foo)
console.log(bar)
console.log(qux)
```

浏览器被Sync的时候会被 Block， 用户就不能操作。

### 解决方案 - async

```
console.log('hi there')

setTimeout(() => {
	console.log('oh yes')
}, 5000)

console.log('hi second')
```

Call Stack

```
[   console.log('hi there')  ]
[   main()  ]
```

```
[   setTimeout(() => {		]
[	 console.log('oh yes')	]
[   }, 5000) 					]
[   main()  ]
```

```
[   console.log('hi second')  ]
[   main()  ]
```

```
[   console.log('oh yes')  ]
[   main()  ]
```
### Concurrency & the Event Loop

浏览器自带一些 web api

- DOM(document)
- ajax(XMLHttpRequest)
- setTimeout

被 webapi 调用的异步代码完成后会被推到 task queue 直到 stack 为空的时候 eventloop 会负责把 callback 从 taskqueue 推到 stack 里执行。

所以setTimeout 0 秒有用。

```
console.log('hi there')

setTimeout(() => {
	console.log('oh yes')
}, 0)

console.log('hi second')
```


### setTimeout 的时间是最少时间

```
setTimeout(()=>{
	console.log('hi')
},1000)
setTimeout(()=>{
	console.log('hi')
},1000)
setTimeout(()=>{
	console.log('hi')
},1000)
setTimeout(()=>{
	console.log('hi')
},1000)

```


### 不要把慢的东西给 Stack 去执行！

### Scrolling 会推很多任务到 stack queue， 所以最好 debounce

### JSConf 笔记 ：
https://www.youtube.com/watch?v=8aGhZQkoFbQ
