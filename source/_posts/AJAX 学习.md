---

title: AJAX 学习

category: AJAX

date: 2018-12-08

author: 林鸿鹄

---

### AJAX 
AJAX = Asynchronous JavaScript and XML（异步的 JavaScript 和 XML）， 现在我们用场不用 XML 而是 JSON 来代替。

### 优点
可以不用刷新页面就可以更新网页里面的内容

### 浏览器差异
在 E7+, Firefox, Chrome, Opera, Safari 浏览器新建 AJAX 链接

```
const xmlreq = new XMLRequestHttp();
```

IE6, IE5 浏览器新建 AJAX 链接

```
const xmlreq = new ActiveXObject("Microsoft.XMLHTTP")
```

### 规定请求类型

```
xmlhttp.open(method, url, async?)
```

如

```
xmlhttp.open("GET","/try/ajax/demo_get2.php?fname=Henry&lname=Ford",true);
xmlhttp.open("POST","/try/ajax/demo_post.php",true);
```

### 向服务器发送请求

如果是 post，send可以传入一个参数。
```
xmlhttp.send()
```

### GET vs POST
1. get  速度快，也更简单
2. post 可以在无法使用缓存，更新数据库的时候使用
3. post 可以拿来上传大文件
4. post 在未知用户输入时 更加稳定


### 设置头文件

```
xmlhttp.setRequestHeader(header, value);
```

如

```
xmlhttp.setRequestHeader("Content-type", "application/x-www.form-urlencoded");
```


### onreadystatechange

```
const btn = document.getElementById('btn');
const body = document.getElementById('body');

btn.addEventlistener('click', function() {
	let ourRequest = new XMLHttpRequest();
	ourRequest.open('GET', 'https://learnwebcode.github.io/json-example/animals-1.json');
	const response = JSON.parse(ourRequest.responseText);
	ourRequest.load = function() {
		if (response.status >= 200 && response.status < 400) {
			body.innerHTML.append(response.pop());
		}
	};
	ourRequest.onerror = function() {};
	ourRequest.send();
});

```

### 服务器响应

- responseText
- responseXML
