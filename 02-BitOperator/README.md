## 位运算符

位运算符是一切二进制数据操作的基础，在`js`里总共只有`7`个符号，看似容易但要熟练起来需要一定的练习。

W3school 这篇文章讲得很清晰: [ECMAScript 位运算符](https://www.w3school.com.cn/js/pro_js_operators_bitwise.asp)

简单总结下，所有数据都可以用字节表示，一个字节就是`1Byte`，`1024Byte`就是`1KB`这个大家都懂的，但它需要转换成 `0101...`这种机器能读得多的符号为二进制代码，其中`1Byte`可以转换成`8`个这个二进制代码，例如数字`1`可以转化为 `00000001`，数字`3`可以转化为`00000010`这个也很好理解。

那么，`1Byte`最大能表示多大的数字呢？那就是二进制代码的`11111111`啦，转换成`10`进制就是`255`。

```js
parseInt("11111111", 2) === 255;
```

但有时候觉得用`11111111`去表示一个数字，对机器来说是没问题的，但对人的阅读来说可能不是很方便，所以就有了除`10`进制以外的进制表示方法，例如常用的`16`进制，那么`11111111`可以转换成`16`进制的`FF`;

```js
(255).toString(16) === "ff";
```

通常你说的`ff`鬼知道它是`16`进制还是`32`进制啊，所有我们约定在数字前面加上进制表示的符号，例如：

```js
// 16进制前面带有0x, 2进制前面带有0b
0xff === 0b11111111;
```

题外话，常见的颜色值可以用`6`位的`16`进制表示，例如白色为`#ffffff`，黑色为`#000000`，请问这个区间可以表示多少种颜色值？

```js
parseInt("ffffff", 16) === 16777215;
```

大致理解后，再阅读 W3school 的文章就容易很多了，那么就来练习一个简单的场景：

获得有一个字节的流为`00011010`，其中第`4`位到第`7`位(`1101`)表示了一个无符号整数，我怎么知道这个数是多少？

```js
((0b00011010 >> 1) & 0b00001111) === 13;
// 或者
((0b00011010 >> 1) & 15) === 13;
```

又或者，现在有两个字节`00011010`和`11000000`，第一个字节的低位 4 个比特(`1010`)和第二个字节的高位 2 个比特(`11`)表示了一个无符号整数，我怎么知道这个数是多少？

```js
(((0b00011010 & 0b00001111) << 2) | (0b11000000 >> 6)) === 43;
// 或者
(((0b00011010 & 15) << 2) | (0b11000000 >> 6)) === 43;
```

可以看出位运算是内部以二进制进行运算，但都是以`10`进制输出的。

### 字节序

字节序是初学者不容易理解的概念，甚至在操作时会产生疑惑，但实际上并不用太关心，因为一般一种数据就只有一种字节序。

可以看看阮一峰的[理解字节序](http://www.ruanyifeng.com/blog/2016/11/byte-order.html)

简单总结下，就是单次写入多个字节的时候，这些字节是怎么排列的，有时候是从大到小是我们直观的表示方式，有时候却是从小到大排。

例如我有两个字节分别是数字`20`和`19`，假如把它看成一个年份，我们最直观的阅读方式就是`2019`啦，这就是大端字节序。但假如要用小端字节序的话就是反过来`1920`，所以字节序没弄清的话会造成很大的差别。

那么怎么判断当前环境是哪个字节序呢？这其实很简单，但关于相关语法后面再讲解。

```js
// 新建一个多字节的缓冲，用 Uint16Array 和 Uint32Array 都可以
var a = new Uint32Array([0x12345678]);

// 再以Uint8Array查看排列顺序
// 假如是大端字节序就会是：[12, 34, 56, 78]
// 假如是小端字节序就会是：[78, 56, 34, 12]
var b = new Uint8Array(a.buffer);
var BigEndian = b[0] == 0x12;
```

### 数字和 Unicode 互转

> 关于 ASCII，Unicode 和 UTF-8 可以网上查一下，了解一下

实际场景中，二进制数据不单单只返回数字，其中还可以返回字符串，这里简单说下数字和`Unicode`字符串的互转，以后肯定会用到。

```js
// 数字转Unicode
String.fromCharCode.call(String, 20013) === "中";

// Unicode转数字
String.prototype.charCodeAt.call("中", 0) === 20013;
// 或者
"中".charCodeAt(0) === 20013;
```

### Base64

`Base64`在数据保存和传输也经常用到，有了以上基础，再看`Base64`原理时就容易得多了。

参考了阮一峰的[Base64 笔记](http://www.ruanyifeng.com/blog/2008/06/base64.html)

> 所谓 Base64，就是说选出 64 个字符----小写字母 a-z、大写字母 A-Z、数字 0-9、符号"+"、"/"（再加上作为垫字的"="，实际上是 65 个字符）----作为一个基本字符集。然后，其他所有符号都转换成这个字符集中的字符。

1. 将每三个字节作为一组，一共是 `24` 个二进制位。
2. 将这 `24` 个二进制位分为四组，每个组有 `6` 个二进制位。
3. 在每组前面加两个 `00`，扩展成 `32` 个二进制位，即四个字节。
4. 根据下表，得到扩展后的每个字节的对应符号，这就是 `Base64` 的编码值。

`因为，Base64` 将三个字节转化成四个字节，因此 `Base64` 编码后的文本，会比原文本大出三分之一左右。

要注意一点，常见的`Base64`字符串前面都有数据类型和编码方式，例如`data:image/png;base64,`，这部分是给服务器或者浏览器识别的，并不是`Base64`的一部分，编码时要区分出来。

浏览器自带的`Base64`编码和反编码方法：

```js
btoa("ABC") === "QUJD";
atob("QUJD") === "ABC";
```

Nodejs 自带的`Base64`编码和反编码方法：

```js
Buffer.from("ABC").toString("base64") === "QUJD";
Buffer.from("QUJD", "base64").toString("ascii") === "ABC";
```
