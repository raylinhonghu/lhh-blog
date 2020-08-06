---

title: CSS 动画特效

category: CSS

date: 2019-09-24

author: 林鸿鹄

---

### 动画的写法
```
.element {
	transition: [property] [duration] [ease] [delay]
}

```


### 动画有什么参数？
- font-size
- background-color
- width
- left


### transform

移动

```
transform: translateX(200px);
```

缩放

```
transform: scaleX(0.5);
```

旋转

```
transform: rotateZ(200deg);
```

### transitions

```

.circle {
	background: red;
	width: 100px;
	height: 100px;
	border-radius: 50px;
	
	transition: background 1s, transform 1s 1s ease;
}

.circle:hover {
	background: salmon;
	transform: translateX(20px);

}
```

### keyframe

```
@keyframes drive {
	from {
		transform: translateX(20px);
	}
	to {
		transform: translateY(20px);
	}
}

.mario {
	animation-name: drive;
	animation-duration: 3s;
	animation-fill-mode: forward;
	animation-delay: 2s;
	animation-iteration-count: 3;
	animation-direction: normal;
	animation-timing-function: linear;
}
```


