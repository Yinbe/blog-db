# JS中的回调函数

回调函数是作为参数传递给另外一个函数，并且改回调函数在函数主题执行完后再执行

```
chrome.history.search({
      text: '',
      startTime: kOneWeekAgo,
      maxResults: 99
    }, function (items) {
  
    });
```

> 存在问题：因为JS是异步的，通过回调函数异步操作，会造成回调函数的嵌套，很影响代码的可读性

# 什么是promise

promise通常用于处理函数的异步调用，通过链式调用的方式

```
var mPromise=function(tag){

  return new Promise(function(resolve,reject){
    if(tag){
      resolve('默认参数success');
    }else{
      reject('默认参数filed');
    }
})
}

mPromise(true).then(function(message){
  console.log(message)
    //"输出默认参数success"
}).catch(function(ErrMsg){
    //获取数据失败时的处理逻辑
})
```

# async/await

先从async关键字说起，它被放置在一个函数前面

```
async function f() {
    return 1
} 

f().then(alert) // 1

```

函数前面的async一词意味着这个函数总是返回一个promise

```
let response = await f()
```

关键词await可以让JavaScript进行等待，直到一个promise执行并返回它的结果，JavaScript才会继续往下执行
await不能工作在顶级作用域，所以我们需要将await代码包裹在一个async函数中

我们可以使用try-catch语句捕获错误

```
async function f() {
    try {
        let response = await fetch('http://no-such-url')
    } catch (err) {
        alet(err) // TypeError: failed to fetch
    }
}

f()

```

如果我们不使用try-catch，然后async函数f()的调用产生的promise变成reject状态的话，我们可以添加.catch去处理它

```
async function f() {
    let response = await fetch('http://no-such-url')
}
// f()变成了一个rejected的promise

f().catch(alert) // TypeError: failed to fetch

```

# 异步处理封装

```
const asyncWorker = promise => {
  return promise.then(res => [null, res]).catch(err => [err])
}
```

```
const historyTabs = async () => {

  const [err, res] = await utils.asyncWorker(
    new Promise((resolve) =>
      chrome.history.search({
        text: '',
        maxResults: 0,
        startTime:oneWeekAgo,
      },items => resolve(items))
    )
  );
  if (err) {
    console.log(err);
  }
  
  console.log(items);
}
```
