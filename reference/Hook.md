# 内置的 Hook

## 状态 Hook

- [`useState`](https://zh-hans.react.dev/reference/react/useState) 声明了一个你可以直接更新的 state 变量。

  ```react
  function ImageGallery() {
    const [index, setIndex] = useState(0);
    // ...
  ```

  

- [`useReducer`](https://zh-hans.react.dev/reference/react/useReducer) 声明了一个带有更新逻辑的 state 变量在一个 [reducer 函数](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer) 中。

  ```react
  const [state, dispatch] = useReducer(reducer, initialArg, init?)
                                       
  ```

  ```react
  import { useReducer } from 'react';
  
  function reducer(state, action) {
    // ...
  }
  
  function MyComponent() {
    const [state, dispatch] = useReducer(reducer, { age: 42 });
    // ...
  ```

#### 参数 

- `reducer`：用于更新 state 的纯函数。参数为 state 和 action，返回值是更新后的 state。state 与 action 可以是任意合法值。
- `initialArg`：用于初始化 state 的任意值。初始值的计算逻辑取决于接下来的 `init` 参数。
- **可选参数** `init`：用于计算初始值的函数。如果存在，使用 `init(initialArg)` 的执行结果作为初始值，否则使用 `initialArg`。

#### 返回值 

`useReducer` 返回一个由两个值组成的数组：

1. 当前的 state。初次渲染时，它是 `init(initialArg)` 或 `initialArg` （如果没有 `init` 函数）。
2. [`dispatch` 函数](https://zh-hans.react.dev/reference/react/useReducer#dispatch)。用于更新 state 并触发组件的重新渲染。

#### 注意

- React [会批量更新 state](https://zh-hans.react.dev/learn/queueing-a-series-of-state-updates)。state 会在 **所有事件函数执行完毕** 并且已经调用过它的 `set` 函数后进行更新，这可以防止在一个事件中多次进行重新渲染。如果在访问 DOM 等极少数情况下需要强制 React 提前更新，可以使用 [`flushSync`](https://zh-hans.react.dev/reference/react-dom/flushSync)。

