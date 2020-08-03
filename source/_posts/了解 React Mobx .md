---

title: 了解 React Mobx 

category: 

- React
- Mobx

date: 2019-01-02

author: 林鸿鹄

---

## Mobx 

### 前期配置

```
create-react-app bird-cage
yarn run eject 
```

eject 可以让我们更好的去操控我们要的配置，这个命令会生成一个 config 文件，里面包括了很多 webpack文件。

因为我们需要在 Mobx 里面使用到装饰器。所以我们需要安装 babel 的一个配置。

```
yarn add babel-plugin-transform-decorators-legacy 
```
并且在 package.json 或是 .babelrc 里面加入 (现在已经不需要这个步骤)

```
  "babel": {
    "presets": [
      "react-app"
    ],
    "plugins": [
      "transform-decorators-legacy"
    ]
  }
```

这样我们就可以使用装饰器了。

### 安装 Mobx

```
yarn add mobx-react
yarn add mobx
```

### 创建 Store

这次我要创建一个关于鸿鹄们的 store

```
class BirdStore {

}

const store = new BirdStore();

export default store;

```

这里我们导出了一份实例， 因为我们希望所有地方都享有同一份数据。

### observable 

observable 可以用于观察存放数据。
假设我们有一个鸟的数组，那么我们可以写成这样：

```
@observable birds = [];
```

### action

是可以拿来改变被 observe 的数据。

```
@action addBird = (bird) => {
   this.birds.push(bird);
}
```

### computed

computed 就像 excel 里的计算，当 observable的数据被改变时， 可以同步计算展示数据。

```
@computed get birdCount() {
   return this.birds.length();
}
```

最终我们的 BirdStore 是这样子：

```
import { observable, action, computed } from 'mobx';

class BirdStore {
    @observable birds = [];

    @action addBird = (bird) => {
        this.birds.push(bird);
    }

    @computed get birdCount() {
        return this.birds.length();
    }
}

const store = new BirdStore();

export default store;
```

### 注册 Store

和 Redux 类似我们可以提供一个 Provider

```
import { Provider } from 'mobx-react';
import BirdStore from './store/BirdStore';
ReactDOM.render(
	<React.StrictMode>
		<Provider>
			<App BirdStore={BirdStore} />
		</Provider>
	</React.StrictMode>,
	document.getElementById('root')
);
```
注册完之后，在 dom 里会添加一个 inject-app-with-BirdStore, BirdStore 会像 props 一样被传进去。

```
<Provider BirdStore=BirdStore{...}>
	<inject-app-with-BirdStore>
		<App ref=fn() BirdStore=BirdStore{...}>

```

### 使用 Store

我们希望注入的是 BirdStore，
还希望这个我们的组件一直观察 BirdStore 的状态。 假如发生变化，就让这个组件触发 render 函数重新渲染。


```
import { inject, observer } from 'mobx-react';

@inject('BirdStore')
@observer
class App extends React.Component {
	render() { }
}

```

因为所有的 Store 都是通过 Props 传入， 那么我们可以把 BirdStore 取出。

```
const { BirdStore } = this.props;

```
调用 birdCount 得到鸟的数量

```
<h2>You have {BirdStore.birdCount}</h2>
```

调用 addBird 函数 添加新的鸟鸟

```
this.props.BirdStore.addBird(this.bird.value);
```

在控制台输入 $r 可以获取被选中组件的信息

最后完整的组件如下： 

```
import React from 'react';

import { inject, observer } from 'mobx-react';

@inject('BirdStore')
@observer
class App extends React.Component {
	handleSubmit = e => {
		e.preventDefault();
		this.props.BirdStore.addBird(this.bird.value);
		this.bird.value = '';
	};

	render() {
		const { BirdStore } = this.props;

		return (
			<div className="App">
				<h2>You have {BirdStore.birdCount}</h2>

				<form onSubmit={this.handleSubmit}>
					<input type="text" value={this.bird} ref={input => (this.bird = input)}></input>
					<button>Add Bird</button>
				</form>

				<ul>
					{BirdStore.birds.map(e => (
						<li key={e}>e</li>
					))}
				</ul>
			</div>
		);
	}
}

export default App;

```

