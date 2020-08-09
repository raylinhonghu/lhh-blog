---

title: React 生命周期

category: React

date: 2018-09-18

author: 林鸿鹄

---

### 生命周期一共被分为四类

1. mounting
2. updating
3. error boundaries
4. unmounting

### constructor
- 会优先于 render 调用
- 可以在这里设置 state， 以及设置 this

执行顺序

```
constructor > render
```

### componentDidMount
- 在 render 之后调用
- 只会调用一次
- 可以在里面写 network 请求

执行顺序

```
constructor > render > componentDidMount
```

### componentDidUpdate(prevProps, preState, snapshot)

- 会在 setState 后调用
- 发生在 render 调用之后


执行顺序


```
constructor > render > componentDidUpdate
```

### componentWillUnmount
- 在组件卸载后调用


执行顺序

```
constructor > render > componentDidMount > componentWillUnmount
```

### shouldComponentUpdate(nextProps, nextState) 
- 可以让 react 知道要不要 re-render
- 假如返回 true， re-render
- 假如返回 false， 不去 re-render

通常这么比较

```
nextProps.xxx && this.props.xxx === nextProps.xxx
```

执行顺序

```
constructor > render > componentDidMount > shouldComponentUpdate
```

### static getDerivedStateFromProps(props,state) 
- 会在所有之前调用
- 在这里可以吧 props 的值复制到 state
- 返回 null 啥都没事
- 也可以返回一个对象给 state

```
static getDerivedStateFromProps(props,state){
	if(props.seed && props.seed !== state.seed) {
		return {
			seed: props.seed,
			counter: props.seed
		}
	}
}
```

执行顺序

```
constructor > getDerivedStateFromProps > componentDidMount 
```


### getSnapshotBeforeUpdate(prevProps, preState)
- 在re-render之前，用于捕获还没在state里的属性
- 这里捕获到的东西会在 componentDidUpdate 那里作为第三个参数


### componentDidCatch(error, info)
- 在有错误的时候可以防止白屏
