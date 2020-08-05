---

title: React 性能优化 

category: React

date: 2019-03-03

author: 林鸿鹄

---

今天来说一下如何在 React 中提升一些些性能。

## 纯函数 （Pure Function）

在了解这些技巧之前，我们先了解一下什么是纯函数，纯函数就是在相同输入的情况下，每次的输出也是一致的。就如一个加法函数： 

```
function add(a, b) {
  return a + b;
}
```

一个不纯的函数就是会从外界改变函数里面的东西，或者从函数里面改变外界的东西，下面这个函数就改变的时间： 

```
function yesterday() {
  const date = new Date();
  date.setDate(date.getDate() - 1);
  return date;
}
```

在 React 里面， 一个组件如果是纯函数，那么他的输出就是由他的“输入”（即 props）决定。假如不是由 input 决定（如他们的 state），那么他就不是纯函数。

所以说假如一个 React 函数都是由 props 决定的那么，他是纯的。因此也就不需要再次渲染函数从而提高效率。


## PureComponent
首先我们先来讲一下 PureComponent:

1. 使用了它所创建的组件会对内部的 state 和 props 进行浅比较(shallow comparsion)。

2. 相当于在 React.Component 上安置了一个
shouldComponentUpdate。

3. 假如没有浅比较没发生变化，组件也不会渲染。

4. 假如一个组件是 PureComponent， 他的子组件也得是 PureComponent。


### 浅比较(shallow comparsion)

原始数据的情况是假如值和类变了，那么就是变了。

复杂数据(数组和对象)是看指向的： 

```
const a = [1,2,3]
const b = [1,2,3]

const c = a;

console.log(a === b) // false!
console.log(a === c) // true!
```

所以说在我们更改数据的时候我们需要新建一个对象或是数组，直接对其操作使数据变化是不好用的！

如

```
a.push('12') // 没用！
```



## React.Memo

在 React 16.6 版本之前， 函数式组件不能和 PureComponent 一样有效率。React.memo 这个高阶函数也可以防止组件重复渲染。

```
import React from "react";

function PureFunction({ seconds }) {
  return <p>I am updating every {seconds} seconds.</p>;
}

function areEqual(prevProps, nextProps) {
  return prevProps.seconds === nextProps.seconds;
}

export default React.memo(PureFunction, areEqual);
```

如代码所示，我们其实可以重写浅比较的方法来作为 memo 的第二个参数， 比较前后的 props。
