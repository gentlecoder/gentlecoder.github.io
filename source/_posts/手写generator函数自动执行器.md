---
title: 手写generator函数自动执行器
date: 2019-04-07 21:05:51
tags:
  - JavaScript
---

[generator函数了解一下](<http://es6.ruanyifeng.com/#docs/generator>)

### 实现

```js
function runGenerator(generatorFUN, initialValue) {
  const generator = generatorFUN(initialValue)
  step()
  function step(arg, isError) {
    const { value: express, done } = isError
      ? generator.throw(arg)
      : generator.next(arg)
    let response
    if (!done) {
      if (typeof express === 'function') {
        response = express()
      } else {
        response = express
      }
      Promise.resolve(response).then(step, err => step(err, true))
    }
  }
}

function* gen1() {
  yield console.log(1)
  yield console.log(2)
  yield console.log(3)
}
runGenerator(gen1)

function* gen2() {
  var value1 = yield Promise.resolve('promise') //直接返回promise
  console.log(value1)

  var value2 = yield () => Promise.resolve('thunk prommise') //thunk里面返回promise
  console.log(value2)

  var value3 = yield 'string type'
  console.log(value3)

  var value4 = yield () => 'thunk string type'
  console.log(value4)
}

runGenerator(gen2)

function* gen3() {
  try {
    var value1 = yield new Promise((resolve, reject) => setTimeout(reject, 0, 'reject error'));
  } catch (error) {
    console.log(error);
  }
  var value2 = yield 3;
  console.log(value2);
}

runGenerator(gen3);
```

每次递归一遍，将上次yield表达式的结果作为next的参数，遇到函数则执行函数将结果返回，遇到报错则抛出问题。

