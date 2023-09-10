# useId

`useId` 是一个 React Hook，可以生成传递给无障碍属性的唯一 ID。

```react
const id = useId()
```

## 参考 

### `useId()` 

在组件的顶层调用 `useId` 生成唯一 ID：

```react
import { useId } from 'react';
function PasswordField() {
  const passwordHintId = useId();
  // ...
```

#### 参数 

`useId` 不带任何参数。

#### 返回值 

`useId` 返回一个唯一的字符串 ID，与此特定组件中的 `useId` 调用相关联。

#### 注意事项 

- `useId` 是一个 Hook，因此你只能 **在组件的顶层** 或自己的 Hook 中调用它。你不能在内部循环或条件判断中调用它。如果需要，可以提取一个新组件并将 state 移到该组件中。
- `useId` **不应该被用来生成列表中的 key**。[key 应该由你的数据生成](https://zh-hans.react.dev/learn/rendering-lists#where-to-get-your-key)。

## 用法

### 为无障碍属性生成唯一 ID 

在组件的顶层调用 `useId` 生成唯一 ID：

```react
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        密码:
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        密码应该包含至少 18 个字符
      </p>
    </>
  );
}
```

### 为多个相关元素生成 ID 

如果你需要为多个相关元素生成 ID，可以调用 `useId` 来为它们生成共同的前缀：

```react
import { useId } from 'react';

export default function Form() {
  const id = useId();
  return (
    <form>
      <label htmlFor={id + '-firstName'}>名字：</label>
      <input id={id + '-firstName'} type="text" />
      <hr />
      <label htmlFor={id + '-lastName'}>姓氏：</label>
      <input id={id + '-lastName'} type="text" />
    </form>
  );
}
```

### 为所有生成的 ID 指定共享前缀 

如果你在单个页面上渲染多个独立的 React 应用程序，请在 [`createRoot`](https://zh-hans.react.dev/reference/react-dom/client/createRoot#parameters) 或 [`hydrateRoot`](https://zh-hans.react.dev/reference/react-dom/client/hydrateRoot) 调用中将 `identifierPrefix` 作为选项传递。这确保了由两个不同应用程序生成的 ID 永远不会冲突，因为使用 `useId` 生成的每个 ID 都将以你指定的不同前缀开头。

```react
import { createRoot } from 'react-dom/client';
import App from './App.js';
import './styles.css';

const root1 = createRoot(document.getElementById('root1'), {
  identifierPrefix: 'my-first-app-'
});
root1.render(<App />);

const root2 = createRoot(document.getElementById('root2'), {
  identifierPrefix: 'my-second-app-'
});
root2.render(<App />);
```

