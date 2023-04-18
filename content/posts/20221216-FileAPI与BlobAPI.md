---
title: FileAPI与BlobAPI
date: 2022-12-16 21:14:17
---

## File

### 简单介绍
HTML5在DOM上为文件输入元素添加了Files集合,当用户在文件字段中选择一个或多个文件时,这个files及合作中会包含一组File对象,表示被选中的文件,File对象是一种特殊的Blob对象

![](https://s2.loli.net/2022/12/20/yU9vCj7KWzAcgEP.png)

### 创建
- input元素选择文件后返回的FileList对象
- 文件拖放操作生成的DataTransfer对象,`event.dataTransfer.files`
## Blob

### 简单介绍

blob 表示二进制大对象(binary larget object),是 javascript 对不可修改二进制数据的封装类型.包含字符串数组,ArrayBuffers,ArrafBufferViews,甚至其他的 Blob 都可以来创建 blob.blob 对象不可修改,从 blob 中读取内容的唯一方法是使用 FileReader

### 创建

`let blob=new Blob(array,options)`

- 返回一个新创建的 Blob 对象，其内容由参数中给定的数组拼接组成。
- array:由 ArrayBuffer,ArrayBufferView,Blob,DOMString 等对象构成
- options:可以在此指定 MIME 类型
  > MIME 类型:媒体类型（通常称为 Multipurpose Internet Mail Extensions 或 MIME 类型）是一种标准，用来表示文档、文件或字节流的性质和格式。它在 IETF RFC 6838 中进行了定义和标准化。

```js
let blob = new Blob(["hello Blob"], { type: "text/plain" });
console.log(blob); // Blob {size: 10, type: 'text/plain'}
```

### 读取

使用 FileReader 来进行 blob 对象的读取,也可以使用 Blob.prototype.text()来读取字符串格式的 blob 对象
- `FileReader`:![](https://s2.loli.net/2022/12/16/yWnTSQ4Eva1R2Cu.png)D

- `Blob.prototype.text()`:![](https://s2.loli.net/2022/12/16/ZLWAJhU3bKdMV1y.png)


### 属性方法

#### 属性

- size:Blob 对象中所包含数据的大小(字节)
- type:Blob 对象的 MIME 类型,如类型为止则为空字符串

#### 方法

- `Blob.prototype.arrayBuffer()`:返回一个 promise，其会兑现一个包含 Blob 所有内容的二进制格式的 ArrayBuffer。
- `Blob.prototype.slice(start,end,contentType)`:返回一个新的 Blob 对象,可以对原来对象进行复制
  - start:切片起点,默认为 0
  - end:切片重点,默认为 blob.size
  - 设置新的 blob 的,MIME 类型,默认为 blob 原始值
  - ![](https://s2.loli.net/2022/12/16/5WgXF7CEP3euv1J.png)
- `Blob.prototype.stream()`:返回一个能读取 Blob 内容的 ReadableStream。
- `Blob.prototype.text()`:返回一个 promise，其会兑现一个包含 Blob 所有内容的 UTF-8 格式的字符串。
