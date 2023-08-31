# react-official-doc-journey

本仓库用于记录阅读 [React 官方文档](https://react.dev/)过程中，觉得重要的知识点以及个人的思考。

什么是 `JSX` ?

个人理解：`JSX` 就是一系列标签和使用花括号（`{}`） 包裹 JavaScript 的组合。

`JSX` 必须闭合标签。React 组件不能返回多个 JSX 标签。必须将它们包裹到一个共享的父级中。

`React 组件` 是 JavaScript 函数，是返回标签的 JavaScript 函数。 React 组件接收数据并返回应该出现在屏幕上的内容。你可以通过响应交互（例如用户输入）向它们传递新数据。然后，React 将更新屏幕以匹配新数据。

你也可以不用 React 去构建整个页面，而只是将 React 添加到现有的 HTML 页面中，在任何地方呈现交互式的 React 组件。

React 组件必须以大写字母开头，而 HTML 标签则必须是小写字母。

一个好用的[工具](https://transform.tools/html-to-jsx)：将 html 转换为 JSX。

在 React 中，你可以使用 `className` 来指定一个 CSS 的 class。它与 HTML 的 [`class`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/class) 属性的工作方式相同。

以 `use` 开头的函数被称为 **Hook**。

Hook 比普通函数更为严格。你只能在你的组件（或其他 Hook）的 **顶层** 调用 Hook。如果你想在一个条件或循环中使用 `useState`，请提取一个新的组件并在组件内部使用它。
