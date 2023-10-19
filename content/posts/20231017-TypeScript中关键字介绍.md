---
title: "TypeScript中关键字介绍"
date: 2023-10-17T20:31:11+08:00
tags: ['Typescript','类型体操']
categories: ["前端篇"]
---

最近学习TS类型体操,其中用到了例如 keyof infer等的关键字,但在日常项目中又不太常用,故有此记录,方便查阅

### extends
#### 用于类型的继承
```ts
interface Animal {
  name:string
  color:string
  gender:'male' | 'female'
}
interface Duck extends Animal{
  sound:string
}

const duck:Duck={
  name:'tang',
  color:'yellow',
  gender:'male',
  sound:'gaga'
}
```
上述代码中Duck继承了父类型,并且又声明了自己独有的类型

#### 用于类型的推断
```ts
type Test1 = 'a' | 'b' | 'c'
type Test2 = 'a' | 's'

type MyExclude<T, U> = T extends U ? never : T;

type TestType = MyExclude<Test1, Test2>
const t: TestType = 'b'
```
当extends两边是联合类型时,就会遵守分配律的规则拆分左侧联合类型,依次来判断拆分后的类型是否满足继承关系,再通过判断继续联合

以上述代码为例

`'a' extends 'a' | 's'  true  never`

`'b' extends 'a' | 's'  false  'b'`

`'c' extends 'a' | 's'  false  'c'`

最终TestType的类型是`'b' | 'c'`



### keyof
keyof用来获取对象类型的所有键的联合类型

```ts
interface Test {
  name: string
  color: string
}

type TestType = keyof Test // "name"|"color"

const t1: TestType = "name"
const t2: TestType = "color"
// const t3: TestType = "gender" // 不能将类型“"gender"”分配给类型“keyof Test”。
```
### infer 
infer是typescript中的高级类型关键字,通常与条件类型一起使用,用于从已知类型中推断出其他类型,允许在类型级别执行复杂的条件分支和推断操作

```ts
type First<T extends any[]> = T extends [infer U, ...any[]] ? U : never;
type FirstType=First<[3, 2, 1]>

const n:FirstType = 3
```

上述代码中infer的作用是推断出第一个数组的第一个元素的类型

### readonly
在typescript中,readonly用于定义对象属性或数组元素为只读(不可修改)的类型

```ts
interface Persion {
  readonly IDCard: string
  readonly name: string
  age: number
}

const person: Persion = {
  IDCard: '136548862113647524',
  name: "zhang",
  age: 19
}

person.name = 'wang'; // 无法为“name”赋值，因为它是只读属性
```

### Exclude
Exclude 是 TypeScript 中内置的一个类型修饰器，用于从一个类型中排除另一个类型。
```ts
interface Test1 {
  name: string
  color: string
}
interface Test2 {
  name: string
}

type TestType = Exclude<keyof Test1, keyof Test2>

const t1: TestType = "name"  // 不能将类型“"name"”分配给类型“"color"”。
```


目前先记录到这里,该篇文章会继续更新...