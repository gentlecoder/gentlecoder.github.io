---
title: leetcode算法 88
date: 2019-03-29 23:10:12
tags: 
  - 算法
  - leetcode
---

## 题目

给定两个有序整数数组 *nums1* 和 *nums2*，将 *nums2* 合并到 *nums1* 中*，*使得 *num1* 成为一个有序数组。

**说明:**

- 初始化 *nums1* 和 *nums2* 的元素数量分别为 *m* 和 *n*。
- 你可以假设 *nums1* 有足够的空间（空间大小大于或等于 *m + n*）来保存 *nums2* 中的元素。

**示例:**

```js
输入:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

输出: [1,2,2,3,5,6]
```

## 解法1

```js
const arr1 = [1, 3, 5, 9];
const arr2 = [0, 2, 4, 7];
const merge = (arr1, arr2) => arr1.splice(0, arr1.length, ...arr1.concat(arr2).sort());
merge(arr1, arr2);
console.log(arr1)
// [0, 1, 2, 3, 4, 5, 7, 9]
```

这里之前想当然的写了一种native的写法：

```js
(arr1, arr2) => (arr1 = arr1.concat(arr2).sort());
// [1, 3, 5, 9]
```

？？一开始的内心是这样的，说好的引用传递呢，转眼间咋又变回来了呢。后来一想觉得应该还是和引用有关，在函数内给arr1赋值，新建了一个数组对象，这个时候对该对象的修改都做用到了新的内存地址上，而该对象在函数执行完毕后又被回收，并没有改变arr1原来所指内存地址上的内容，所以最终arr1还为原值。总而言之，函数传递的是对象的引用，在函数内给对象重新赋值一个新的对象，并不会做用到原来的引用上。

ps: 再多说两句相应的情况

```js
const arr = [1, 3, 5, 9];
const emptyArr = arr => ((arr = []), arr.push(1));
emptyArr(arr);
console.log(arr); // [1,3,5,9]

const a = { name: "hello" },
  b = { name: "hi" };
const exchange = (a, b) => {
  let c1 = a;
  a = b;
  b = c1;
  a.name += "1";
  b.name += "2";
  c(a.name, b.name);
};
exchange(a, b);
c(a.name, b.name); // hi1 hello2 hello2 hi1
```



## 解法2

Tobe done