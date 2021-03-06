# ES6灌溉垒土: Let 和 Const
[TOC]
`万丈高楼起于垒土` 
## let
### 只在代码块的作用域有效
ES6 常用的`let` 类似于`var`对变量的声明 
`var `是在全局作用域都有效
`let`是在代码块的作用域有效
```
{
    var a = 1
    let b = 2
}
a // 1
b // 报错
```
所以 `let`只在声明代码快的作用域有效
### 变量不存在提升
很简单 我们在使用`var`的时候
```
console.log(a)
var a= 2
a // undefined
```
虽然是undefined，但是也奇怪的 强迫症受不了吧
所以let 就不能这样了 你要提升？不存在了 
```
console.log(a) // 报错了
let a = 2
```
### 不允许重复声明
```
var a= 3
var a = 4
a // 4

let b = 1
let b = 2
b // 报错了
```
这代码清楚的很 其实就是不允许 在代码块同一个作用域内 重复声明而已
这里还有一种情况要注意踩坑 那就是 传参不能重复声明
```
function fun(arr) {
    let arr 
}
fun()// 报错了

function fun(arr) {
    {
        let arr = 2
    }
}
fun () // 不报错
```
所以说 别在同一个作用域内 声明一样的变量就行了 
其实这es6是更严谨了 一般来说 自己写的程序很少声明一样的变量 那就是找抽啊

### for循环很好使
```
for (var i = 0 ; i < 10; i ++) {
    setTimeout(function(){
        console.log(i)
      })
}
// 10
```
这里输出10次10 因为`i`是全局变量 每一次循环代码块都是对`i`进行操作
`setTimeout` 在for循环体内 每一次获得都是同一个`i` 循环结束后运行 也就是输出10次10了
我要输出0123456789怎么办 用var改成let就行了
```
 for (let i = 0 ; i < 10; i ++) {
    setTimeout(function(){
        console.log(i)
      })
}
// 0123456789
```
这里代码块工作 每一次循环`i` 都是在当前这次循环内有效 也可以理解成每一次都是一次 新的`i`
竟然每一次都是新的 那么在`setTimeout`代码块内 每一次都是不一样的`i` 也就是最后输出0123456789

我当时看到这里也有疑问 既然每一次都是新的`i` 为什么能记得上一个循环的是几？
这里看了看阮一峰大神的文档  有解释到
```
这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算。
```
反正很牛13就对了

![](./_image/2020-12-11-20-35-01.jpg)

### 我很霸道
let还有一个强制的约定 专业术语`暂时性死区`
看看下面代码
```
var a = 1
{
    console.log(a) // 报错
    let a = 1
}
```
如果按照正常来说 我定义的全局变量`a` 代码往下执行`console`出来的应该是`1`
但是这里就报错了 
ES6这里严格约定 , 只要我代码块内 有声明`let`或者`const` 命令 那么这个代码块就马上给我封闭起来
只要你我声明之前 使用这些变量 我就给你报错
这段语法上 也叫`暂时性死区`
其实这也是属于严谨点 也是好的 我们编写代码 都是一般 先声明在调用 能不使用全局变量 就不使用!
(你特别排除外)





## 块级作用域
js一直都只有全局作用域 和 函数作用域
这样带来很多 麻烦的事情
比如for声明的i 其实在全区作用域一直存活
内层的变量 导致被外层的变量覆盖
其实`let` 和 `const` 也间接的成就了块级作用域
```
function a {
    b = 1
    {
        b = 2
    }
    console.log (b) // 1

}
```
我们常用的匿名函数 就是伪块级作用域
```
(function(){
 var a = 1 
 }())
 
 直接被取代成
 {
     let a = 1
 }
```
es6 确实使用的越来越轻松

## 块级作用域内使用函数
在es5规范汇总 函数都是用在最顶层 或者 函数内
如果要在块级作用域内 使用函数
要尽量使用 函数表达式
```
// 块级作用域内部，优先使用函数表达式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```
## const
cosnt声明的是一个可读变量 也就是说 只能看 不能更改
```
const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动。对于简单类型的数据（数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。
但对于复合类型的数据（主要是对象和数组），变量指向的内存地址，保存的只是一个指向实际数据的指针，const只能保证这个指针是固定的（即总是指向另一个固定的地址），至于它指向的数据结构是不是可变的，就完全不能控制了
```
这段话炒自阮一峰的文档 对于const的说明 很有意思 
```
const a = 1

const b = {}
b.age = 26
b.age // 26

b = {} // 报错
```
这栗子结合上面那段话食用就很有意思了  也就是说 
我const 指向你内存的这个位置 你是一个对象`{}`  但是你这个对象的数据结构 你要怎么更改我不管 
但是你这个变量`{}` 一旦更改 脱离了我原本指向的内存地址 那么很抱歉 我给你报错

数组也是一样
```
var a= []
a.push(1)
a[0] // 0
a.length // 1

a = [1,2,3] // 报错
```
反正就是说 const定义的变量 我指向的这个内存地址 你不能给我更改! 但是这个内存的结构 我不管你 
有意思
为了让const的原则贯彻到底 我就是不给更改
可以用`Object.freeze` 来把对象冻结
```
const a = Object.freeze({})
a.b = 1 // 没有生效
```
感觉有点多此一举了 直接用let不就完事了

let和const的读看心得到此 let是es6的一大亮点 能很方便解决很多es5需要绕一圈的问题
