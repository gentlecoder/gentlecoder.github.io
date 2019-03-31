---
title: leetcode算法 5
date: 2019-03-30 21:29:15
tags:
  - 算法
  - leetcode
---

## 题目

给定一个字符串 `s`，找到 `s` 中最长的回文子串。你可以假设 `s` 的最大长度为 1000。

**示例 1：**

```
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```

**示例 2：**

```
输入: "cbbd"
输出: "bb"
```



## 解法1

```js
const s = "casffaffsgh";
const longestPalindrome = s => {
  if (s.length < 2) return s;
  let start = 0,
    p = [], // 用二维数组保存结果
    maxLen = 0;
  for (let i = 0; i < s.length; i++) {
    for (let j = 0; j < i + 1; j++) {
      p[j] = p[j] || [];
      if (i - j < 3) {
        p[j][i] = s[i] === s[j];
      } else {
        p[j][i] = s[i] === s[j] && p[j + 1][i - 1];
      }
      if (p[j][i] && maxLen < i - j + 1) {
        //判断为回文串且长度大于最大
        start = j;
        maxLen = i - j + 1;
      }
    }
  }
  return s.substr(start, maxLen);
};
c(longestPalindrome(s));
```

