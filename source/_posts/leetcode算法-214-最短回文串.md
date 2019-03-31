---
title: leetcode算法 214
date: 2019-03-30 22:33:45
tags:
  - 算法
  - leetcode
---

## 题目

给定一个字符串 ***s***，你可以通过在字符串前面添加字符将其转换为回文串。找到并返回可以用这种方式转换的最短回文串。

**示例 1:**

```
输入: "aacecaaa"
输出: "aaacecaaa"
```

**示例 2:**

```
输入: "abcd"
输出: "dcbabcd"
```

## 解法1

```js
const s = "abcd";
const longestPalindrome = s => {
  if (s.length < 2) return s;
  let start = 0,
    p = [],
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
  return { start: start, len: maxLen, str: s.substr(start, maxLen) };
};
const shortestPalindrome = s => {
  let pstr = longestPalindrome(s);
  let head = s.substring(0, pstr.start);
  let foot = s.substring(pstr.start + pstr.len);
  return (
    [...foot].reverse().join("") +
    head +
    pstr.str +
    [...head].reverse().join("") +
    foot
  );
};
c(shortestPalindrome(s));
```

