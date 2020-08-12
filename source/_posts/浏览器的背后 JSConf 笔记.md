---

title: 浏览器的背后 JSConf 笔记

category: JavaScript

date: 2018-11-10

author: 林鸿鹄

---

## 浏览器的背后 JSConf 笔记

### broswer 浏览器

```
1. [binding]
2. [rendering: parse, layout, painting]
3. [platform]
4. [JavaScript VM]
```

本篇只会涉及到第二点 rendering

### High Level Flow 蓝图

```
parse HTML
  			=> render tree => layout => paint
parse CSS

```

### Parsing HTML
- 假如出做会 error out

```
// ok! 浏览器会自动插入响应的 closing tag
<body>
<p class=wat> My first website
<div><span> vistor count: 0
```

- 过程可暂停， 当遇到以下 tag

```
// net work latency
<script> <link> <style>
```

- 可推测 parsing

### Parsing Flow
 
```
Script Execution => Tokeniser => Tree Construction => DOM =>  Script Execution => ....

```

### Tokeniser

```
[<] [div] [>] 
```

### Parse Tree

```
html
	-- head
	-- body
		-- p.wat
			-- #text
		-- div
			-- span
				-- #text
```

Parse Tree 之后会转换成 DOM Tree 如下

### DOM Tree
```
HTMLHtmlElement
	-- HTMLHeadElement
	-- HTMLBodyElement
		-- HTMLParagraphElement
			-- Text
		-- HTMLDivElement
			-- HTMLSpanElement
				-- Text
```


### Performance Insight 性能优化 

- ```<script />``` 放底下不会中断 parsing

### CSS parsing
- 生成 CSS Object Modal （CSSOM）

## 2.Render/ Frame Tree

结合两个 object model 是真正在页面显示的表示

```
Frame Tree = DOM + CSSOM
```

JS 也添加进来了，也会影响树的构建

```
parse HTML
  JS   	=> render tree => layout => paint
parse CSS
```

### DOM NODE => RenderObject

- visual output
- geometric info 几何数据
- can layout and paint 
- hold style & computed metrics 

### Calculating Visual Properties 

- Combines all styles 合成样式
- styles computation  样式计算

## 3.Layout
Layout 有时候是马上的有时候不是马上的

### Immediate Layout

改变 浏览器大小 resize，fontsize 改变，以及JS 获取属性比如（node.offsetHeight）会立即触发 re-Layout。

### 性能优化
最好JS一次性获取所有的信息。

BAD

```
var divHeight1 = div.clientWidth / 1.7; // 读取
div.style.height = divHeight1 + 'px' // 写入

var divHeight2 = div.clientWidth / 1.7; // 读取
div.style.height = divHeight2 + 'px' // 写入
```

Good 

```
var divHeight1 = div.clientWidth / 1.7; // 读取
var divHeight2 = div.clientWidth / 1.7; // 读取

div.style.height = divHeight1 + 'px' // 写入
div.style.height = divHeight2 + 'px' // 写入
```

## 4.Paint
- 获取 render tree 的布局
- 创造图层


## 总结

1. Parsing -> DOM Tree
2. DOM Tree -> Render Tree
3. DOM Tree + CSSOM => Frame Tree 
4. Layout 计算 node 的位置放到页面上
5. Paint 计算位图到页面上


### JSConf Youtube：
https://www.youtube.com/watch?v=SmE4OwHztCc
