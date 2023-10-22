---
title: 'React中useContext,useReducer简单实现全局状态管理'
date: 2023-09-07T22:50:20+08:00
draft: false
tags: ['React']
categories: ["前端篇"]
---

### useContext
Context Hook 是可以让子组件给后代组件共享数据

接受一个上下文对象(从' React.createContext '返回的值)并返回当前 context 值，该值由最近的给定上下文提供程序提供。

### useReducer

useReducer 是 useState 的另一种选择,当我们要实现的状态逻辑较为复杂时,useReducer 优先级更高,还可以优化触发深度更新组件的性能,因为 useReduce 可以向下出传递 dispatch 而不是回调

useReducer 支持我们传递三个参数, 第一个参数是 reducer 纯函数 第二个参数是我们要使用的初始化值 第三个参数是一个函数,它支持我们对 useReducer 进行初始化操作


### 实操

#### 定义reducer函数

我们可以先通过useReducer来实现主题或者一些其他状态的收集以及修改,基本代码如下:

```typescript
export type State={
  theme:'dark' | 'light'
}
export enum ActionType{
  UPDATE="UPDATE",
}
type UpdateAction={
  type: ActionType
  field: string
  value: any
}

export type Action = UpdateAction

const initStateStorage = JSON.parse(localStorage.getItem('mySiteState') || '{}')

const initData: State={
  theme:'dark',
  ...initStateStorage
}
export function reducer(state: State, action: Action) {
  switch (action.type) {
    case ActionType.UPDATE:
      localStorage.setItem('mySiteState', JSON.stringify({ ...state, [action.field]: action.value }))
      return { ...state, [action.field]: action.value }
    default:
      throw new Error()
  }
}
```

上方代码中抽离出来了一个reducer函数来改变全局状态的值,并定义了一个初始化的状态,之后可以通过判断动作来进行状态的更新,并将状态储存到localStorage中实现持久化

#### 定义上下文

接下来需要定义一个全局容器来提供状态,具体代码如下:

```tsx
import { Dispatch, ReactNode, createContext, useContext, useMemo, useReducer } from "react"
import { Action, State, initState, reducer } from "./reducers/AppReducer"

type AppContextProps = {
  state: State
  dispatch: Dispatch<Action>
}

const AppContext=createContext(null!)

export function useAppContext(){
  return useContext(AppContext)
}

export default function AppContextProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(reducer, initState)
  // 把创建函数和依赖数组项作为参数传入 useMemo，它仅仅会在数组依赖项中的值改变时才会重新计算值
  const contextValue = useMemo(() => {
    return { state, dispatch }
  }, [state, dispatch])
  return <AppContext.Provider value={contextValue}>{children}</AppContext.Provider>
}

```

#### 使用
在main.ts或是其他位置对App组件进行包裹,实现全局状态的使用
```typescript
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.tsx'
import AppContextProvider, { useAppContext } from './AppContext.tsx'

const Container = () => {
  const { state: { theme } } = useAppContext()
  return (
      <div style={{
        backgroundColor: theme === 'dark' ? 'black' : 'white',
        width: '100vw',
        height: '100vh'
      }}>
        <App />
      </div>
  )
}

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <AppContextProvider>
      <Container />
    </AppContextProvider>
  </React.StrictMode>,
)
```

全局状态修改

```tsx
import { useAppContext } from './AppContext'
import { ActionType } from './reducers/AppReducer'
function App() {
  const { state: { loading, theme }, dispatch } = useAppContext()
  const changeTheme = () => {
    if (theme === 'dark') {
      dispatch({
        type: ActionType.UPDATE,
        field: 'theme',
        value: 'light'
      })
    } else {
      dispatch({
        type: ActionType.UPDATE,
        field: 'theme',
        value: 'dark'
      })
    }
  }
  return (
      <button onClick={async () => changeTheme()}>
        changeTheme
      </button>
  )
}

export default App
```