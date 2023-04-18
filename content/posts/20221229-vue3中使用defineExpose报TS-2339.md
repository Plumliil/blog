---
title: "vue3+ts使用defineExpose报TS:2339"
date: 2022-12-29 19:32:44
tags: ["Vue", "踩坑记录"]
categories: ["前端篇"]
---

开头先把错误贴上

```shell
src/hooks/usePageSearch.ts:9:27
TS2339: Property 'getPageData' does not exist on type '{ $:ComponentInternalInstance; $data: {}; $props: Partial<{}> & Pick<Readonly<ExtractPropTypes<{}>> & VNodeProps &AllowedComponentProps & ComponentCustomProps, "key" | ... 10 more ... | "style">; ... 10 more ...; $watch(source: string | Function, cb:Function, options?: WatchOptions<...> | undefined): WatchStopHan...'.
  >  9 |     pageContentRef.value?.getPageData({})

```

在这里解释一下错误,错误的原因是将导入的组件转换为类型后组件中没有对应的`getPageData`类型

经过测试这个错误只在`vue-cli`中出现,在新版本的`vite`中没有出现

解决办法是将`getPageData`的类型进行定义然后对导入的`pageContent`类型使用`extends`进行继承,具体代码如下

```ts
import { ref } from "vue";
import PageContent from "@/components/page-content";
interface INewPageContent extends InstanceType<typeof PageContent> {
  getPageData(): void;
}

const pageContentRef = ref<INewPageContent>();
pageContentRef.value?.getPageData();
```

组件中

```html
<script setup lang="ts">
  const getpageData=(){
    console.log('get the data here')
  }
  defineExpose({
    getPageData
  })
</script>
```

关于 extends 类型继承可以参考以下代码

```ts
interface Animal {
  eat: boolean;
}
interface Bird extends Animal {
  fly: boolean;
}
interface Fish extends Animal {
  swim: boolean;
}

let bird = <Bird>{};
bird.eat = true;
bird.fly = true;
// bird.swim --> error: Property 'swim' does not exist on type 'Bird'
```

至此问题解决 🎉
