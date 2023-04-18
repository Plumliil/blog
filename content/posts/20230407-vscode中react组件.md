---
title: "vscode中ES7+ React/Redux/React-Native snippets插件使用"
date: 2023-04-07 23:21:32
tags: ['Vscode','插件']
categories: ["工具篇"]
---

通过使用这个插件我们可以很方便的进行组件/方法/文件的导入

本篇博客仅对插件进行介绍翻译,便于自己以后使用

常用片段列表

- imr: 引入 React `import React from 'react'`
- imrc: 导入 React 组件 `import React, { Component } from 'react'`
- imrn: 导入 Import React-Native Element `import { first } from 'react-native'`
- cdm: 快速生成类组件中 componentWillMount 函数`componentWillMount() {}`
- cdup: 快速生成类组件中 componentDidUpdate 函数`componentDidUpdate(prevProps, prevState) {} `
- cwm: 快速生成类组件中 componentWillMont 函数`componentWillMount() {}`
- cwup: 快速生成类组件中 componentWillUpdate 函数`componentWillUpdate(nextProps, nextState) {}`
- rcc: 快捷生成类组件
  ```tsx
  import React, { Component } from "react";
  export default class test extends Component {
  render() {
  return (
  ```
- res: 快捷生成函数组件

```tsx
import React from "react";
const test = () => {
  return <div></div>;
};
export default test;
```

- fcc: 快捷生成类组件(附带类型声明)

```tsx
import * as React from "react";
type Props = {};
type State = {};
export class test extends React.Component<Props, State> {
  render() {
    return <div></div>;
  }
}
```

- fsc: 快捷生成函数组件(箭头函数类型)

```tsx
import * as React from "react";
type Props = {};
export const test = (props: Props) => {
  return <div></div>;
};
```

- cmmb: 生成大块注释

```tsx
/**
 * first
 */
```

- ednf: 生成默认导出函数`export default function first(second) {third}`
- edf: 生成默认导出函数`export default (first) => {second}`

类似的快捷方式还是有很多,在这里仅记录常用的,如需更多请去往[React-Native/React/Redux snippets for es6/es7 version Standard](https://marketplace.visualstudio.com/items?itemName=EQuimper.react-native-react-redux-snippets-for-es6-es7-version-standard)