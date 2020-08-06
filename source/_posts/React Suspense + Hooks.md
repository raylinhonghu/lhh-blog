---

title: React Suspense + Hooks

category: React

date: 2019-05-08

author: 林鸿鹄

---

### Code Splitting

在之前的 React， 我们在引入组件的时候都是使用常规的 import 方法。

```
import Nav from './Nav';
```

即使我们分组件，所有的 js 代码还是被打包成一个 bundle.js 文件。 随着组件的数量增多，这个文件就会越大，导致下载这个压缩文件的时长增加。假如 webpack 的 code splitting 类似， React 推出了 lazy loading 以及 suspense。

### Dynamic Loading

之前的动态加载是通过一个回调函数。

```
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

### React.Lazy

React.lazy() 返回一个 promise，必须调用一个动态 import，来载入组件，那个组件必须是 default export 的 React 组件。

```
const Nav = React.lazy(() => import('./Nav'));
```

React Lazy 一定要被 Suspense 接收。 被 lazy load 后的组件会生成不一样的 js 压缩文件。

### Suspense

Suspense 主来是用来处理异步代码，它可以根据 Lazy 的状态来决定渲染什么。

 - fallback 参数是用来显示载入画面的组件。

 - Suspense 可以拿来包裹多个 Lazy 组件。


### 使用 @reach/router

我们首先创建两个组件 Home， Calculation，以默认导出的方式供我们懒加载。

```
import React from 'react';

const Home = () => {
	return <div>Home</div>;
};

export default Home;
```

```
import React from 'react';

const Calculation = () => {
	return <div>Calculation</div>;
};

export default Calculation;
```

```
const Home = lazy(() => import('./Home'));
const Calculation = lazy(() => import('./Calculation'));
```

然后包裹在 Suspense 里面，当第一次访问那个页面的时候，页面会呈现 loading 状态，加载之后切换路由就不会重新下载 js 文件了。

```
import React, { Suspense, lazy } from 'react';
import { Router } from '@reach/router';

const Nav = lazy(() => import('./Nav'));
const Home = lazy(() => import('./Home'));
const Calculation = lazy(() => import('./Calculation'));

const Loading = () => {
	return <nav> Loading...</nav>;
};

const LoadingPage = () => {
	return <nav> Loading Page</nav>;
};

function App() {
	return (
		<div className="App">
			<Suspense fallback={<Loading />}>
				<Nav/>
			</Suspense>

      <Suspense fallback={<LoadingPage />}>
        <Router>
          <Home path="/"/>
          <Calculation path="/calculation"/>
        </Router>
      </Suspense>
		</div>
	);
}

export default App;
```


### 用 Hooks + Suspense 实现请求加载

现在我打算写一个 useFetchSuspense 函数来实现请求加载。

这个 hook 将通过 url 和 fetchOptions 来获取数据并在请求的过程中抛出 promise 让 Suspense 进行渲染。

在数据第一次请求的时候，这个函数会优先去查看缓存里是否存在该数据，假如已经有数据，那么直接返回缓存中的数据。如果缓存中没有数据，那么会用 fetch 去发起一个请求。然后在 fetch promise 结束的时候，设置缓存，并且重新渲染。在 promise pending 的过程中 Suspense 会显示 fallback 的内容。

```
import LRU from 'lru-cache';
import md5 from 'md5';
import produce from 'immer';

const cache = new LRU(50);

const useFetchSuspense = (url, fetchOptions) => {
	const key = `${url}${md5(JSON.stringify(fetchOptions))}`;
	const value = cache.get(key) || { status: 'new', data: null };

	if (value.status === 'resolved') {
		return value.data;
	}

	const promise = fetch(url, fetchOptions).then(response => response.json());

	promise.then(data => {
		cache.set(
			key,
			produce(value, draft => {
				draft.status = 'resolved';
				draft.data = data;
			})
		);
	});

	throw promise;
};

export default useFetchSuspense;

```


### 用 Hooks + Suspense 实现CPU计算
这个和上面的 Hook 类似，但是这次不是发请求，而是 CPU 计算。这里我会使用一个 js 库 “workpool”。 这个库会让代码不会让计算执行在 main thread， 而是会开起 worker thread。

```
import workerpool from "workerpool";

const pool = workerpool.pool();

const promise = pool.exec(func, args); // func: 函数，args 函数的参数， 也会返回一个 promise 可以让 Suspense catch
```

整体代码和 useFetchSuspense 很像： 

```
  
import workerpool from "workerpool";
import LRU from "lru-cache";
import md5 from "md5";

const pool = workerpool.pool();
const cache = new LRU(50);
if (process.env.NODE_ENV === "development") {
  window.useWorkerCache = cache;
}

const useWorker = (func, args) => {
  const key = `${func.name}.${md5(JSON.stringify(args))}`;
  const value = cache.get(key) || { status: "new", data: null };

  if (value.status === "resolved") {
    return value.data;
  }

  const promise = pool.exec(func, args);

  promise.then(result => {
    value.status = "resolved";
    value.data = result;
    cache.set(key, value);
  });

  throw promise;
};

export default useWorker;
```
