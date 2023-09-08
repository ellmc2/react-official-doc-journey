# useState

useState 是一个 React Hook，它允许你向组件添加一个状态变量。

```react
const [state, setState] = useState(initialState);
```

## 参考

### `useState(initialState)`

在组件的顶层调用 `useState` 来声明一个 [状态变量](https://zh-hans.react.dev/learn/state-a-components-memory)。

```react
import { useState } from 'react';

function MyComponent() {

  const [age, setAge] = useState(28);

  const [name, setName] = useState('Taylor');

  const [todos, setTodos] = useState(() => createTodos());

  // ...
```

#### 参数

- `initialState` 你希望 state 初始化的值。它可以是任何类型的值，但对于函数有特殊的行为。在初始渲染后，此参数将被忽略。
  - 如果传递函数作为 `initialState`，则它将被视为 **初始化函数**。它应该是纯函数，不应该接受任何参数，并且应该返回一个任何类型的值。当初始化组件时，React 将调用你的初始化函数，并将其返回值存储为初始状态。

#### 返回

`useState` 返回一个由两个值组成的数组：

1. 当前的 state。在首次渲染时，它将与你传递的 `initialState` 相匹配。
2. [`set` 函数](https://zh-hans.react.dev/reference/react/useState#setstate)，它可以让你将 state 更新为不同的值并触发重新渲染。

#### 注意事项

- `useState` 是一个 Hook，因此你只能在 **组件的顶层** 或自己的 Hook 中调用它。你不能在循环或条件语句中调用它。如果你需要这样做，请提取一个新组件并将状态移入其中。
- 在严格模式中，React 将 **两次调用初始化函数**，以 [帮你找到意外的不纯性](https://zh-hans.react.dev/reference/react/useState#my-initializer-or-updater-function-runs-twice)。这只是开发时的行为，不影响生产。如果你的初始化函数是纯函数（本该是这样），就不应影响该行为。其中一个调用的结果将被忽略。
