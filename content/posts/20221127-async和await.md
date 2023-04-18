---
title: async和await
date: 2022-11-27 22:53:08
---

- async,await 是基于 promise,async 函数始终返回一个 Promise 对象
- await 的意思是等待,函数等待 await 执行之后才继续往下执行
- async 和 await 的其中一个作用就是可以优化 Fetch 语法

#### 使用 generator 来实现 async,await

数据获取

```js
function getData(arg) {
  return new Promise((resolve) => {
    let rdm = Math.floor(Math.random() * (1000 - 100) + 100);
    setTimeout(() => {
      resolve(arg);
    }, rdm);
  });
}
```

语法糖

```js
async function aFn() {
  const data1 = await getData(1);
  const data2 = await getData(data1);
  return `success,${data2}`;
}
aFn().then((res) => console.log(res));
```

迭代器

```js
function* fn() {
  const data1 = yield getData(1);
  console.log(data1);
  const data2 = yield getData(data1);
  console.log(data2);
  return `success,${data2}`;
}
function generatorToAsync(genFn) {
  return function () {
    const gen = genFn.apply(this, arguments);
    return new Promise((resolve, reject) => {
      function step(key, arg) {
        let res;
        try {
          res = gen[key](arg);
        } catch (error) {
          return reject(error);
        }
        const { value, done } = res;
        if (done) {
          return resolve(value);
        } else {
          return Promise.resolve(value).then(
            (val) => step("next", val),
            (err) => step("throw", err)
          );
        }
      }
      step("next");
    });
  };
}
const asyncFn = generatorToAsync(fn);
asyncFn().then((res) => console.log(res));
```
