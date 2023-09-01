# useReducer

`useReducer` 是一个 React Hook，它允许你向组件里面添加一个 [reducer](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer)。

```react
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

## 参考

### `useReducer(reducer, initialArg, init?)`

在组件的顶层作用域调用 `useReducer` 以创建一个用于管理状态的 [reducer](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer)。

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

#### 注意事项

- `useReducer` 是一个 Hook，所以只能在 **组件的顶层作用域** 或自定义 Hook 中调用，而不能在循环或条件语句中调用。如果你有这种需求，可以创建一个新的组件，并把 state 移入其中。
- 严格模式下 React 会 **调用两次 reducer 和初始化函数**，这可以 [帮助你发现意外的副作用](https://zh-hans.react.dev/reference/react/useReducer#my-reducer-or-initializer-function-runs-twice)。这只是开发模式下的行为，并不会影响生产环境。只要 reducer 和初始化函数是纯函数（理应如此）就不会改变你的逻辑。其中一个调用结果会被忽略。

### `dispatch` 函数

`useReducer` 返回的 `dispatch` 函数允许你更新 state 并触发组件的重新渲染。它需要传入一个 action 作为参数：

```react
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
```

React 会调用 `reducer` 函数以更新 state，`reducer` 函数的参数为当前的 state 与传递的 action。

#### 参数

- `action`：用户执行的操作。可以是任意类型的值。通常来说 action 是一个对象，其中 `type` 属性标识类型，其它属性携带额外信息。

#### 返回值

`dispatch` 函数没有返回值。

#### 注意

- `dispatch` 函数 **是为下一次渲染而更新 state**。因此在调用 `dispatch` 函数后读取 state [并不会拿到更新后的值](https://zh-hans.react.dev/reference/react/useReducer#ive-dispatched-an-action-but-logging-gives-me-the-old-state-value)，也就是说只能获取到调用前的值。
- 如果你提供的新值与当前的 `state` 相同（使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较），React 会 **跳过组件和子组件的重新渲染**，这是一种优化手段。虽然在跳过重新渲染前 React 可能会调用你的组件，但是这不应该影响你的代码。
- React [会批量更新 state](https://zh-hans.react.dev/learn/queueing-a-series-of-state-updates)。state 会在 **所有事件函数执行完毕** 并且已经调用过它的 `set` 函数后进行更新，这可以防止在一个事件中多次进行重新渲染。如果在访问 DOM 等极少数情况下需要强制 React 提前更新，可以使用 [`flushSync`](https://zh-hans.react.dev/reference/react-dom/flushSync)。

## 用法

### 向组件添加 reducer

在组件的顶层作用域调用 `useReducer` 来创建一个用于管理状态（state）的 [reducer](https://zh-hans.react.dev/learn/extracting-state-logic-into-a-reducer)。

```react
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...


function handleClick() {
  dispatch({ type: 'incremented_age' });
}

function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

通过 `useImmerReducer` 编写简洁的更新逻辑

```react
import { useImmerReducer } from 'use-immer';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex(t =>
        t.id === action.task.id
      );
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];
```

### 避免重新创建初始值

你可以通过给 `useReducer` 的第三个参数传入 **初始化函数** 来解决这个问题
