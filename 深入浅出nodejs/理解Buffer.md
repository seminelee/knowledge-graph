# 理解Buffer

js对字符串的操作十分友好，无论是宽子节字符串还是单子节字符串，都被认为是一个字符串。

除了字符串，Node还要处理网络协议、操作数据库、处理图片、接收上传文件等，在网络流和文件的操作中，还要处理大量二进制数据，这就需要`Buffer`对象了。

## 1 Buffer结构

### 1.1 模块结构

`Buffer`是一个典型的JavaScript与C++结合的模块，它将性能相关部分用C++实现（C++内建模块`node_buffer`），将非性能相关的部分用JavaScript实现（JavaScript核心模块Buffer/SlowBuffer）。Node在进程启动时就已经加载了`Buffer`，并将其放在全局对象（global），所以无须`require()`就可以直接使用`Buffer`。

### 1.2 Buffer对象

`Buffer`对象类似于数组，有`length`属性，可以通过下标访问元素。它的元素是16进制的两位数（0～255），中文字在UTF-8编码下占用3个元素，英文字符和半角标点符号占用1个元素。

```js
var str = '深入浅出node.js'
var buf = Buffer.from(str) // <Buffer e6 b7 b1 e5 85 a5 e6 b5 85 e5 87 ba 6e 6f 64 65 2e 6a 73>
buf[10] // 135
```

声明API：

- [`Buffer.from(array)`](http://nodejs.cn/s/F5r61t) 返回一个新的 `Buffer`，其中包含提供的八位字节数组的副本。`Buffer.from(arrayBuffer[, byteOffset [, length\]])`，`Buffer.from(buffer)`，`Buffer.from(string[, encoding\])`
- [`Buffer.alloc(size[, fill[, encoding\]])`](http://nodejs.cn/s/Du96og) 返回一个指定大小的新建的的已初始化的 `Buffer`。 
- [`Buffer.allocUnsafe(size)`](http://nodejs.cn/s/TWpeWk) 和 [`Buffer.allocUnsafeSlow(size)`](http://nodejs.cn/s/PUENLw) 分别返回一个指定大小的新建的未初始化的 `Buffer`。 由于 `Buffer` 是未初始化的，因此分配的内存片段可能包含敏感的旧数据。

### 1.3 Buffer内存分配

如上一章最后提到，Buffer对象的内存分配不是在V8的堆内存中，而是在Node的C++层面实现内存的申请的，在JavaScript中分配内存，这避免了大量的内存申请的系统调用。



## 2 Buffer的转换

- 字符串转`Buffer`：`Buffer.from(string[, encoding\])`
- `Buffer`转字符串：`buf.toString([encoding[, start[, end]]])`

支持的编码：ASCII、UTF-8、UTF-16/UCS-2、Base64、Binary、Hex

对于不支持的编码，可以用iconv和iconv-lite两个模块可以支持更多的编码类型



## 3 Buffer的拼接

### 3.1 中文字符乱码

``` js
var fs = require('fs')

var rs = fs.createReadStream('test.md')
var data = ''
rs.on('data', function(chunk) {
  data += chunk
})
rs.on('end', function() {
  console.log(data)
})
```

如果文档中有中文字符，则有可能会出现乱码。

__原因__：因为这里的`data += chunk`实际上等同于`data = data.toString() + chunk.toString()`。每次读取一定长度，而宽子节的中文字符在UTF-8编码下占用3个元素，所以有可能被截断。比如一个汉字（e6 9c 88）被截断成两半，2个字节（e6 9c）和1个字节（88）都不能形成文字，只能显示乱码。

### 3.2 正确拼接Buffer

- `readable.setEncoding(encoding)`  。上例中加上`rs.setEncoding('utf8')`即可。但依然是按段读取。
- `Buffer.concat()`。用一个数组存储接收到的所有`Buffer`片段，再使用`Buffer.concat()`生成一个合并的`Buffer`对象。



## 4 Buffer与性能

用`ab`命令（apache的性能测试工具）进行性能测试，对比传输相同内容的字符串和`Buffer`的QPS和传输率，传输`Buffer`的性能提高近一倍。

文件读取：读取同一个大文件，highWaterMark值（分段读取，每次读取的长度）的大小越大，读取速度越快。