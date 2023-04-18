---
title: vue3+ts使用v-for出现unknown问题
date: 2022-12-27 19:00:45
---

最近在写项目时遇到了一个问题,当我从父组件向子组件传数据并且需要将子组件对传入的数据进行`v-for`循环渲染时,在此出遇到了一个ts报错


报错为循环出的`data`类型为`unknown`

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0fa99b77d6104f04924756f91b534911~tplv-k3u1fbpfcp-zoom-1.image)

具体代码如下 :

子组件(修改前) :
~~~ html
<!-- child -->
<template>
  <div class="child">
    <div v-for="data in dataList" :key="data.id">
      {{ data.name }}
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
export default defineComponent({
  props: {
    dataList: {
      type: Array,
      required: true
    }
  },
  setup() {
    return {}
  }
})
</script>
~~~
父组件 :
~~~html
<!-- parent -->
<template>
  <child :dataList="dataList"></child>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import child from './child.vue'
import { IData } from './types'
export default defineComponent({
  components: {
    child
  },
  setup() {
    const dataList: IData[] = [
      { id: 0, name: 'zs', age: 18 },
      { id: 1, name: 'ls', age: 19 },
      { id: 2, name: 'ww', age: 20 }
    ]
    return {
      dataList
    }
  }
})
</script>

<style scoped lang="less"></style>

~~~
类型声明 :
~~~ts
// types
export interface IData {
  id: number
  name: string
  age: number
}
~~~


解决办法使用`props`的时候使用`PropType`将数据类型完整标注即可

> PropType:用于在用运行时 props 声明时给一个 prop 标注更复杂的类型定义。
>
> 更多PropType详情可见 : https://cn.vuejs.org/api/utility-types.html#typing-component-props

解决后的代码

子组件(修改后) :
~~~html
...
<script lang="ts">
import { defineComponent, PropType } from 'vue'
import { IData } from './types'
export default defineComponent({
  props: {
    dataList: {
      // 修改位置在此
      type: Array as PropType<IData[]>,
      required: true
    }
  },
  setup() {
    return {}
  }
})
</script>
...
~~~
此时问题已修复并且data可以进行类型推导

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9eec5bc99da4c728c0f8fb005994bfe~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0b4b3b9a0619467d96db7bddccc3bf3f~tplv-k3u1fbpfcp-zoom-1.image)

至此问题解决 🎉