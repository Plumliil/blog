---
title: localStorage数据过期时间的设置
date: 2022-11-01 23:51:03
tags: ['TypeScript']
categories: ["前端篇"]
---


使用ts简单封装localStorage函数,使之具有时效性

```ts
class MyLocalStorage {
  private static time: number = new Date().getTime();
  private constructor() {}
  public static get(key: string) {
    let value = localStorage.getItem(key);
    if (!value) return null;
    let valueArr = value.split("|"); // 如果和储存字符冲突可以另换其它字符
    if (valueArr.length && valueArr.length > 1) {
      if (this.time > parseInt(valueArr[1])) {
        localStorage.removeItem(key);
        console.log(this.time, parseInt(valueArr[1]));
        return "";
      } else {
        return JSON.parse(valueArr[0]);
      }
    } else {
      return JSON.parse(value);
    }
  }
  public static set(key: string, value: any) {
    localStorage.setItem(key, JSON.stringify(value));
  }
  public static cacheSet(
    key: string,
    value: any,
    delay: number = 1000 * 60 * 3600
  ) {
    value = JSON.stringify(value) + "|" + (this.time + delay);
    localStorage.setItem(key, value);
  }
  public static remove(key: string) {
    localStorage.removeItem(key);
    return "";
  }
}
```

思路很简单,存入数据时将时间也一并存入添加到数据中并用特殊符号分割,当取出时和缓存时间对比,判断是否需要去除或者移除.