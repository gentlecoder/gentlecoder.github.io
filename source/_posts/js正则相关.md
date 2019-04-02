---
title: js正则相关
date: 2019-04-01 21:45:34
tags:
  - 正则
---

### 千分位符号

没有小数的情况

```js
c("11111234".replace(/(\d)(?=(\d{3})+$)/g, "$1,"));  // 11,111,234
```

有小数的情况

```js
c("11111234.22".replace(/(\d)(?=(\d{3})+\.)/g, "$1,"));  // 11,111,234.22
```

### 首字母大写

```js
c("abc,sdggs".toLowerCase().replace(/^[a-z]/g, L => L.toUpperCase()));  // Abc,sdggs
```

### 替换字符串前几位

```js
c("asdasfg123".replace(/(^\w{4})/g, "xxxx$`"));  // xxxxsfg123
```

