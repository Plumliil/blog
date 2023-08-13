---
title: "百度Amis+React实践"
date: 2023-06-24T17:09:03+08:00
draft: fasle
tags: ["React", "Amis", "低代码"]
categories: ["前端篇"]
---

### 背景

在项目中有集成低代码平台的想法,经过多方对比最后选择了 amis,主要是需要通过 amis 进行页面配置,导出 json 供移动端和 PC 端进行渲染,所以接下来讲一下近两周研究 amis 的新的以及一些简单经验,供大家参考.

### 什么是 amis

amis 是一个低代码前端框架，它使用 JSON 配置来生成页面，可以减少页面开发工作量，极大提升效率。

### 如何使用 amis

在 amis 官网提供了两种使用 amis 的方式分别是

- JSSDK 可以在任意页面使用
- React 可以在 React 项目中使用

博主是在 umi 框架下结合 React 使用 amis,所以本文主要着重介绍第二种方法

在使用时需要对 amis 进行安装,项目中也需要使用 amis-editor 进行页面配置所以需要同时安装如下两个包

```json
{
  "amis": "^3.1.1",
  "amis-editor": "^5.4.1"
}
```

### amis

首先介绍 amis,amis 提供了 render 方法来对 amis-editor 生成的 JSON 对象页面配置进行渲染,如下,在使用是 render 主要作用就是进行渲染

```jsx
import { render as renderAmis } from "amis";

const App = () => {
  return (
    <div>
      {renderAmis({
        type: "button",
        label: "保存",
        level: "primary",
        onClick: function () {
          console.log("TEST");
        },
      })}
    </div>
  );
};

export default App;
```

### amis-editor

amis-editor 提供了一个编译器组件 `<Editor />`

```tsx
import { useState } from "react";
import { Editor, setSchemaTpl } from "amis-editor";
import type { SchemaObject } from "amis";
import { render as renderAmis } from "amis";
import type { Schema } from "amis/lib/types";
// 以下样式均生效
import "amis/lib/themes/default.css";
import "amis/lib/helper.css";
import "amis/sdk/iconfont.css";
import "amis-editor-core/lib/style.css";
import "amis-ui/lib/themes/antd.css";
type Props = {
  defaultPageConfig?: Schema;
  codeGenHandler?: (codeObject: Schema) => void;
  pageChangeHandler?: (codeObject: Schema) => void;
};

export function Amis(props: Props) {
  const [mobile, setMobile] = useState(false);
  const [preview, setPreview] = useState(false);
  const [defaultPageConfig] = useState<Schema>(props.defaultPageConfig); // 传入配置
  const defaultSchema: Schema | SchemaObject = defaultPageConfig || {
    type: "page",
    body: "",
    title: "标题",
    regions: ["body"],
  };
  const [schema] = useState(defaultSchema);
  let pageJsonObj: Schema = defaultSchema;
  const onChange = (value: Schema) => {
    pageJsonObj = value;
    props.pageChangeHandler && props.pageChangeHandler(value);
  };
  const onSave = () => {
    props.codeGenHandler && props.codeGenHandler(pageJsonObj);
  };
  return (
    <>
      {renderAmis({
        type: "form",
        mode: "inline",
        title: "",
        body: [
          {
            type: "switch",
            option: "预览",
            name: "preview",
            onChange: function (v: any) {
              setPreview(v);
            },
          },
          {
            type: "switch",
            option: "移动端",
            name: "mobile",
            onChange: function (v: any) {
              setMobile(v);
            },
          },
          {
            type: "button",
            label: "保存",
            level: "primary",
            onClick: function () {
              onSave();
            },
          },
          {
            type: "button",
            label: "退出",
            level: "danger",
            onClick: function () {
              // if (!window.confirm('确定退出?')) return;
              if (props.cancleGenHandler) props.cancleGenHandler();
            },
          },
        ],
      })}
      <Editor
        preview={preview}
        isMobile={mobile}
        onChange={onChange}
        value={schema as SchemaObject}
        theme={"antd"}
        onSave={onSave}
      />
    </>
  );
}

export default Amis;
```

在 amis 中提供了两套组件样式供我们选择,分别是 cxd(云舍)和 antd(仿 Antd),我们可以通过设置`Editor`组件中 theme 属性来进行主题的选择,同时需要引入对应的组件样式在以上代码中,我们对`Editor`组件进行了二次封装,暴露出了`defaultPageConfig`(进入编译器默认页面 JSON 配置)属性和`codeGenHandler`(代码生成保存方法),`cancleGenHandler`(退出页面编辑器方法),`pageChangeHandler`(页面改变方法)供外部使用

### 自定义组件

在 amis-editor 中使用的组件可以是我们的自定义组件.在编写自定义组件时特别需要主义的是它的 plugin 配置接下来以`MyButton`为例来进行自定义组件的介绍

首先来介绍以下组件结构

```shell
├─MyButton
  │  ├─comp.tsx # 组件本体
  │  ├─index.tsx # 整体导出
  │  ├─plugin.tsx # 右侧panel配置

```

#### 在`comp.tsx`中主要进行组件的开发

```tsx
import React from "react";
import type { Schema } from "amis/lib/types";
import { Button } from "antd";

const MyButtonRender = React.forwardRef((props: Schema, ref: any) => {
  // const props = this.props
  return (
    <Button
      {...props}
      ref={ref}
      type={props.level || "primary"}
      name={props.name}
    >
      {props.label}
    </Button>
  );
});

class MyButtonRender2 extends React.Component<any, any> {
  handleClick = (nativeEvent: React.MouseEvent<any>) => {
    const { dispatchEvent, onClick } = this.props;
    // const params = this.getResolvedEventParams();
    dispatchEvent(nativeEvent, {});
    onClick?.({});
  };

  handleMouseEnter = (e: React.MouseEvent<any>) => {
    const { dispatchEvent } = this.props;
    // const params = this.getResolvedEventParams();

    dispatchEvent(e, {});
  };

  handleMouseLeave = (e: React.MouseEvent<any>) => {
    const { dispatchEvent } = this.props;
    // const params = this.getResolvedEventParams();

    dispatchEvent(e, {});
  };

  render() {
    return (
      <MyButtonRender
        onClick={this.handleClick}
        onMouseEnter={this.handleMouseEnter}
        onMouseLeave={this.handleMouseLeave}
      >
        {this.props.label}2
      </MyButtonRender>
    );
  }
}

export default MyButtonRender2;
```

在上述代码中`MyButtonRender`简单的对`Button`组件进行了简单的封装,`MyButtonRender2`对 amis 中组件的事件进行了简单的处理并暴露出去

在`plugin.tsx`中主要对`MyButtonRender`组件进行渲染器注册以及对组件的 plugin 进行配置,注册渲染器是为了将自定义组件拖入中间预览区域是可以正常的显示,这一操作与 amis 的工作原理有关(amis 的渲染过程是将 json 转成对应的 React 组件。先通过 json 的 type 找到对应的 Component，然后把其他属性作为 props 传递过去完成渲染。[工作原理
](https://aisuda.bce.baidu.com/amis/zh-CN/docs/extend/internal))

#### 在`plugin.tsx`中进行 panel 配置

```tsx
import { Renderer } from "amis";
import MyButtonRender from "./comp";
import type { BaseEventContext } from "amis-editor-core";
import { BasePlugin } from "amis-editor-core";
import { getSchemaTpl } from "amis-editor-core";
import type { RendererPluginAction, RendererPluginEvent } from "amis-editor";
import { getEventControlConfig } from "amis-editor/lib/renderer/event-control/helper";

// 渲染器注册
Renderer({
  type: "my-button",
  autoVar: true,
  // eslint-disable-next-line @typescript-eslint/ban-ts-comment
  // @ts-ignore
})(MyButtonRender);

export class MyButton extends BasePlugin {
  // 关联渲染器名字
  rendererName = "my-button";
  $schema = "/schemas/ActionSchema.json";
  order = -400;
  // 组件名称
  name = "MyButton";
  isBaseComponent = true;
  description =
    "用来展示一个按钮，你可以配置不同的展示样式，配置不同的点击行为。";
  docLink = "/amis/zh-CN/components/button";
  tags = ["自定义"];
  icon = "fa fa-square";
  pluginIcon = "button-plugin";
  scaffold = {
    type: "my-button",
    label: "MyButton",
    wrapperBody: true,
  };
  previewSchema: any = {
    type: "my-button",
    label: "MyButton",
    wrapperBody: true,
  };

  panelTitle = "MyButton";

  // 事件定义
  events: RendererPluginEvent[] = [
    {
      eventName: "click",
      eventLabel: "点击",
      description: "点击时触发",
      defaultShow: true,
      dataSchema: [
        {
          type: "object",
          properties: {
            nativeEvent: {
              type: "object",
              title: "鼠标事件对象",
            },
          },
        },
      ],
    },
    {
      eventName: "mouseenter",
      eventLabel: "鼠标移入",
      description: "鼠标移入时触发",
      dataSchema: [
        {
          type: "object",
          properties: {
            nativeEvent: {
              type: "object",
              title: "鼠标事件对象",
            },
          },
        },
      ],
    },
    {
      eventName: "mouseleave",
      eventLabel: "鼠标移出",
      description: "鼠标移出时触发",
      dataSchema: [
        {
          type: "object",
          properties: {
            nativeEvent: {
              type: "object",
              title: "鼠标事件对象",
            },
          },
        },
      ],
    },
  ];

  // 动作定义
  actions: RendererPluginAction[] = [];

  panelJustify = true;

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  panelBodyCreator = (context: BaseEventContext) => {
    return getSchemaTpl("tabs", [
      {
        title: "属性",
        body: [
          getSchemaTpl("label", {
            label: "按钮名称",
          }),
          {
            type: "input-text",
            label: "字段名称",
            name: "name",
          },
          {
            type: "select",
            label: "按钮类型",
            name: "level",
            options: [
              {
                label: "默认",
                value: "primary",
              },
              {
                label: "危险",
                value: "danger",
              },
              {
                label: "警告",
                value: "warn",
              },
              {
                label: "成功",
                value: "success",
              },
              {
                label: "浅色",
                value: "default",
              },
            ],
            multiple: false,
            selectFirst: false,
          },
          {
            type: "input-text",
            label: "按钮图标",
            name: "icon",
            // 提示
            labelRemark: {
              icon: 'icon-close',
              trigger: ['hover'],
              className: 'Remark--warning',
              title: '提示',
              content: '图标请从My Iconfont库中选择 部分图标需要加icon-前缀 如close -> icon-close',
            // },
          },
        ],
      },
      {
        title: "样式",
        body: [
          getSchemaTpl("buttonLevel", {
            label: "高亮样式",
            name: "activeLevel",
            visibleOn: "data.active",
          }),

          getSchemaTpl("switch", {
            name: "block",
            label: "块状显示",
          }),

          getSchemaTpl("size", {
            label: "尺寸",
          }),
        ],
      },
      {
        title: "事件",
        className: "p-none",
        body:
          !!context.schema.actionType ||
          ["submit", "reset"].includes(context.schema.type)
            ? [
                getSchemaTpl("eventControl", {
                  name: "onEvent",
                  ...getEventControlConfig(this.manager, context),
                }),
                // getOldActionSchema(this.manager, context)
              ]
            : [
                getSchemaTpl("eventControl", {
                  name: "onEvent",
                  ...getEventControlConfig(this.manager, context),
                }),
              ],
      },
    ]);
  };
}
```

当点选某个组件的时候，编辑器内部会触发面板构建动作，每个插件都可以通过实现 `buildEditorPanel` 来插入右侧面板。通常右侧面板都是表单配置，使用 amis 配置就可以完成。所以推荐的做法是，直接在这个插件上面定义 `panelBody` 或者 `panelBodyCreator` 即可。

具体配置可以参考上述代码,其中需要注意的是`getSchemaTpl`这一方法,该方法通过获取预先通过`setSchemaTpl`设置的模板来进行渲染某些元素组件,一下部分源码可进行参考,tpl 部分全部源码可参考[这里](https://github.com/baidu/amis/blob/master/packages/amis-editor-core/src/tpl.tsx)

```jsx
export function getSchemaTpl(
  name: string,
  patch?: object,
  rendererSchema?: any
): any {
  const tpl = tpls[name] || {};
  let schema = null;

  if (typeof tpl === "function") {
    schema = tpl(patch, rendererSchema);
  } else {
    schema = patch
      ? {
          ...tpl,
          ...patch,
        }
      : tpl;
  }

  return schema;
}

export function setSchemaTpl(name: string, value: any) {
  tpls[name] = value;
}
```

在`index.tsx`中主要进行自定义组件插件的注册以及导出

```tsx
import { registerEditorPlugin } from "amis-editor";
import { MyButton } from "./plugin";

registerEditorPlugin(MyButton);
```

### 其他

在拖拽组件生成页面时,amis-editor 可选择的组件有很多,如果我们想使用自己组建的同时忽略隐藏原有组件可以通过`disabledRendererPlugin`来对原生组件进行隐藏

```tsx
import { registerEditorPlugin, BasePlugin } from "amis-editor";
import type {
  RendererEventContext,
  SubRendererInfo,
  BasicSubRenderInfo,
} from "amis-editor";

/**
 * 用于隐藏一些不需要的Editor组件
 * 备注: 如果不知道当前Editor中有哪些预置组件，可以在这里设置一个断点，console.log 看一下 renderers。
 */

// 需要在组件面板中隐藏的组件
const disabledRenderers = [
  // 'flex',
  "crud2",
  "crud2",
  "crud2",
  // 'crud',
  // 'input-text',
  "input-email",
  "input-password",
  "input-url",
  // "button",
  "reset",
  "submit",
  "tpl",
  "grid",
  "container",
  // 'flex',
  // 'flex',
  "collapse-group",
  "panel",
  "tabs",
  // 'form',
  "service",
  "textarea",
  "input-number",
  // 'select',
  "nested-select",
  "chained-select",
  "dropdown-button",
  "checkboxes",
  "radios",
  "checkbox",
  "input-date",
  "input-date-range",
  "input-file",
  "input-image",
  "input-excel",
  "input-tree",
  "input-tag",
  "list-select",
  "button-group-select",
  "button-toolbar",
  "picker",
  "switch",
  "input-range",
  "input-rating",
  "input-city",
  "transfer",
  "tabs-transfer",
  "input-color",
  "condition-builder",
  "fieldset",
  "combo",
  "input-group",
  "input-table",
  "matrix-checkboxes",
  "input-rich-text",
  "diff-editor",
  "editor",
  "search-box",
  "input-kv",
  "input-repeat",
  "uuid",
  "location-picker",
  "input-sub-form",
  "hidden",
  "button-group",
  "nav",
  "anchor-nav",
  "tooltip-wrapper",
  "alert",
  "wizard",
  "table-view",
  "web-component",
  "audio",
  "video",
  "custom",
  "tasks",
  "each",
  "property",
  "iframe",
  "qrcode",
  "icon",
  "link",
  "list",
  "mapping",
  "avatar",
  "card",
  "card2",
  "cards",
  "table",
  "table2",
  "chart",
  "sparkline",
  "carousel",
  "image",
  "images",
  "date",
  "time",
  "datetime",
  "tag",
  "json",
  "progress",
  "status",
  "steps",
  "timeline",
  "divider",
  "code",
  "markdown",
  "collapse",
  "log",
  "input-array",
  "control",
  "input-datetime",
  "input-datetime-range",
  "formula",
  "group",
  "input-month",
  "input-month-range",
  "input-quarter",
  "input-quarter-range",
  "static",
  "input-time",
  "input-time-range",
  "tree-select",
  "input-year",
  "input-year-range",
  "breadcrumb",
  "custom",
  "hbox",
  "page",
  "pagination",
  "plain",
  "wrapper",
  "column-toggler",
];

export class ManagerEditorPlugin extends BasePlugin {
  order = 9999;
  buildSubRenderers(
    context: RendererEventContext,
    renderers: SubRendererInfo[]
  ): BasicSubRenderInfo | BasicSubRenderInfo[] | void {
    // 更新NPM自定义组件排序和分类
    // console.log(renderers);
    for (let index = 0, size = renderers.length; index < size; index++) {
      // 判断是否需要隐藏 Editor预置组件
      const pluginRendererName = renderers[index].rendererName;
      if (
        pluginRendererName &&
        disabledRenderers.indexOf(pluginRendererName) > -1
      ) {
        renderers[index].disabledRendererPlugin = true; // 更新状态
      }
    }
  }
}

registerEditorPlugin(ManagerEditorPlugin);
```

### 写在最后

一个阶段的结束伴随着另一个阶段的开始,在新的阶段中会继续学习继续进步,如文章有问题通过页脚邮箱进行联系
