---
title: "Antd及AntdPro组件使用积累"
date: 2023-08-06T14:24:44+08:00
draft: false
---

Ant Design 和 Ant Design Pro 可能是 React 选手在日常工作中接触最多的库了.

接下来就说说在工作用可能会用到但是不经常用,但用的时候会忘的一些属性或方法的使用

### Modal

#### 自定义页脚按钮属性:

![](https://img1.imgtp.com/2023/08/06/tLPCEaZf.png)

传入 okButtonProps 和 cancelButtonProps 可分别自定义确定按钮和取消按钮的 props。

![](https://img1.imgtp.com/2023/08/06/v1rqPCtk.png)

| 参数 | 说明 | 类型 |
| :---------------: | ----------------: | ----------- |
| cancelButtonProps | cancel 按钮 props | ButtonProps |
| okButtonProps | ok 按钮 props | ButtonProps |

```react
<Modal
  title="cancelButtonProps 使用"
  visible={true}
  cancelButtonProps={{
    style: {
      display: "none",
    },
  }}
  okButtonProps={{
    style: {
      color: "white",
      backgroundColor: "pink",
    },
  }}
>
  <h1>隐藏取消按钮</h1>
</Modal>
```

### ProTable

```react

import type { ProColumns, ProFormInstance } from "@ant-design/pro-components";
import { ProTable } from "@ant-design/pro-components";
import { useState } from "react";

export type TableListItem = {
  field: string;
  createdAt: string;
};

const columns: ProColumns<TableListItem>[] = [
  {
    title: "我是字段1字段1字段1",
    dataIndex: "field",
  },
  {
    title: "创建时间",
    dataIndex: "createdAt",
    valueType: "date",
  },
];
const data = [
  {
    field: "1",
    createdAt: "2023-08-05",
  },
];
export default () => {
  const [collapsed, setCollapsed] = useState(false);
  return (
    <>
      <ProTable<TableListItem>
        style={{
          margin: "16px",
        }}
        columns={columns}
        dataSource={data}
        rowKey="key"
        pagination={{
          showSizeChanger: true,
        }}
        search={{
          labelWidth: "auto",
          collapsed,
          onCollapse: setCollapsed,
        }}
        options={false}
        dateFormatter="string"
      />
    </>
  );
};

```

#### 查询栏控制

通过 search 属性可以对 ProTable 表格进行查询配置

###### columns 中模糊查询配置

```json
  {
    title: "类型",
    dataIndex: "selectKey",
    valueType: "select",
    fieldProps: {
      showSearch: true,
    },
  },
```

通过配置 column 中 fieldProps 中 showSearch 属性可以进行模糊查询

###### labelWidth

![](https://img1.imgtp.com/2023/08/06/AlJVqx6D.png)

如上图红框所示,字段长度过长,显示会出问题,此时可以
通过 labelWidth 对查询栏中的字段进行长度配置

![](https://img1.imgtp.com/2023/08/06/BLLi2E0l.png)

```react
search={{
  labelWidth: "auto",
}}
```

###### collapsed

可收起查询字段配置

```react
const [collapsed, setCollapsed] = useState(false);
search={{
    collapsed,
    onCollapse: setCollapsed,
}}
```

#### 表格嵌套

![](https://img1.imgtp.com/2023/08/06/51Yjue6Q.png)

对于数据表格嵌套的实现可以借助 expandable 属性来实现

```react
expandable={{
  // 默认展开行
  defaultExpandAllRows: true,
  // 展开行渲染
  expandedRowRender: (record) =>
    projectContentRender({ record }),
}}
```

完整代码

```tsx
import React, { useRef, useState } from "react";
import type { ActionType, ProColumns } from "@ant-design/pro-table";
import { data } from "./data";
import { ProCard, ProFormField, ProTable } from "@ant-design/pro-components";
function classifyProjectsByYearAndClassification(projects: any) {
  const classifiedProjects: any = [];
  for (const project of projects) {
    const yearEntry = classifiedProjects.find(
      (entry: any) => entry.title === project.grade
    );
    if (!yearEntry) {
      classifiedProjects.push({
        title: project.grade,
        data: {
          one: [],
          two: [],
          three: [],
        },
      });
    }
    const updatedYearEntry = classifiedProjects.find(
      (entry: any) => entry.title === project.grade
    );
    updatedYearEntry.data[project.class].push(project);
  }

  return classifiedProjects;
}

const StudentList = () => {
  const actionRef = useRef<ActionType>();
  const [expandedRowKeys, setExpandedRowKeys] = useState(["高二"]); // 设置初始展开的行的 key
  const dataSource = classifyProjectsByYearAndClassification(data);
  console.log(dataSource);

  const studentColumns: ProColumns[] = [
    {
      title: "姓名",
      dataIndex: "name",
      align: "center",
    },
    {
      title: "语文",
      dataIndex: "subjectYuWen",
      align: "center",
    },
    {
      title: "数学",
      dataIndex: "subjectShuXue",
      align: "center",
    },
    {
      title: "英语",
      dataIndex: "subjectYingYu",
      align: "center",
    },
    {
      title: "排名",
      align: "center",
      dataIndex: "ranking",
    },
    {
      title: "班级",
      dataIndex: "class",
      valueType: "select",
      align: "center",
      render: (val) => {
        return val === "one" ? "一班" : val === "two" ? "二班" : "三班";
      },
    },
    {
      title: "年级",
      dataIndex: "grade",
      align: "center",
      valueType: "select",
    },
  ];
  const gradeColumns: ProColumns[] = [
    {
      title: "",
      dataIndex: "title",
      align: "center",
      hideInSearch: true,
    },
    {
      title: "姓名",
      dataIndex: "name",
      align: "center",
      hideInTable: true,
    },
    {
      title: "语文",
      dataIndex: "subjectYuWen",
      align: "center",
      hideInTable: true,
    },
    {
      title: "数学",
      dataIndex: "subjectShuXue",
      align: "center",
      hideInTable: true,
    },
    {
      title: "英语",
      dataIndex: "subjectYingYu",
      align: "center",
      hideInTable: true,
    },
    {
      title: "排名",
      align: "center",
      dataIndex: "ranking",
      hideInTable: true,
    },
    {
      title: "班级",
      dataIndex: "class",
      valueType: "select",
      align: "center",
      hideInTable: true,
      render: (val) => {
        return val === "one" ? "一班" : val === "two" ? "二班" : "三班";
      },
    },
    {
      title: "年级",
      dataIndex: "grade",
      align: "center",
      hideInTable: true,
      valueType: "select",
    },
  ];
  const classColumns: ProColumns[] = [
    {
      title: "",
      dataIndex: "title",
      align: "center",
    },
  ];
  const studentRender = (params: any) => {
    const { record1, data1 } = params;
    console.log({ record1, data1 });
    return (
      <ProTable
        className="expandedTable"
        scroll={{ x: "max-content" }}
        columns={studentColumns}
        headerTitle={false}
        search={false}
        options={false}
        dataSource={data1[record1.type]}
        pagination={false}
      />
    );
  };
  const classRender = (params: any) => {
    const { record } = params;
    console.log("record", record.data.three);

    return (
      <>
        <ProTable
          rowKey="title"
          scroll={{ x: "max-content" }}
          columns={classColumns}
          expandable={{
            defaultExpandAllRows: true,
            expandedRowRender: (record1) =>
              studentRender({ record1, data1: record.data }),
          }}
          headerTitle={false}
          search={false}
          options={false}
          dataSource={[
            {
              title: "一班",
              data: record.data.one,
              type: "one",
            },
            {
              title: "二班",
              data: record.data.two,
              type: "two",
            },
            {
              title: "三班",
              data: record.data.three,
              type: "three",
            },
          ].filter((item) => item.data.length > 0)}
          pagination={false}
        />
        {record.data.length > 0 && (
          <p
            style={{
              textAlign: "right",
              marginRight: 10,
            }}
          ></p>
        )}
      </>
    );
  };
  return (
    <ProTable
      actionRef={actionRef}
      columns={gradeColumns}
      search={{
        labelWidth: 120,
      }}
      scroll={{ x: "max-content" }}
      dataSource={dataSource}
      rowKey="title"
      expandable={{
        expandedRowKeys,
        expandedRowRender: (record) => classRender({ record }),
        onExpand: (expanded, record) => {
          const keys = expanded
            ? [...expandedRowKeys, record.title]
            : expandedRowKeys.filter((key) => key !== record.title);
          setExpandedRowKeys(keys);
        },
      }}
      dateFormatter="string"
      pagination={false}
    />
  );
};

export default StudentList;
```

### 可编辑表格

#### recordCreatorProps

通过设置 recordCreatorProps 属性中的 newRecordType 为 dataSource 可以取消添加数据限制

```react
recordCreatorProps={{
  newRecordType: "dataSource",
}}
```

### ProFormDependency
依赖某一个字段渲染另一个字段
```tsx
import { ProForm, ProFormDependency, ProFormSelect } from '@ant-design/pro-components';
export default () => {
  const option = [{ label: '1', value: 1 }, { label: '2', value: 2 }, { label: '3', value: 3 }]
  return (
    <div style={{
      marginTop: 200
    }}>
      <ProForm submitter={false}>
        <ProFormSelect
          name="field1"
          options={option}
        />
        <ProFormDependency name={['field1']}>
          {
            ({ field1 }) => {
              if (!field1) return <></>
              const option1 = [{ label: '1-1', value: 11 }, { label: '1-2', value: 12 }, { label: '1-3', value: 13 }]
              const option2 = [{ label: '2-1', value: 21 }, { label: '2-2', value: 22 }, { label: '2-3', value: 23 }]
              const option3 = [{ label: '3-1', value: 31 }, { label: '3-2', value: 32 }, { label: '3-3', value: 33 }]
              return <ProFormSelect
                name="field2"
                options={field1 === 1 ? option1 : field1 === 2 ? option2 : option3}
              />
            }
          }
        </ProFormDependency>
      </ProForm>
    </div>
  );
};
```