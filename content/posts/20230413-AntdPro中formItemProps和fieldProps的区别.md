---
title: Antd Pro中formItemProps和fieldProps的区别
date: 2023-04-13 12:50:23
tags: ["React", "Ant Design"]
categories: ["前端篇"]
---

最近在工作中接触到了 antd 和 antd pro,作为一个 react 和 antd 新人,在学习和使用中遇到了不少的问题,下边就常见的一个问题来进行记录,后续还会记录更多的问题以及心得

Form 以及 ProForm 组件在项目中是最常用的组件,Antd 中表单相关组件常用的两个 props 是 fieldProps 和 formItemProps。在做项目配置 columns 时,因为不熟悉时常被这两个 props 搞得很烦,接下来详细解释一下这两个东西

在介绍这两个 Props 之前需要先了解以下`Form.Item`

在 Form 中 Form.Item 用于封装表单项组件,用于渲染表单控件和对应的标签,错误信息等内容,通常被嵌套在 Form 组件中使用,简单使用如下:

```tsx
...
const verifyPassword = (rule, value) => {
  let passwordReg = /^(?![a-zA-Z]+$)(?!\d+$)(?![^\da-zA-Z\s]+$){1,9}$/;
  const flag = passwordReg.test(value);
  if (flag) {
    return Promise.resolve();
  } else {
    return Promise.reject(new Error("密码需由字母、数字、特殊符，任意2种组成，1-9位"));
  }
};
const verifyEmail = (rule, value) => {
  let emailReg =
    /^([a-zA-Z\d][\w-]{2,})@(\w{2,})\.([a-z]{2,})(\.[a-z]{2,})$/;
  const flag = emailReg.test(value);
  console.log(flag);
  if (flag) {
    return Promise.resolve();
  } else {
    return Promise.reject(new Error("邮箱格式不正确"));
  }
};
...
<Form
  name="basic"
  labelCol={{
    span: 8,
  }}
  wrapperCol={{
    span: 16,
  }}
  autoComplete="off"
>
  <Form.Item
    label="Email"
    name="email"
    rules={[
      {
        required: true,
        message: "Please input your username!",
      },
      { validator: verifyEmail },
    ]}
  >
    <Input />
  </Form.Item>

  <Form.Item
    label="Password"
    name="password"
    rules={[
      {
        required: true,
        message: "Please input your password!",
      },
      { validator: verifyPassword },
    ]}
  >
    <Input.Password />
  </Form.Item>
</Form>
```

如上述代码,我们可以在 Form.Item 中进行规则校验,渲染`<Input />`标签,当然可以渲染的不仅仅有`Input`还有一些其他的表单组件

接下来我们看 ProForm 组件,ProForm 在原来的 Form 的基础上增加一些语法糖和更多的布局设置，帮助我们快速的开发一个表单。同时添加一些默认行为，让我们的表单更加好用。

以 ProFormDigit 为例,ProFormDigit 是在 inputNumber 中的基础上进行了二次封装,我们可以直接在 ProFormDigit 上通过 fieldProps 向 inputNumber 组件上进行属性透传,这个属性也可以是类似于 onBlur 之类的事件

```tsx
<ProFormDigit
  label="总量"
  name="totalCount"
  readonly={detailState}
  initialValue={0}
  rules={[
    {
      required: true,
      message: "该字段必填",
    },
  ]}
  placeholder="请输入"
  colProps={{ span: 12 }}
  fieldProps={{
    onBlur: async (e) => {
      if (e) {
        console.log(e);
      }
    },
    precision: 2,
  }}
/>
```

fieldProps 和 formItemProps 在 ProTable 中也经常使用,我们通常在 colums 中进行配置,而且在配置时我们可以使用函数的形式,在函数中进行一些验证之类,值得注意的是如果我们还需要透传其他属性,需要返回一个对象来包裹这些属性,示例如下

```tsx
const columns=[
  ...,
    {
    title: '总量',
    dataIndex: 'totalCount',
    key: 'totalCount',
    width: 150,
    align: 'center',
    fixed: 'left',
    valueType: 'digit',
    formItemProps: {
      rules: [{ required: true, message: '此字段必填' }],
    },
    fieldProps: (form, config: any) => {
      // 金额校验
      const flag = valid(config.entry);
      if (!flag) {
        return {
          addonBefore: (
            // 在输入框前渲染提示组件
            <Tooltip title="金额校验">
              <ExclamationCircleOutlined style={{ color: 'red' }} />
            </Tooltip>
          ),
        };
      }
      return {
        precision: 2,
      };
    },
  ...
]
```

简单来说 fieldProps 用于设置在 FormItem 组件内部渲染的表单字段组件（如 Input、Select 等）的 props。 它是一个对象，接受字段组件的各种 props，例如 value、onChange、disabled、placeholder 等。fieldProps 向下传递到底层表单字段组件，用于控制其行为和外观。

formItemProps 用于设置 FormItem 组件本身的道具。 它是一个对象，它接受 FormItem 组件的各种属性，例如标签、名称、规则、冒号等。formItemProps 用于控制 FormItem 组件的布局和行为，例如其标签、验证规则等。

本篇到此结束
