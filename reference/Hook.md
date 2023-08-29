# 内置的 Hook

## 状态 Hook

### `useState`

- [`useState`](https://zh-hans.react.dev/reference/react/useState) 声明了一个你可以直接更新的 state 变量。

  ```react
  const [state, setState] = useState(initialState);
  ```

#### 参数 

- `initialState` 你希望 state 初始化的值。它可以是任何类型的值，但对于函数有特殊的行为。在初始渲染后，此参数将被忽略。
  - 如果传递函数作为 `initialState`，则它将被视为 **初始化函数**。它应该是纯函数，不应该接受任何参数，并且应该返回一个任何类型的值。当初始化组件时，React 将调用你的初始化函数，并将其返回值存储为初始状态。
- 传递初始化函数和直接传递初始状态是有区别的。有时，我们需要避免重复创建初始状态，而将初始化函数传递给 `useState`。
- 

#### 返回 

`useState` 返回一个由两个值组成的数组：

1. 当前的 state。在首次渲染时，它将与你传递的 `initialState` 相匹配。
2. [`set` 函数](https://zh-hans.react.dev/reference/react/useState#setstate)，它可以让你将 state 更新为不同的值并触发重新渲染。
3. 当向 `set` 函数传递一个更新函数时，React 会将更新函数放入队列中。然后，在下一次渲染期间，它将按照相同的顺序调用它们。

#### 注意事项 

- `set` 函数 **仅更新 *下一次* 渲染的状态变量**。如果在调用 `set` 函数后读取状态变量，则 [仍会得到在调用之前显示在屏幕上的旧值](https://zh-hans.react.dev/reference/react/useState#ive-updated-the-state-but-logging-gives-me-the-old-value)。
- 如果你提供的新值与当前 `state` 相同（由 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较确定），React 将 **跳过重新渲染该组件及其子组件**。这是一种优化。虽然在某些情况下 React 仍然需要在跳过子组件之前调用你的组件，但这不应影响你的代码。
- React 会 [批量处理状态更新](https://zh-hans.react.dev/learn/queueing-a-series-of-state-updates)。它会在所有 **事件处理函数运行** 并调用其 `set` 函数后更新屏幕。这可以防止在单个事件期间多次重新渲染。在某些罕见情况下，你需要强制 React 更早地更新屏幕，例如访问 DOM，你可以使用 [`flushSync`](https://zh-hans.react.dev/reference/react-dom/flushSync)。
- 在渲染期间，只允许在当前渲染组件内部调用 `set` 函数。React 将丢弃其输出并立即尝试使用新状态重新渲染。这种方式很少需要，但你可以使用它来存储 **先前渲染中的信息**。[请参见下面的示例](https://zh-hans.react.dev/reference/react/useState#storing-information-from-previous-renders)。
- 在严格模式中，React 将 **两次调用你的更新函数**，以帮助你找到 [意外的不纯性](https://zh-hans.react.dev/reference/react/useState#my-initializer-or-updater-function-runs-twice)。这只是开发时的行为，不影响生产。如果你的更新函数是纯函数（本该是这样），就不应影响该行为。其中一次调用的结果将被忽略。



### `useReducer`

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

