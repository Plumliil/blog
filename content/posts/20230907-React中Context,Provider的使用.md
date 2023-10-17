---
title: 'React中Context,Provider的使用'
date: 2023-09-07T22:50:20+08:00
draft: true
---

### Context

Context Hook 是可以让子组件给后代组件共享数据

接受一个上下文对象(从' React.createContext '返回的值)并返回当前* context 值，该值由最近的给定上下文提供程序提供。

通常我们可以通过两步来创建一个

Provider

useReducer

三者结合使用