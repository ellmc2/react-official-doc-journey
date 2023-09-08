# useEffect

`useEffect` 是一个 React Hook，它允许你 [将组件与外部系统同步](https://zh-hans.react.dev/learn/synchronizing-with-effects)。

```
useEffect(setup, dependencies?)
```

## 参考

### `useEffect(setup, dependencies?)`

在组件的顶层调用 `useEffect` 来声明一个 Effect：

```react
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

#### 参数

- `setup`：处理 Effect 的函数。setup 函数选择性返回一个 **清理（cleanup）** 函数。当组件被添加到 DOM 的时候，React 将运行 setup 函数。在每次依赖项变更重新渲染后，React 将首先使用旧值运行 cleanup 函数（如果你提供了该函数），然后使用新值运行 setup 函数。在组件从 DOM 中移除后，React 将最后一次运行 cleanup 函数。
- **可选** `dependencies`：`setup` 代码中引用的所有响应式值的列表。响应式值包括 props、state 以及所有直接在组件内部声明的变量和函数。如果你的代码检查工具 [配置了 React](https://zh-hans.react.dev/learn/editor-setup#linting)，那么它将验证是否每个响应式值都被正确地指定为一个依赖项。依赖项列表的元素数量必须是固定的，并且必须像 `[dep1, dep2, dep3]` 这样内联编写。React 将使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 来比较每个依赖项和它先前的值。如果省略此参数，则在每次重新渲染组件之后，将重新运行 Effect 函数。如果你想了解更多，请参见 [传递依赖数组、空数组和不传递依赖项之间的区别](https://zh-hans.react.dev/reference/react/useEffect#examples-dependencies)。

#### 返回值

`useEffect` 返回 `undefined`。

#### 注意事项

- `useEffect` 是一个 Hook，因此只能在 **组件的顶层** 或自己的 Hook 中调用它，而不能在循环或者条件内部调用。如果需要，抽离出一个新组件并将 state 移入其中。
- 如果你 **没有打算与某个外部系统同步**，[那么你可能不需要 Effect](https://zh-hans.react.dev/learn/you-might-not-need-an-effect)。
- 当严格模式启动时，React 将在真正的 setup 函数首次运行前，**运行一个开发模式下专有的额外 setup + cleanup 周期**。这是一个压力测试，用于确保 cleanup 逻辑“映射”到了 setup 逻辑，并停止或撤消 setup 函数正在做的任何事情。如果这会导致一些问题，[请实现 cleanup 函数](https://zh-hans.react.dev/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)。
- 如果你的一些依赖项是组件内部定义的对象或函数，则存在这样的风险，即它们将 **导致 Effect 过多地重新运行**。要解决这个问题，请删除不必要的 [对象](https://zh-hans.react.dev/reference/react/useEffect#removing-unnecessary-object-dependencies) 和 [函数](https://zh-hans.react.dev/reference/react/useEffect#removing-unnecessary-function-dependencies) 依赖项。你还可以 [抽离状态更新](https://zh-hans.react.dev/reference/react/useEffect#updating-state-based-on-previous-state-from-an-effect) 和 [非响应式的逻辑](https://zh-hans.react.dev/reference/react/useEffect#reading-the-latest-props-and-state-from-an-effect) 到 Effect 之外。
- 如果你的 Effect 不是由交互（比如点击）引起的，那么 React 会让浏览器 **在运行 Effect 前先绘制出更新后的屏幕**。如果你的 Effect 正在做一些视觉相关的事情（例如，定位一个 tooltip），并且有显著的延迟（例如，它会闪烁），那么将 `useEffect` 替换为 [`useLayoutEffect`](https://zh-hans.react.dev/reference/react/useLayoutEffect)。
- 即使你的 Effect 是由一个交互（比如点击）引起的，**浏览器也可能在处理 Effect 内部的状态更新之前重新绘制屏幕**。通常，这就是你想要的。但是，如果你一定要阻止浏览器重新绘制屏幕，则需要用 [`useLayoutEffect`](https://zh-hans.react.dev/reference/react/useLayoutEffect) 替换 `useEffect`。
- Effect **只在客户端上运行**，在服务端渲染中不会运行。

## 用法

### 连接到外部系统

有些组件需要与网络、某些浏览器 API 或第三方库保持连接，当它们显示在页面上时。这些系统不受 React 控制，所以称为外部系统。

要 [将组件连接到某个外部系统](https://zh-hans.react.dev/learn/synchronizing-with-effects)，请在组件的顶层调用 `useEffect`：

```react
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
  	const connection = createConnection(serverUrl, roomId);
    connection.connect();
  	return () => {
      connection.disconnect();
  	};
  }, [serverUrl, roomId]);
  // ...
}
```

Effect 可以让你的组件与某些外部系统（比如聊天服务）[保持同步](https://zh-hans.react.dev/learn/synchronizing-with-effects)。在这里，外部系统是指任何不受 React 控制的代码，例如：

- 由 [`setInterval()`](https://developer.mozilla.org/zh-CN/docs/Web/API/setInterval) 和 [`clearInterval()`](https://developer.mozilla.org/zh-CN/docs/Web/API/clearInterval) 管理的定时器。
- 使用 [`window.addEventListener()`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/addEventListener) 和 [`window.removeEventListener()`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/removeEventListener) 的事件订阅。
- 一个第三方的动画库，它有一个类似 `animation.start()` 和 `animation.reset()` 的 API。

**如果你没有连接到任何外部系统，[你或许不需要 Effect](https://zh-hans.react.dev/learn/you-might-not-need-an-effect)**。

### 在自定义 Hook 中封装 Effect

Effect 是一个 [“逃生出口”](https://zh-hans.react.dev/learn/escape-hatches)：当你需要“走出 React 之外”或者当你的使用场景没有更好的内置解决方案时，你可以使用它们。如果你发现自己经常需要手动编写 Effect，那么这通常表明你需要为组件所依赖的通用行为提取一些 [自定义 Hook](https://zh-hans.react.dev/learn/reusing-logic-with-custom-hooks)。

例如，这个 `useChatRoom` 自定义 Hook 把 Effect 的逻辑“隐藏”在一个更具声明性的 API 之后：

```react
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

然后你可以像这样从任何组件使用它：

```react
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

### 控制非 React 小部件

### 使用 Effect 请求数据

如果你想手动从 Effect 中请求数据，你的代码可能是这样的：

```react
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    };
  }, [person]);

  // ...
```

如果完全不传递依赖数组，则 Effect 会在组件的 **每次单独渲染（和重新渲染）之后** 运行。

### 在 Effect 中根据先前 state 更新 state

```react
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(c => c + 1); // ✅ 传递一个 state 更新器
    }, 1000);
    return () => clearInterval(intervalId);
  }, []); // ✅现在 count 不是一个依赖项

  return <h1>{count}</h1>;
}
```

### 删除不必要的对象依赖项

避免使用渲染期间创建的对象作为依赖项。相反，在 Effect 内部创建对象：

```react
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    // 👀 看这里
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}
// ...
```

### 删除不必要的函数依赖项

避免使用在渲染期间创建的函数作为依赖项，请在 Effect 内部声明它：

```react
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() {
      return {
        serverUrl: serverUrl,
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
    </>
  );
}

// ...
```

### 从 Effect 读取最新的 props 和 state

```react
function Page({ url, shoppingCart }) {
  const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, shoppingCart.length)
  });

  useEffect(() => {
    onVisit(url);
  }, [url]); // ✅ 所有声明的依赖项
  // ...
}
```

**Effect 事件不是响应式的，必须始终省略其作为 Effect 的依赖项**。这就是让你在其中放置非响应式代码（可以在其中读取某些 props 和 state 的最新值）的原因。通过在 `onVisit` 中读取 `shoppingCart`，确保了 `shoppingCart` 不会使 Effect 重新运行。

### 在服务器和客户端上显示不同的内容

```react
function MyComponent() {
  const [didMount, setDidMount] = useState(false);

  useEffect(() => {
    setDidMount(true);
  }, []);

  if (didMount) {
    // ... 返回仅客户端的 JSX ...
  }  else {
    // ... 返回初始 JSX ...
  }
}
```
