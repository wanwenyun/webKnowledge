<!-- TOC -->

- [ES6](#es6)
    - [var、let 及 const 区别](#varlet-%E5%8F%8A-const-%E5%8C%BA%E5%88%AB)
    - [Proxy](#proxy)
    - [map](#map)
    - [filter](#filter)
    - [reduce](#reduce)
    - [Es6 中箭头函数与普通函数的区别](#es6-%E4%B8%AD%E7%AE%AD%E5%A4%B4%E5%87%BD%E6%95%B0%E4%B8%8E%E6%99%AE%E9%80%9A%E5%87%BD%E6%95%B0%E7%9A%84%E5%8C%BA%E5%88%AB)
    - [Promise](#promise)
    - [async 和 await](#async-%E5%92%8C-await)
    - [Generator 生成器](#generator-%E7%94%9F%E6%88%90%E5%99%A8)
    - [生成器原理](#%E7%94%9F%E6%88%90%E5%99%A8%E5%8E%9F%E7%90%86)
    - [ES Module](#es-module)
  - [手写 Promise](#%E6%89%8B%E5%86%99-promise)
    - [简易版 Promise](#%E7%AE%80%E6%98%93%E7%89%88-promise)

<!-- /TOC -->

# ES6

### var、let 及 const 区别

- var 声明的变量会挂载在 window 上，而 let 和 const 不会
- var 声明变量存在变量提升，let 和 const 不会
- let、const 的作用范围是块级作用域，而 var 的作用范围是函数作用域
- 同一作用域下 let 和 const 不能声明同名变量，而 var 可以
- 同一作用域下在 let 和 const 声明前使用会存在暂时性死区
- const
  - 一旦声明必须赋值，不能使用 null 占位
  - 声明后不能再修改
  - 如果声明的是复合类型数据，可以修改其属性

### Proxy

Proxy 是 ES6 中新增的功能，它可以用来自定义对象中的操作。 Vue3.0 中将会通过 Proxy 来替换原本的 Object.defineProperty 来实现数据响应式。

```js
let p = new Proxy(target, handler)
```

`target` 代表需要添加代理的对象，`handler` 用来自定义对象中的操作，比如可以用来自定义 set 或者 get 函数。

```js
let onWatch = (obj, setBind, getLogger) => {
  let handler = {
    set(target, property, value, receiver) {
      setBind(value, property)
      return Reflect.set(target, property, value)
    },
    get(target, property, receiver) {
      getLogger(target, property)
      return Reflect.get(target, property, receiver)
    }
  }
  return new Proxy(obj, handler)
}

let obj = { a: 1 }
let p = onWatch(
  obj,
  (v, property) => {
    console.log(`监听到属性 ${property}改变为 ${v}`)
  },
  (target, property) => {
    console.log(`'${property}' = ${target[property]}`)
  }
)
p.a = 2 // 控制台输出：监听到属性 a 改变
p.a // 'a' = 2
```

自定义 set 和 get 函数的方式，在原本的逻辑中插入了我们的函数逻辑，实现了在对对象任何属性进行读写时发出通知。

当然这是简单版的响应式实现，如果需要实现一个 Vue 中的响应式，需要我们在 get 中收集依赖，在 set 派发更新，之所以 Vue3.0 要使用 Proxy 替换原本的 API 原因在于 Proxy 无需一层层递归为每个属性添加代理，一次即可完成以上操作，性能上更好，并且原本的实现有一些数据更新不能监听到，但是 Proxy 可以完美监听到任何方式的数据改变，唯一缺陷可能就是浏览器的兼容性不好了。

### map

map 作用是生成一个新数组，遍历原数组，将每个元素拿出来做一些变换然后返回一个新数组，原数组不发生改变。

map 的回调函数接受三个参数，分别是当前索引元素，索引，原数组

```js
var arr = [1, 2, 3]
var arr2 = arr.map(item => item + 1)
arr //[ 1, 2, 3 ]
arr2 // [ 2, 3, 4 ]
```

```js
;['1', '2', '3'].map(parseInt)
// -> [ 1, NaN, NaN ]
```

- 第一个 parseInt('1', 0) -> 1
- 第二个 parseInt('2', 1) -> NaN
- 第三个 parseInt('3', 2) -> NaN

### filter

filter 的作用也是生成一个新数组，在遍历数组的时候将返回值为 true 的元素放入新数组，我们可以利用这个函数删除一些不需要的元素

filter 的回调函数接受三个参数，分别是当前索引元素，索引，原数组

### reduce

reduce 可以将数组中的元素通过回调函数最终转换为一个值。
如果我们想实现一个功能将函数里的元素全部相加得到一个值，可能会这样写代码

```js
const arr = [1, 2, 3]
let total = 0
for (let i = 0; i < arr.length; i++) {
  total += arr[i]
}
console.log(total) //6
```

但是如果我们使用 reduce 的话就可以将遍历部分的代码优化为一行代码

```js
const arr = [1, 2, 3]
const sum = arr.reduce((acc, current) => acc + current, 0)
console.log(sum)
```

对于 reduce 来说，它接受两个参数，分别是回调函数和初始值，接下来我们来分解上述代码中 reduce 的过程

- 首先初始值为 0，该值会在执行第一次回调函数时作为第一个参数传入
- 回调函数接受四个参数，分别为累计值、当前元素、当前索引、原数组，后三者想必大家都可以明白作用，这里着重分析第一个参数
- 在一次执行回调函数时，当前值和初始值相加得出结果 1，该结果会在第二次执行回调函数时当做第一个参数传入
- 所以在第二次执行回调函数时，相加的值就分别是 1 和 2，以此类推，循环结束后得到结果 6。

### Es6 中箭头函数与普通函数的区别

- 普通 function 的声明在变量提升中是最高的，箭头函数没有函数提升
- 箭头函数的this对象是定义时所在的对象，而不是使用时
- 箭头函数不能使用yield,箭头函数不可作为Generator函数
- 箭头函数没有属于自己的 `this`，`arguments`，`super`、`new.target`
- 箭头函数不能作为构造函数，不能被 new，没有 property
- call 和 apply 方法只有参数，没有作用域

### Promise

`Promise` 翻译过来就是承诺的意思，这个承诺会在未来有一个确切的答复，并且该承诺有三种状态，这个承诺一旦从等待状态变成为其他状态就永远不能更改状态了。

- 等待中（pending）
- 完成了（resolved）
- 拒绝了（rejected）

当我们在构造 Promise 的时候，构造函数内部的代码是立即执行的。

```js
new Promise((resolve, reject) => {
  console.log('new Promise')
  resolve('success')
})
console.log('finifsh')

// 先打印 new Promise， 再打印 finifsh
```

Promise 实现了链式调用，也就是说每次调用 then 之后返回的都是一个 Promise，并且是一个全新的 Promise，原因也是因为状态不可变。如果你在 then 中 使用了 return，那么 return 的值会被 Promise.resolve() 包装。

```js
Promise.resolve(1)
  .then(res => {
    console.log(res) // => 1
    return 2 // 包装成 Promise.resolve(2)
  })
  .then(res => {
    console.log(res) // => 2
  })
```

当然了，Promise 也很好地解决了回调地狱的问题

```js
ajax(url)
  .then(res => {
    console.log(res)
    return ajax(url1)
  })
  .then(res => {
    console.log(res)
    return ajax(url2)
  })
  .then(res => console.log(res))
```

其实它也是存在一些缺点的，比如无法取消 Promise，错误需要通过回调函数捕获。

### promise 必知必会



1. promise 中 `return new Error('error')`不会抛出错误， 因此不会被`catch`接住

```js

Promise.resolve()
  .then(() => {
    return new Error('error!!!')
  })
  .then((res) => {
    console.log('then: ', res)
  })
  .catch((err) => {
    console.log('catch: ', err)
  })
// 输出
then: Error: error!!!
    at Promise.resolve.then (...)
    at ...
```

要能被catch 需要修改成下面两种：

```js
return Promise.reject(new Error('error'))

throw new Error('error!!')
```

因为返回任意一个值都会被包装成promise对象，即 `return new Error('error!!!')` 等价于 `return Promise.resolve(new Error('error!!!'))`。

2、promise循环引用会报错

```js
const promise = Promise.resolve()
  .then(() => {
    return promise
  })
promise.catch(console.error)
```

![1568028277611](../img/1568028277611.png)

3、`.then` 或者 `.catch` 的参数期望是函数，传入非函数则会发生值穿透

```js
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
  
  // 1
```

4、`.then` 可以接收两个参数，第一个是处理成功的函数，第二个是处理错误的函数。`.catch` 是 `.then` 第二个参数的简便写法，但是它们用法上有一点需要注意：`.then` 的第二个处理错误的函数捕获不了第一个处理成功的函数抛出的错误，而后续的 `.catch` 可以捕获之前的错误

```js
Promise.resolve()
  .then(function success (res) {
    throw new Error('error')
  }, function fail1 (e) {
    console.error('fail1: ', e)
  })
  .catch(function fail2 (e) {
    console.error('fail2: ', e)
  })
  
  fail2: Error: error
    at success (...)
    at ...

```

5、`process.nextTick` 和 `promise.then` 都属于 microtask，而 `setImmediate` 属于 macrotask，在事件循环的 check 阶段执行。事件循环的每个阶段（macrotask）之间都会执行 microtask，事件循环的开始会先执行一次 microtask。

```
process.nextTick(() => {
  console.log('nextTick')
})
Promise.resolve()
  .then(() => {
    console.log('then')
  })
setImmediate(() => {
  console.log('setImmediate')
})
console.log('end')

// end
// nextTick
// then
// setImmediate
```

5、promise 构造函数是同步执行，`promise.then` 中的函数是异步执行的。

```js
const promise = new Promise((resolve, reject) => {
  console.log(1)
  resolve()
  console.log(2)
})
promise.then(() => {
  console.log(3)
})
console.log(4)
1 2 4 3
```

6、promise 有 3 种状态：pending、fulfilled 或 rejected。状态改变只能是 pending->fulfilled 或者 pending->rejected，状态一旦改变则不能再变。构造函数中的 resolve 或 reject 只有第一次执行有效，多次调用没有任何作用，

```js
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success')
  }, 1000)
})
const promise2 = promise1.then(() => {
  throw new Error('error!!!')
})

console.log('promise1', promise1)
console.log('promise2', promise2)

setTimeout(() => {
  console.log('promise1', promise1)
  console.log('promise2', promise2)
}, 2000)

promise1 Promise { <pending> }
promise2 Promise { <pending> }
(node:50928) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: error!!!
(node:50928) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
promise1 Promise { 'success' }
promise2 Promise {
  <rejected> Error: error!!!
    at promise.then (...)
    at <anonymous> }
```





### async 和 await

一个函数如果加上 async ，那么该函数就会返回一个 Promise

```js
async function test() {
  return '1'
}
console.log(test())
// -> Promise {<resolved>: "1"}
```

async 就是将函数返回值使用 Promise.resolve() 包裹了下，和 then 中处理返回值一样，并且 await 只能配套 async 使用。

```js
async function test() {
  let value = await sleep()
}
```

async 和 await 可以说是异步终极解决方案了，相比直接使用 Promise 来说，优势在于处理 then 的调用链，能够更清晰准确的写出代码，毕竟写一大堆 then 也很恶心，并且也能优雅地解决回调地狱问题。

当然也存在一些缺点，因为 **await 将异步代码改造成了同步代码**，如果多个异步代码没有依赖性却使用了 await 会导致性能上的降低。

```js
async function test() {
  // 以下代码没有依赖性的话，完全可以使用 Promise.all 的方式
  // 如果有依赖性的话，其实就是解决回调地狱的例子了
  await fetch(url)
  await fetch(url1)
  await fetch(url2)
}
```

看一个使用 await 的例子：

```js
let a = 0
let b = async () => {
  a = a + (await 10)
  console.log('2', a)
}
b()
a++
console.log('1', a)

// 先输出  ‘1’, 1
// 在输出  ‘2’, 10
```

- 首先函数 b 先执行，在执行到 await 10 之前变量 a 还是 0，因为 await 内部实现了 generator ，generator 会保留堆栈中东西，所以这时候 a = 0 被保存了下来
- 因为 await 是异步操作，后来的表达式不返回 Promise 的话，就会包装成 Promise.reslove（返回值)，然后会去执行函数外的同步代码
- 同步代码 a++ 与打印 a 执行完毕后开始执行异步代码，将保存下来的值拿出来使用，这时候 a = 0 + 10

上述解释中提到了 await 内部实现了 generator，其实 **await 就是 generator 加上 Promise 的语法糖，且内部实现了自动执行 generator**。

### Generator 生成器

```js
function* foo(x) {
  let y = 2 * (yield x + 1)
  let z = yield y / 3
  return x + y + z
}
let it = foo(5)
console.log(it.next()) // => {value: 6, done: false}
console.log(it.next(12)) // => {value: 8, done: false}
console.log(it.next(13)) // => {value: 42, done: true}
```

- 首先 Generator 函数调用和普通函数不同，它会返回一个迭代器

- 当执行第一次 next 时，传参会被忽略，并且函数暂停在 yield (x + 1) 处，所以返回 5 + 1 = 6

- 当执行第二次 next 时，传入的参数等于上一个 yield 的返回值，如果你不传参，yield 永远返回 undefined。此时 let y = 2 _ 12，所以第二个 yield 等于 2 _ 12 / 3 = 8

- 当执行第三次 next 时，传入的参数会传递给 z，所以 z = 13, x = 5, y = 24，相加等于 42

### 生成器原理

当 yeild 产生一个值后，生成器的执行上下文就会从栈中弹出。但由于迭代器一直保持着队执行上下文的引用，上下文不会丢失，不会像普通函数一样执行完后上下文就被销毁

### ES Module

ES Module 是原生实现的模块化方案，与 CommonJS 有以下几个区别

- CommonJS 支持动态导入，也就是 require(\${path}/xx.js)，后者目前不支持，但是已有提案
- CommonJS 是同步导入，因为用于服务端，文件都在本地，同步导入即使卡住主线程影响也不大。而后者是异步导入，因为用于浏览器，需要下载文件，如果也采用同步导入会对渲染有很大影响
- CommonJS 在导出时都是值拷贝，就算导出的值变了，导入的值也不会改变，所以如果想更新值，必须重新导入一次。但是 ES Module 采用实时绑定的方式，导入导出的值都指向同一个内存地址，所以导入值会跟随导出值变化
- ES Module 会编译成 require/exports 来执行的

ES6 模块的设计思想是尽量的静态化，使得编译的时候就可以确定依赖关系，以及输入和输出的变量，
CommonJS 和 AMD 模块都只能在运行时确定这些东西

```js
// 引入模块 API
import XXX from './a.js'
import { XXX } from './a.js'
// 导出模块 API
export function a() {}
export default function() {}
```

### [ES5 / ES6 的继承除了写法以外，还有什么区别？](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/20https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/20)

ES5 先创造了子类的实例对象 this，然后再将父类的方法添加到this上， ES6的继承是先将父类实例对象的属性和方法添加到this上（constructor中必须先调用super方法）然后再用子类的构造函数修改this







## 手写 Promise

先看看 Promise 是怎么用的

```js
new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve(1)
  }, 0)
}).then(value => {
  console.log(value)
})
```

在 Promise 的构造器中传入一个函数，这个函数有两个参数 resolve 和 reject，这两个参数都是 Promise 的回调函数，不需要自己写，在需要的时候调用就可以了，他们分别是成功的回调 resolve 和失败的回调 reject。

### 简易版 Promise

**第一步**，先来搭建构建函数的大体框架

```js
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

function MyPromise(fn) {
  const that = this
  that.state = PENDING
  that.value = null
  that.resolvedCallbacks = []
  that.rejectedCallbacks = []
  // 待完善 resolve 和 reject 函数
  // 待完善执行 fn 函数
}
```

- 首先创建三个常量用于表示状态，对于经常使用的一些值应该通过常量来管理，便于开发及后期维护
- 在函数体内部首先创建常量 that，因为代码可能会异步执行，用于获取正确的 this 对象
- 一开始 Promise 的状态是 pending
- value 变量用于保存 resolve 或者 reject 中传入的值
- resolvedCallbacks 和 rejectedCallback 用于保存 then 中的回调，因为当执行完 Promise 时状态可能还是等待中，这时候应该把 then 中的回调保存起来用于状态改变时使用

**第二步**，完善 resolve 和 reject 函数，添加在 MyPromise 函数体内部

```js
function resolve(value) {
  if (that.state === PENDING) {
    that.state = RESOLVED
    that.value = value
    that.resolvedCallbacks.map(cb => cb(that.value))
  }
}

function reject(value) {
  if (that.state === PENDING) {
    that.state = REJECTED
    that.value = value
    that.rejectedCallbacks.map(cb => cb(that.value))
  }
}
```

- 首先两个函数都得判断当前状态是否为等待中，因为规范规定只有等待态才可以改变状态
- 将当前状态更改为对应状态，并且将传入的值赋值给 value
- 遍历回调数组并执行

**第四步**，实现如何执行 Promise 中传入的函数了

```js
try {
  fn(resolve, reject)
} catch (e) {
  reject(e)
}
```

- 实现很简单，执行传入的参数并且将之前两个函数当做参数传进去
- 要注意的是，可能执行函数过程中会遇到错误，需要捕获错误并且执行 reject 函数

**第五步**，实现较为复杂的 then 函数。

then 函数是在 Promise 构造器中成功状态下调用的 resolve 方法的回调。

then 函数是可以接收两个参数的，一个是用户自定义的成功处理，另一个是用户自定义的错误处理，第二个参数可不传。

```js
MyPromise.prototype.then = function(onFulfilled, onRejected) {
  const that = this
  // 对传入的两个参数做判断，如果不是函数将其转为函数
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v // onFulfilled = v => v
  onRejected =
    typeof onRejected === 'function'
      ? onRejected
      : r => {
          throw r
        }

  if (that.state === PENDING) {
    that.resolvedCallbacks.push(onFulfilled)
    that.rejectedCallbacks.push(onRejected)
  } else if (that.state === RESOLVED) {
    onFulfilled(that.value)
  } else {
    onRejected(that.value)
  }
}
```

- 首先判断两个参数是否为函数类型，因为这两个参数是可选参数
- 当参数不是函数类型时，需要创建一个函数赋值给对应的参数，同时也实现了透传，比如如下代码

```js
// 该代码目前在简单版中会报错
// 只是作为一个透传的例子
Promise.resolve(4)
  .then()
  .then(value => console.log(value))
```

- 接下来就是一系列判断状态的逻辑，当状态不是等待态时，就去执行相对应的函数。如果状态是等待态的话，就往回调函数中 push 函数，比如如下代码就会进入等待态的逻辑
