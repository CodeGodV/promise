# Promise

## 执行过程

> 状态

pending：在微任务队列中，还未执行

fulfilled：已经在主线程中执行完

rejected：已经在主线程中执行完

> js执行顺序

主线程>微任务>宏任务

> 主线程

console.log()

Promise()中代码，例如Promis()中的console.log

> 微任务

Promise().then

> 宏任务

setTimeout、setInterval、文件读取、ajax

```js
# 执行顺序：resolve、1、2、setTimeout

setTimeout(() => {
  console.log('setTimeout') // 宏任务
}, 0)

new Promise((resolve, reject) => {
  resolve()
  console.log('resolve')   // 主线程
}).then(() => {
  console.log('2')  // 微任务
})

// 主线程
console.log('1')

# Promise、1、setTimeout、resolve

# 此时宏任务setTimeout已经被放到主线程中执行了，况且宏任务不执行也不会创建微任务
# 执行过程：1、new Promise中的回调函数（主线程）执行，遇到setTimeout放到宏任务队列中
# 2、往下执行console.log("Promise")
# 3、往下执行connsole.log("1")
# 4、循环宏任务队列中的任务，取出放到主线程中执行
# 5、执行console.log('setTimeout')
# 6、执行resolve()，改变Promise状态，将promise.then放到微任务队列中
# 7、循环微任务队列中的任务，取出放到主线程中执行
# 8、执行console.log("resolve")
new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('setTimeout')
    resolve()
  })
  console.log("Promise")
}).then(() => {
  console.log("resolve")
})
console.log("1")

```

## then

```js
# Promise是一个对象，Promise.then也是一个Promise对象（pending）
new Promise((resolve, reject) => {
  resolve('fulfilled')
})
.then(value => {
  # 如果不加return，下面的then是对此then的处理，默认是resolve
  # 加上return，下面的then是对return的Promise对象的处理
  # return一个值(字符串、类、对象)，下面的then接收的value就是此值
  return new Promise((resolve, reject) => {
    resolve('成功')
  })
})
.then(value => {
  console.log(value)
}, reason => {
  console.log(reason)
})
```

## catch

```js
# new Promise中可以直接抛出错误，then中可以接收抛出的错误，因为其内部自己会实现try catch捕获错误
new Promise((resovle, reject) => {
  // reject('fail')
  // throw new Error('faill')
  try {
    abc + 1
  } catch(error) {
    reject(error)
  }
})
  .then(value => {
  console.log(value)
})
  .catch(reason => {
  console.log(reason)   // 输出错误faill
})

# 使用catch一般放在最后面，这样是对上面所有的错误进行处理
# catch放在最后面，如果其中的某一个Promise有reject的处理，会执行对应的错误处理，不会执行catch，即catch之前Promise的错误有错误处理，则会执行其对应错误处理，不会执行catch

new Promise((resolve, reject) => {
  reject("fail")
})
  .then(value => {
  console.log(value)
})
  .catch(reason => {
  console.log(reason)  // 输出错误fail
})
# 或者
new Promise((resolve, reject) => {
		// reject("fail")
		resolve("success")
	})
	.then(value => {
		return new Promise((resolve, reject) => {
			reject("no")
		})
	})
	.catch(reason => {
		console.log(reason)  // 输出no
	})
```

## ajax

```js
function ajax(url) {
  return new Promise((resolve, reject) => {
    let xhr = new XMLHttpRequest();
    xhr.open("GET", url);
    xhr.send();
    xhr.onload = function() {
      if (this.status === 200) {
        resolve(JSON.parse(this.response))
      } else {
        reject("失败")
      }
    }
    xhr.onerror = function() {
      reject("跨域了呀")
    }
  })
}

ajax(url).then(value => {
  console.log(value)
}, reason => {
  console.log('请求失败')
})
```

## finally

```js
# 无论Promise成功或失败，finally始终会执行
new Promise((reslove, reject) => {
  reslove('fulfilled');
  // reject("fail");
})
  .then(value => {
  console.log(value);
})
  .catch(reason => {
  console.log(reason);
})
  .finally(() => {
  console.log('finally');
})
# 使用场景：可以在发送ajax请求时，请求完时，用finally将loading图隐藏
```

## Promise.resolve缓存数据

```js
function query() {
  const cache = query.cache || (query.cache = new Map());
  if (cache.has("name")) {
    console.log("走缓存");
    # 可以这样使用
    return Promise.resolve(cache.get("name"))
  }
  return ajax(url).then(value => {
    console.log("没走缓存");
    cache.set('name',value)
    return value;
  })
}

query().then(value => {
  console.log(value)
})

setTimeout(() => {
  query().then(value => {
    console.log(value)
  })
}, 2000)
```

## Promise.reject

```js
new Promise((resolve, reject) => {
  resolve('success')
})
.then(value => {
  console.log(value);
  # 在then中可以使用Promise.reject改变状态
  return Promise.reject('参数错误')
})
.catch(reason => {
  console.log(reason);  // 参数错误
})
```

## Promise.all

```js
# 必须都是resolve解决状态，才会返回结果
const first = new Promise((resolve, reject) => {
  setTimeout(() => {
    // resolve('first');
    reject("first fail");
  }, 1000)
})
const second = new Promise((resolve, reject) => {
  setTimeout(() => {
    // resolve('second');
    reject("second fail");
  }, 1000)
})
Promise.all([first, second]).then(value => {
  console.log(value);  // 都是resolve，返回["first", "second"]
}).catch(reason => {
  console.log(reason);   // first fail，返回第一个reject
})

# 如果传入可迭代对象为空时为同步
var p = Promise.all([]);
var p2 = Promise.all([1337, "hi"]);
console.log(p);  // Promise { <state>: "fulfilled", <value>: Array[0] }，state状态fullfilled已解决，主线程立即执行
console.log(p2); // Promise { <state>: "pending" }，不为空时，还是pending状态，还在微任务队列中
setTimeout(function(){
  console.log('the stack is now empty');
  console.log(p2); // Promise { <state>: "fulfilled", <value>: Array[2] }，p2从栈中取出执行，状态变为fulfilled已解决
});
```

## Promise.allSettled

```js
# 无论是resolve还是reject状态，都会返回对应的结果
const p1 = Promise.resolve(1);
const p2 = Promise.reject(2);

Promise.allSettled([p1, p2]).then(value => {
  console.log(value);  // 0: {status: "fulfilled", value: 1}, 1: {status: "rejected", reason: 2}
})
```

## Promise.any

```js
# 此方法依然是实验性的，尚未被所有的浏览器完全支持
# 与all相反，其中一个成功，就返回成功的promise
# 都失败，返回AggregateError: All promises were rejected
let p1 = Promise.resolve(1)
let p2 = Promise.reject(2)
Promise.any([p1,p2]).then(v=>console.log(v)).catch(_ => console.log(_)) // 1

# 都reject
let p1 = new Promise((resolve, reject) => {
  // resolve('succ')
  reject('fail1')
})
let p2 =  new Promise((resolve, reject) => {
  // resolve('succ')
  reject('fail2')
})
Promise.any([p1,p2]).then(v=>console.log(v)).catch(_ => console.log(_)) // AggregateError: All promises were rejected
```

## Promise.race

```js
# 返回处理最快的结果
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(1)
  }, 2000)
})
const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(2)
  }, 1000)
})
Promise.race([p1, p2]).then(value => {
  console.log(value);   // 2
})

# 使用场景：请求超时
function query(url, delay = 2000) {
  let promises = [
    ajax(url),
    new Promise((resolve, reject) => {
      setTimeout(() => {
        reject('请求超时')
      }, delay)
    })
  ]
  return Promise.race(promises);
}
query(url, 1000).then(value => {
  console.log(value)
}).catch(reason => {
  console.log(reason)
})
```

## async

```js
# async是generator和promise的语法糖
# 无论async函数有无await操作，总是返回一个Promise，因此后面可以直接调用then方法
# 函数内部返回的值，会成为then回调函数的参数
# 函数内部抛出的错误，会被catch捕获到
# async没有显示return，相当于return Promise.resolve()
# return非Promise数据的data，相当于return Promise.resolve(data)
# return Promise，会得到Promise本身

async function ryan() {}
console.log(ryan())  // Promise {<fulfilled>: undefined}

async function ljn() {
  return 'ljn'
}
ljn().then(v => console.log(v)) // ljn
```

## await

```js
# await是then的语法糖，只能在async函数中使用
async function pdd() {
  let wait = await "lijingnan"
  console.log(wait) // lijingnan
}
pdd()

async function study() {
  let name = await new Promise(resolve => {
    setTimeout(() => {
      resolve('lijingnan')
    }, 2000)
  })
  console.log(name)  // 2s后输出lijingnan
  let site = await new Promise(resolve => {
    setTimeout(() => {
      resolve('study')
    }, 2000)
  })
  console.log(site) // 4s后输出study
}

study()
```

## async & await & ajax

```js
# async返回一个Promise，所以此时get函数就是一个Promise，await相当于then，所以会先执行user，再执行age
async function get(name) {
  let user = await ajax(url1 + name)
  let age = await ajax(url2 + user.id)

  console.log(age)
}
get('lijingnan')
```

## async & await & catch

```js
async function study(name) {
  let lesson = await ajax(`url=${name}`)
  return lesson
}
study('lijingnan').then(v => {
  console.log(v)
}).catch(reason => {
  console.log(reason)
})
```

## class

```js
# 如果一个类中包装一个then方法，这个类就会被包装成Promise
class User{
  then(resolve, reject) {
    resolve()
  } 
}
async function get() {
  await new User()
  console.log(123)
}
get();

class User() {
  async get(name) {
    let user = await ajax('url');
    user.name += 'test';
    
    return user;
  }
}
new User().get('lijingnan').then(v => console.log(v))
```

## await并行执行

```js
function p1() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('p1')
    }, 2000)
  })
}
function p2() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('p2')
    }, 2000)
  })
}
async function study() {
  let s1 = p1()
  let s2 = p2()
  let res1 = await s1;  // await当then用
  let res2 = await s2;
  console.log(res1, res2)
}
// 或者
async function study() {
  let res = await Promise.all([p1(), p2()])
}
```

