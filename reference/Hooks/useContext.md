# useContext

`useContext` 是一个 React Hook，可以让你读取和订阅组件中的 [context](https://zh-hans.react.dev/learn/passing-data-deeply-with-context)。

```react
const value = useContext(SomeContext)
```

## 参考

### `useContext(SomeContext)`

在组件的顶层调用 `useContext` 来读取和订阅 [context](https://zh-hans.react.dev/learn/passing-data-deeply-with-context)。

```react
import { useContext } from 'react';

function MyComponent() {
  const theme = useContext(ThemeContext);
  // ...
```

## 用法

### 向组件树深层传递数据

在组件的最顶级调用 `useContext` 来读取和订阅 [context](https://zh-hans.react.dev/learn/passing-data-deeply-with-context)。

```react
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

`useContext` 返回你向 context 传递的 context value。为了确定 context 值，React 搜索组件树，为这个特定的 context **向上查找最近的** context provider。

若要将 context 传递给 `Button`，请将其或其父组件之一包装到相应的 context provider：

```react
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

function MyPage() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  // ... 在内部渲染 buttons ...
}
```

provider 和 `Button` 之间有多少层组件并不重要。当 `Form` 中的任何位置的 `Button` 调用 `useContext(ThemeContext)` 时，它都将接收 `"dark"` 作为值。

### 通过 context 更新传递的数据 

通常，你会希望 context 随着时间的推移而改变。要更新 context，请将其与 [state](https://zh-hans.react.dev/reference/react/useState) 结合。在父组件中声明一个状态变量，并将当前状态作为 context value 传递给 provider。

```react
function MyPage() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <Button onClick={() => {
        setTheme('light');
      }}>
       Switch to light theme
      </Button>
    </ThemeContext.Provider>
  );
}
```

现在 provider 中的任何一个 `Button` 都会接收到当前的 `theme` 值。如果调用 `setTheme` 来更新传递给 provider 的 `theme` 值，则所有 `Button` 组件都将使用新的值 `'light'` 来重新渲染。

1. 官网同时使用多个 context，并把 provider 抽离成组件的例子和 reducer 和 context 同时使用的例子非常好；

### 指定回退默认值

如果 React 没有在父树中找到该特定 context 的任何 provider，`useContext()` 返回的 context 值将等于你在 [创建 context](https://zh-hans.react.dev/reference/react/createContext) 时指定的 默认值：

```react
const ThemeContext = createContext(null);
```

默认值 **从不改变**。如果你想要更新 context，请按 [上述方式](https://zh-hans.react.dev/reference/react/useContext#updating-data-passed-via-context) 将其与状态一起使用。

通常，除了 `null`，还有一些更有意义的值可以用作默认值，例如：

```react
const ThemeContext = createContext('light');
```

### 覆盖组件树一部分的 context 

通过在 provider 中使用不同的值包装树的某个部分，可以覆盖该部分的 context。

你可以根据需要多次嵌套和覆盖 provider。

### 在传递对象和函数时优化重新渲染

你可以通过 context 传递任何值，包括对象和函数。

用 [`useCallback`](https://zh-hans.react.dev/reference/react/useCallback) 包装 `login` 函数，并将对象创建包装到 [`useMemo`](https://zh-hans.react.dev/reference/react/useMemo) 中。这是一个性能优化的例子：

```react
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

根据以上改变，即使 `MyApp` 需要重新渲染，调用 `useContext(AuthContext)` 的组件也不需要重新渲染，除非 `currentUser` 发生了变化。