---

title: Mobx + React Hooks (2020)

category: Mobx

date: 2020-01-25

author: 林鸿鹄

---

##  Mobx + React Hooks (2020)

自从 react hooks 的引进， mobx 也跟随着改版了， 今天我要讲的是如何在 functional component 里面使用 mobx，以及 mobx 如何与 react hooks 进行组合。

### whats's new? 

```
import { useLocalStore, useObserver } from "mobx-react";
```
- useLocalStore  可以用来创建一个 mobx store
- useObserver 可以来包裹一个组件 让他可以去观察变化

### Context

这次我们将使用 React 的 Context 的概念去实现组件的数据通讯， Context 可以让包括的组件以及他的子组件享有 store 包裹的数据。

所以我们创建一个 context： 

```
const StoreContext = React.createContext();

```
接下来， 我们可以创建一个 store 并把它作为 value 提供给 StoreContext。

### useLocalStore

之前说过 useLocalStore 可以用来创建一个 mobx store， 里面返回一个对象，其包含了 observable 数据， 修改数据的方法， 以及 computed value。

下面我们定义了一个 bugs 变量，addBug 的方法，可以往 bugs 里推新的数据， 以及 bugsCount 的 computed value。

```
const store = useLocalStore(() => ({
    bugs: ["Centipede"],
    addBug: bug => {
      store.bugs.push(bug);
    },
    get bugsCount() {
      return store.bugs.length;
    }
}));

```

然后我们把这个 store 传给 StoreContext：

```
const StoreProvider = ({ children }) => {
  const store = useLocalStore(() => ({
    bugs: ["Centipede"],
    addBug: bug => {
      store.bugs.push(bug);
    },
    get bugsCount() {
      return store.bugs.length;
    }
  }));

  return (
    <StoreContext.Provider value={store}>{children}</StoreContext.Provider>
  );
};

```

### 如何连接 mobx 和 组件

我们可以用 Context 包裹着我们的组件， 这样我们的的组件就成功连接到 mobx 啦！

```
export default function App() {
  return (
    <StoreProvider>
      <main>
      </main>
    </StoreProvider>
  );
}
```

### 如何使用

我们需要在我们要 mobx 数据的组件里面使用 useContext 获取 Context 去消费我们前面创建的 StoreContext。

```
const store = React.useContext(StoreContext);
```
然后我们可以从 store 中取出 bugsCount

```
<h1>{store.bugsCount} Bugs!</h1>
```

但是这样的bugsCount 是不会变的。我们需要把这个组件变成可观察的。

```
useObserver(() => <h1>{store.bugsCount} Bugs!</h1>);
```

整体组件如下所示： 

```
const BugsHeader = () => {
  const store = React.useContext(StoreContext);
  return useObserver(() => <h1>{store.bugsCount} Bugs!</h1>);
};

```
假设现在我们要用一个表单去更新数据。同样我们需要 获取到 context， 以及我们需要一个状态去跟踪 bugs。

```
const store = React.useContext(StoreContext);
const [bug, setBug] = React.useState("");
```

然后我们新建一个表单， 然后一个 input 和 button。

```
<form
     onSubmit={e => {
        store.addBug(bug);
        setBug("");
        e.preventDefault();
     }}
>
      <input
        type="text"
        value={bug}
        onChange={e => {
          setBug(e.target.value);
        }}
      />
      <button type="submit">Add</button>
</form>

```

[bug, setBug] 用于 input 的更新。
提交表单我们需要调用 store 的 addBug 函数。

最后完整的代码： 

```
  
import React from "react";
import { useLocalStore, useObserver } from "mobx-react";

const StoreContext = React.createContext();

const StoreProvider = ({ children }) => {
  const store = useLocalStore(() => ({
    bugs: ["Centipede"],
    addBug: bug => {
      store.bugs.push(bug);
    },
    get bugsCount() {
      return store.bugs.length;
    }
  }));

  return (
    <StoreContext.Provider value={store}>{children}</StoreContext.Provider>
  );
};

const BugsHeader = () => {
  const store = React.useContext(StoreContext);
  return useObserver(() => <h1>{store.bugsCount} Bugs!</h1>);
};

const BugsList = () => {
  const store = React.useContext(StoreContext);

  return useObserver(() => (
    <ul>
      {store.bugs.map(bug => (
        <li key={bug}>{bug}</li>
      ))}
    </ul>
  ));
};

const BugsForm = () => {
  const store = React.useContext(StoreContext);
  const [bug, setBug] = React.useState("");

  return (
    <form
      onSubmit={e => {
        store.addBug(bug);
        setBug("");
        e.preventDefault();
      }}
    >
      <input
        type="text"
        value={bug}
        onChange={e => {
          setBug(e.target.value);
        }}
      />
      <button type="submit">Add</button>
    </form>
  );
};

export default function App() {
  return (
    <StoreProvider>
      <main>
        <BugsHeader />
        <BugsList />
        <BugsForm />
      </main>
    </StoreProvider>
  );
}
```
以上就是一些些关于 mobx 2020 的学习啦～ 🙏
