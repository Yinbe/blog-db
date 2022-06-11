> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [studygolang.com](https://studygolang.com/articles/30161)

> iota 是一个古希腊字母. 在 golang 中表示常量计数器. 使用的规则如下: 每当 const 出现时, 都会使 iota 初始化为 0.const 中每新增一行常量声明将使 iota 计数一次. 我们再来看看示例代码: co......

`iota`是一个[古希腊字母](https://en.wiktionary.org/wiki/iota). 在 `golang`中表示常量计数器.

使用的规则如下:

1. 每当 `const`出现时, 都会使 `iota`初始化为 0.
2. `const`中每新增一行常量声明将使 `iota`计数一次.

我们再来看看示例代码:

```
const a0 = iota // a0 = 0  // const出现, iota初始化为0

const (
    a1 = iota   // a1 = 0   // 又一个const出现, iota初始化为0
    a2 = iota   // a1 = 1   // const新增一行, iota 加1
    a3 = 6      // a3 = 6   // 自定义一个常量
    a4          // a4 = 6   // 不赋值就和上一行相同
    a5 = iota   // a5 = 4   // 第5行数组位置为4, 所以这里是4
)
```
