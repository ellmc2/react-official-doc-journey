# useRef

`useRef` 是一个 React Hook，它能让你引用一个不需要渲染的值。

```react
const ref = useRef(initialValue)
```

## 参考

```react
useRef(initialValue)
```

在你组件的顶层调用 `useRef` 声明一个 [ref](https://zh-hans.react.dev/learn/referencing-values-with-refs)。

当你希望组件“记住”某些信息，但又不想让这些信息 [触发新的渲染](https://zh-hans.react.dev/learn/render-and-commit) 时，你可以使用 **ref** 。

```react
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

### 参数

- `initialValue`：ref 对象的 `current` 属性的初始值。可以是任意类型的值。这个参数会首次渲染后被忽略。

### 返回值

`useRef` 返回一个只有一个属性的对象:

- `current`：最初，它被设置为你传递的 `initialValue`。之后你可以把它设置为其他值。如果你把 ref 对象作为一个 JSX 节点的 `ref` 属性传递给 React，React 将为它设置 `current` 属性（这句话的具体理解可见通过 ref 操作 DOM）。

在后续的渲染中，`useRef` 将返回同一个对象。

### 注意事项

- 你可以修改 `ref.current` 属性。与 state 不同，它是可变的。然而，如果它持有一个用于渲染的对象（例如，你的 state 的一部分），那么你就不应该修改这个对象。
- 当你改变 `ref.current` 属性时，React 不会重新渲染你的组件。React 不知道你何时改变它，因为 ref 是一个普通的 JavaScript 对象。
- 除了 [初始化](https://zh-hans.react.dev/reference/react/useRef#avoiding-recreating-the-ref-contents) 外不要在渲染期间写入 **或者读取** `ref.current`。这会使你的组件的行为不可预测。
- 在严格模式下，React 将会 **调用两次组件方法**，这是为了 [帮助你发现意外的问题](https://zh-hans.react.dev/reference/react/useRef#my-initializer-or-updater-function-runs-twice)。这只是开发模式下的行为，不影响生产模式。每个 ref 对象将会创建两次，但是其中一个版本将被丢弃。如果你的组件函数是纯的（应该如此），这不会影响其行为。

## 使用方法

### 用 ref 引用一个值

在你的组件的顶层调用 `useRef` 声明一个或多个 [refs](https://zh-hans.react.dev/learn/referencing-values-with-refs)。

```react
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
```

**改变 ref 不会触发重新渲染。** 这意味着 ref 是存储一些不影响组件视图输出的信息的完美选择。例如，如果你需要存储一个 [intervalID](https://developer.mozilla.org/zh-CN/docs/Web/API/setInterval) 并在以后检索它，你可以把它放在一个 ref 中。如果要更新 ref 里面的值，你需要手动改变它的 `current` 属性：

```react
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

在之后，你可以从 ref 中读取 interval ID，这样你就可以 [清除定时器](https://developer.mozilla.org/zh-CN/docs/Web/API/clearInterval)：

```react
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

通过使用 ref，你可以确保：

- 你可以在重新渲染之间 **存储信息**（不像是普通对象，每次渲染都会重置）。
- 改变它 **不会触发重新渲染**（不像是 state 变量，会触发重新渲染）。
- 对于你的组件的每个副本来说，**这些信息都是本地的**（不像是外面的变量，是共享的）。

### 注意

- 不要在渲染期间写入 ref；

- 不要在渲染期间读取 ref；

  ```react
  function MyComponent() {
    // ...
    // 🚩 不要在渲染期间写入 ref
    myRef.current = 123;
    // ...
    // 🚩 不要在渲染期间读取 ref
    return <h1>{myOtherRef.current}</h1>;
  }
  ```

- 可以在 **事件处理程序或者 effects** 中读取和写入 ref。

  ```react
  function MyComponent() {
    // ...
    useEffect(() => {
      // ✅ 你可以在 effects 中读取和写入 ref
      myRef.current = 123;
    });
    // ...
    function handleClick() {
      // ✅ 你可以在事件处理程序中读取和写入 ref
      doSomething(myOtherRef.current);
    }
    // ...
  ```

### 通过 ref 操作 DOM

使用 ref 操作 [DOM](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API) 是非常常见的。React 内置了对它的支持。

首先，声明一个 initial value 为 `null` 的 ref 对象

```react
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
```

然后将你的 ref 对象作为 `ref` 属性传递给你想要操作的 DOM 节点的 JSX：

```react
  // ...
  return <input ref={inputRef} />;
```

当 React 创建 DOM 节点并将其渲染到屏幕时，React 将会把 DOM 节点设置为你的 ref 对象的 `current` 属性。现在你可以访问 `<input>` 的 DOM 节点，并且可以调用类似于 [`focus()`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/focus) 的方法：

```react
  function handleClick() {
    inputRef.current.focus();
  }
```

当节点从屏幕上移除时，React 将把 `current` 属性设回 `null`。

### 避免重复创建 ref 的内容

```react
❌

function Video() {
  const playerRef = useRef(new VideoPlayer());
  // ...

// 虽然 new VideoPlayer() 的结果只会在首次渲染时使用，但是你依然在每次渲染时都在调用这个方法。如果是创建昂贵的对象，这可能是一种浪费。
```

```react
✔️
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

## 疑难解答

### 无法获取自定义组件的 ref

如果你尝试像这样传递 `ref` 到你自己的组件：

```react
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

在控制台中得到这样的错误：

```javascript
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

默认情况下，你自己的组件不会暴露它们内部 DOM 节点的 ref。

为了解决这个问题，首先，找到你想获得 ref 的组件：

```react
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});

export default MyInput;
```

最后，组件就可以得到它的 ref。
