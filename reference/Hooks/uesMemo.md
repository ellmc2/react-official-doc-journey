# useMemo

`useMemo` 是一个 React Hook，它在每次重新渲染的时候能够缓存计算的结果。

```react
const cachedValue = useMemo(calculateValue, dependencies)
```

## 参考

### `useMemo(calculateValue, dependencies)`

在组件的顶层调用 `useMemo` 来缓存每次重新渲染都需要计算的结果。

```react
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}
```

#### 参数

- `calculateValue`：要缓存计算值的函数。它应该是一个没有任何参数的纯函数，并且可以返回任意类型。React 将会在首次渲染时调用该函数；在之后的渲染中，如果 `dependencies` 没有发生变化，React 将直接返回相同值。否则，将会再次调用 `calculateValue` 并返回最新结果，然后缓存该结果以便下次重复使用。
- `dependencies`：所有在 `calculateValue` 函数中使用的响应式变量组成的数组。响应式变量包括 props、state 和所有你直接在组件中定义的变量和函数。依赖项数组的长度必须是固定的并且必须写成 `[dep1, dep2, dep3]` 这种形式。React 使用 [`Object.is`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 将每个依赖项与其之前的值进行比较。

#### 返回值

在初次渲染时，`useMemo` 返回不带参数调用 `calculateValue` 的结果。

在接下来的渲染中，如果依赖项没有发生改变，它将返回上次缓存的值；否则将再次调用 `calculateValue`，并返回最新结果。

#### 注意

1. `useMemo` 是一个 React Hook，所以你只能 **在组件的顶层** 或者自定义 Hook 中调用它。你不能在循环语句或条件语句中调用它。如有需要，将其提取为一个新组件并使用 state。
2. 除非有特定原因，React **不会丢弃缓存值**。例如，在开发过程中，React 会在你编辑组件文件时丢弃缓存。无论是在开发环境还是在生产环境，如果你的组件在初始挂载期间被终止，React 都会丢弃缓存。在未来，React 可能会添加更多利用丢弃缓存的特性——例如，如果 React 在未来增加了对虚拟化列表的内置支持，那么丢弃那些滚出虚拟化列表视口的缓存是有意义的。你可以仅仅依赖 `useMemo` 作为性能优化手段。否则，使用 [state 变量](https://zh-hans.react.dev/reference/react/useState#avoiding-recreating-the-initial-state) 或者 [ref](https://zh-hans.react.dev/reference/react/useRef#avoiding-recreating-the-ref-contents) 可能更加合适。
3. 这种缓存返回值的方式也叫做 [记忆化（memoization）](https://en.wikipedia.org/wiki/Memoization)，这也是该 Hook 叫做 `useMemo` 的原因。

### 跳过组件的重新渲染

```react
export default function TodoList({ todos, tab, theme }) {
  // 告诉 React 在重新渲染之间缓存你的计算结果...
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // ...所以只要这些依赖项不变...
  );
  return (
    <div className={theme}>
      {/* ... List 也就会接受到相同的 props 并且会跳过重新渲染 */}
      <List items={visibleTodos} />
    </div>
  );
}
```

```react
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```

**通过将 `visibleTodos` 的计算函数包裹在 `useMemo` 中，你可以确保它在重新渲染之间具有相同值**，直到依赖项发生变化。你 **不必** 将计算函数包裹在 `useMemo` 中，除非你出于某些特定原因这样做。在此示例中，这样做的原因是你将它传递给包裹在 [`memo`](https://zh-hans.react.dev/reference/react/memo) 中的组件，这使得它可以跳过重新渲染。添加 `useMemo` 的其他一些原因将在本页进一步描述。

### 记忆另一个 Hook 的依赖

假设你有一个计算函数依赖于直接在组件主体中创建的对象：

```react
function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ 只有当 text 改变时才会发生改变

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ 只有当 allItems 或 serachOptions 改变时才会发生改变
  // ...
```

更好的方法

```react
function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ 只有当 allItems 或者 text 改变的时候才会重新计算
  // ...
```

### 记忆一个函数

```react
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

```react
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```
