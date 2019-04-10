---
title: 手写promise，promise.all，promise.race
date: 2019-04-07 21:31:07
tags:
  - JavaScript
---

### Promise实现

#### 简单写法

```js
const PENDING = Symbol()
const FULFILLED = Symbol()
const REJECTED = Symbol()
const myPromise = function(fn) {
  this.state = PENDING
  this.value = ''
  let resolve = v => {
    if (this.state === PENDING) {
      this.state = FULFILLED
      this.value = v
    }
  }
  let reject = (v) =>{
    if (this.state === PENDING) {
      this.state = REJECTED
      this.value = v
    }
  }
  try {
    fn(resolve, reject)
  } catch (error) {
    reject(error)
  }
}
myPromise.prototype.then = function(onFulfilled, onRejeced) {
  if (this.state === FULFILLED) {
    onFulfilled(this.value)
  } else {
    onRejeced(this.value)
  }
}
const p = new myPromise(function(resolve, reject) {
  c('hahah')
  resolve('hei')
})
p.then(v => c(v))
```

<!--more-->

#### 复杂写法

```js
const PENDING = 1;
const FULFILLED = 2;
const REJECTED = 3;

function MyPromise(executor) {
    let self = this;
    this.resolveQueue = [];
    this.rejectQueue = [];
    this.state = PENDING;
    this.val = undefined;
    function resolve(val) {
        if (self.state === PENDING) {
            setTimeout(() => {
                self.state = FULFILLED;
                self.val = val;
                self.resolveQueue.forEach(cb => cb(val));
            });
        }
    }
    function reject(err) {
        if (self.state === PENDING) {
            setTimeout(() => {
                self.state = REJECTED;
                self.val = err;
                self.rejectQueue.forEach(cb => cb(err));
            });
        }
    }
    try {
        // 回调是异步执行 函数是同步执行
        executor(resolve, reject);
    } catch(err) {
        reject(err);
    }
}

MyPromise.prototype.then = function(onResolve, onReject) {
    let self = this;
    // 不传值的话默认是一个返回原值的函数
    onResolve = typeof onResolve === 'function' ? onResolve : (v => v); 
    onReject = typeof onReject === 'function' ? onReject : (e => { throw e });
    if (self.state === FULFILLED) {
        return new MyPromise(function(resolve, reject) {
            setTimeout(() => {
                try {
                    let x = onResolve(self.val);
                    if (x instanceof MyPromise) {
                        x.then(resolve);
                    } else {
                        resolve(x);
                    }
                } catch(e) {
                    reject(e);
                }
            });
        });
    }

    if (self.state === REJECTED) {
        return new MyPromise(function(resolve, reject) {
            setTimeout(() => {
                try {
                    let x = onReject(self.val);
                    if (x instanceof MyPromise) {
                        x.then(resolve);
                    } else {
                        resolve(x);
                    }
                } catch(e) {
                    reject(e);
                }
            });
        });
    }
    
    if (self.state === PENDING) {
        return new MyPromise(function(resolve, reject) {
            self.resolveQueue.push((val) => {
                try {
                    let x = onResolve(val);
                    if (x instanceof MyPromise) {
                        x.then(resolve);
                    } else {
                        resolve(x);
                    }
                } catch(e) {
                    reject(e);
                }
            });
            self.rejectQueue.push((val) => {
                try {
                    let x = onReject(val);
                    if (x instanceof MyPromise) {
                        x.then(resolve);
                    } else {
                        resolve(x);
                    }
                } catch(e) {
                    reject(e);
                }
            });
        });
    }
}

MyPromise.prototype.catch = function(onReject) {
    return this.then(null, onReject);
}

MyPromise.resolve = function(val) {
    return new MyPromise(function(resolve, reject) {
        resolve(val);
    });
}

MyPromise.reject = function(err) {
    return new MyPromise(function(resolve, reject) {
        reject(err);
    })
}
```

### All实现

```js
MyPromise.all = function(promises) {
    return new MyPromise(function(resolve, reject) {
        let cnt = 0;
        let result = [];
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(res => {
                result[i] = res;
                if (++cnt === promises.length) resolve(result);
            }, err => {
                reject(err);
            })
        }
    });
}
```

今天有个小姐姐说这边有bug，不过当时也没有仔细听清是由于什么原因引起的什么bug。刚刚抽空测试了一下感觉没什么问题，测试如下：

```js
const all = function(promises) {
  return new Promise(function(resolve, reject) {
    let cnt = 0;
    let result = [];
    for (let i = 0; i < promises.length; i++) {
      promises[i].then(
        res => {
          result[i] = res;
          if (++cnt === promises.length) resolve(result);
        },
        err => {
          reject(err);
        }
      );
    }
  });
};

const p1 = new Promise((resolve, reject) => {
  setTimeout(resolve, 2000, 1);
});

const p2 = new Promise((resolve, reject) => {
  setTimeout(resolve, 1000, 2);
});

const p3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 5000, 3);
});

all([p1, p2, p3]).then(
  data => {
    console.log(data); // [1, 2, 3] 结果顺序和promise实例数组顺序是一致的
  },
  err => {
    console.log(err);
  }
);

Promise.all([p1, p2, p3]).then(
  data => {
    console.log(data); // [1, 2, 3] 结果顺序和promise实例数组顺序是一致的
  },
  err => {
    console.log(err);
  }
);
```

这里先都换成了promise。现在想想是不是她说的是出现的结果和实际运行的循序不同呢？不过原生`Promise.all`也是这样的逻辑诶。**<u>有没有哪位大佬看出问题在哪里，麻烦留言一下。</u>**

附上`Promise.all`的定义：

> `p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。
>
> （1）只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。
>
> （2）只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。

### Race实现

```js
MyPromise.race = function(promises) {
    return new MyPromise(function(resolve, reject) {
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(resolve, reject);
        }
    });
}
```

