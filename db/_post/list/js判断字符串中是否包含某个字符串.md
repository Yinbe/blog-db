### ES6：

- includes()：是否包含了参数字符串，返回布尔值
- startsWith()：参数字符串是否在原字符串的头部，返回布尔值
- endsWith()：参数字符串是否在原字符串的尾部，返回布尔值

```js
const homeUrl = 'https://qszone.com'
homeUrl.includes('qszone')  //true
homeUrl.startsWith('https') //true
homeUrl.endsWith('cn') //false
```

注：这三个方法都支持第二个参数，表示开始匹配的位置

### ES5:

1. indexOf()：返回某个指定的字符串值在字符串中首次出现的位置，如果检索的字符串没有出现，则返回-1，此方法同样适用于数组

```js
var homeUrl = 'https://qszone.com'
homeUrl.indexOf('qszone') !== -1 //true，表示包含

var arr = ['https','qszone','com']
arr.indexOf('http')  //-1
arr.indexOf('qszone') //1
```

2. search()：跟indexOf方法类似，参数可以是正则表达式，不过不适用数组，不推荐使用
3. 正则方法：match()、test()、exec()

```js
var homeUrl = 'https://qszone.com'
var reg = RegExp(/1024/)
homeUrl.match(reg))  //null,返回匹配的对象，不匹配则返回null

reg.test(homeUrl)  //false,返回bool值

reg = RegExp(/qszone/)
reg.exec(homeUrl)  //["qszone", index: 8, input: "https://qszone.com", groups: undefined],不匹配则返回null
```
