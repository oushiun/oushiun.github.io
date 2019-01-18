---
title: 模板字符串

toc: true
tags:
 - ES6
categories:
 - 前端
 - JavaScript
 - ECMAScript 6
date: 2018-06-14 10:38:35
thumbnail: https://static.oushiun.com/blog/banner/es6.jpeg
---

### 语法

模板字符串(_Template String_)是增强版的字符串，用反引号(`)标识，它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

<!-- more -->

### 用法

``` javascript
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
not legal.`

// 字符串中嵌入变量
var name = "Bob", 
    time = "today"
`Hello ${name}, how are you ${time}?`   // Hello Bob, how are you today?
```

上面代码中，模板字符串都是用反引号表示，如果在模板字符串中需要使用反引号，则前面需要用反斜杠转义。

``` javascript
var greeting = `\`Yo\` World!` // `Yo` World!
```

如果使用模板字符串表示多行字符串，则所有的空格、缩进和换行都会被保留在输出中。

``` javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`)
```

上面代码中，所有模板字符串的空格和换行都是被保留的，比如 `<ul>` 标签前面会有一个换行。如果想把行首和行尾的换行、空格等去掉，则使用 `trim` 方法即可。

``` javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim())
```

模板字符串中嵌入变量，要将变量名写在 `${}` 之中。大括号内可以放入任意的 JavaScript 表达式，可以进行运算，以及引入对象属性。

``` javascript
var x = 1,
    y = 2

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

var obj = { x: 1, y: 2 }
`${obj.x + obj.y}`
// "3"

```

模板字符串之中还可以调用函数。

``` javascript
function func() {
    return 'Hello'
}

`${func()} World`
// "Hello World"
```

如果大括号中的值不是字符串，则将按照一般的规则转换为字符串。如，若大括号中是一个对象，则将默认调用对象的 toString 方法，把对象转换为字符串。
如果模板字符串中的变量没有声明，则会报错。

``` javascript
// 变量place没有声明
var msg = `Hello, ${place}`
// ReferenceError: place is not defined
```

模板字符串之间还可以进行嵌套。

``` javascript
var tmpl = addrs => `
    <table>
    ${addrs.map(addr => `
        <tr><td>${addr.first}</td></tr>
        <tr><td>${addr.last}</td></tr>
    `).join('')}
    </table>
`

tmpl([{ first: 'a', last: 'b' }])

// output:
/*"
    <table>

        <tr><td>a</td></tr>
        <tr><td>b</td></tr>

    </table>
"*/
```

如果需要引用模板字符串本身，在需要时执行，可以像下面这样写。

``` javascript
// 写法一
var str = 'return ' + '`Hello ${name}!`'
var func = new Function('name', str)
func('Amy') // "Hello Amy!"

// 写法二
var str = '(name) => `Hello ${name}!`'
var func = eval.call(null, str)
func('Amy') // "Hello Amy!"
```

### 标签模板

模板字符串的功能，不仅是上面那些，它还可以紧跟在一个函数后面，该函数将被调用来处理这个模板字符串，这种称为“标签模板”功能(_Tagged template_)。

标签模板函数第一个参数是字符串模板的常量数组，后面的每一个参数为表达式的计算结果，函数名称可以任意指定。下面是一个例子：

``` javascript
var a = 5,
    b = 10

function tag(strings, ...values) {
    console.log(strings[0]) // "Hello "
    console.log(strings[1]) // " world"
    console.log(strings[2]) // ""
    console.log(values[0]) // 15
    console.log(values[1]) // 50

    return 'Anything'
}

tag`Hello ${a + b} world ${a * b}`
// Anything
```

``` javascript
alert`123`

// 等同于
alert(123)
```

标签模板其它是一种特殊的函数调用形式，“标签”指的就是函数，紧跟在后面的模板字符串就是它的参数。

``` javascript
var a = 1,
    b = 2

tag`Helo ${a + b} world ${a * b}`
```

上面代码中，模板字符串前面有一个标识名 tag，它是一个函数。整个表达式的返回值就是 tag 函数处理模板字符串后的返回值。

函数 tag 依次会接收到多个参数。

``` javascript
function tag(stringArr, value1, value2) {
    // ...
}

// 等同于

function tag(stringArr, ...values) {
    // ...
}
```

tag 函数的第一个参数是一个数组，该数组的成员是模板字符串中那些没有变量替换的部分，也就是说，变量替换只发生在数组的第一个成员与第二个成员之间、第二个成员与第三个成员之间，以此类推。

tag 函数的其他参数，都是模板字符串各个变量被替换的值。本例中，模板字符串含有两个变量，因此 tag 会接收到 value1 和 value2 两个参数。

tag 函数所有参数的实际值如下：

-   第一个参数： [‘Hello ‘, ’ world’, ”]
-   第二个参数: 3
-   第三个参数: 2

也就是说，tag 函数实际上是用下面的形式调用：

``` javascript
tag(['Hello ', ' world', ''], 3, 2)
```

### String 对象的 raw 方法

String.raw 方法用来充当模板字符串的处理函数，返回一个除表达式和变量会被替换，其它都保持原样的字符串。

``` javascript
String.raw`Hi\n${2 + 3}!`
// "Hi\n5!"

String.raw`Hi\u000A!`
// "Hi\u000A!"

String.rwa`Hi\\n`
// "Hi\\n"
```

String.raw 方法可以作为处理模板字符串的基本方法，它会将所有变量替换，而且对斜杠进行转义，方便下一步作为字符串来使用。
