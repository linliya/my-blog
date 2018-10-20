---
title: bytes & blob
date: 2018-09-12 14:57:10
tags:
---

### blob是什么？

blob, 即binary large object，二进制大对象，MDN上的定义是:
> Blob 对象表示一个不可变、原始数据的类文件对象。Blob 表示的不一定是JavaScript原生格式的数据。File 接口基于Blob，继承了 blob 的功能并将其扩展使其支持用户系统上的文件

Blob 对象表示一个二进制文件的数据内容,可以看做是存放二进制数据的容器，一直以来，JS都没有比较好的可以直接处理二进制的方法。而Blob的出现，使我们可以通过JS直接操作二进制数据

> 拓展: MySql/Oracle数据库中，有一种Blob类型，专门存放二进制数据


### 构建blob对象的方式

#### Blob构造函数
```
new Blob(array [, options])
```
通过Blob构造函数可以创建blob对象，Blob构造函数接受两个参数。第一个参数是数组，成员是字符串或二进制对象，表示新生成的Blob实例对象的内容；第二个参数是可选的，是一个配置对象，目前只有一个属性type，它的值是一个字符串，表示数据的 MIME 类型，默认是空字符串。

```
var obj = { hello: 'world' };
var blob = new Blob([ JSON.stringify(obj) ], {type : 'application/json'});
```

Blob具有两个实例属性size和type，分别返回数据的大小和类型。
> 数据类型参见[MIME类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

- slice方法
可以实现大文件的[分片上传](https://scarletsky.github.io/2015/01/27/javascript-upload-slice-file-using-blob/)
> slice() 方法接受三个参数，起始偏移量，结束偏移量，还有可选的 mime 类型。如果 mime 类型，没有设置，那么新的 Blob 对象的 mime 类型和父级一样


###  File对象
文件(File) 接口提供有关文件的信息，并允许网页中的 JavaScript 访问其内容

File 对象是特殊类型的 Blob，且可以用在任意的 Blob 类型的 context 中

all files are blobs(所有的flies对象都是blobs对象)

not all blobs are files(不是所有的blobs对象都是file对象)

#### 1. 怎么拿到一个file对象？
- input 输入框
```
<input type="file" onChange="display(event)">
  <img id="img"/>
  function display(e) {
    let img = document.getElementById('img')
    img.src = URL.createObjectURL(e.target.files[0])
    img.onload = function () {
      URL.revokeObjectURL(this.src)
    }
  }
```
- drag and drop 拖放
```
element.ondrop = (e) => {
  let file = e.dataTransfer.files[0]
  console.log(file)
}
```
#### 2. file对象的属性

- `file.name` 文件名
- `file.size` 文件大小
- `file.type` 文件类型
- `file.lastModifiedDate` 文件最后修改时间

#### 3. 拿到file对象后如何处理?
- FileReader.readAsArrayBuffer(Blob | File)
> 开始读取指定的 Blob中的内容, 一旦完成, result 属性中保存的将是被读取文件的 ArrayBuffer 数据对象
- FileReader.readAsDataURL(Blob | File)
> 开始读取指定的Blob中的内容。一旦完成，result属性中将包含一个data: URL格式的字符串以表示所读取文件的内容。
- FileReader.readAsText(Blob | File)
> 开始读取指定的Blob中的内容。一旦完成，result属性中将包含一个字符串以表示所读取的文件内容。
- FileReader.abort()
> 中止读取操作。在返回时，readyState属性为DONE

例子: 用户上传文件后预览

### ArrayBuffer 对象 & TypedArray视图 & Dataview视图
简单来说，ArrayBuffer是装着二进制数据的对象， [ArrayBuffer](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)对象代表原始的二进制数据，[TypedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) 视图用来读写简单类型的二进制数据，[DataView](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView)视图用来读写复杂类型的二进制数据。
ArrayBuffer 不能直接操作，而是要通过TypedArray对象或 DataView 对象来操作

```
// 创建一个8字节的ArrayBuffer
var b = new ArrayBuffer(8);  
  
// 创建一个指向b的视图v1，采用Int32类型，开始于默认的字节索引0，直到缓冲区的末尾  
var v1 = new Int32Array(b);  
  
// 创建一个指向b的视图v2，采用Uint8类型，开始于字节索引2，直到缓冲区的末尾  
var v2 = new Uint8Array(b, 2);  
  
// 创建一个指向b的视图v3，采用Int16类型，开始于字节索引2，长度为2  
var v3 = new Int16Array(b, 2, 2);  
```

上面代码里变量的数据结构如下表所示：
![](https://user-gold-cdn.xitu.io/2018/9/12/165cc5d7d759e61d?w=846&h=470&f=png&s=33252)


### URL对象
调用 URL 对象的 createObjectURL 方法，传入一个 File 对象或者 Blob 对象，能生成一个链接
```
var blob = new Blob(['Hello World!'])
var a = document.createElement('a')
a.href = window.URL.createObjectURL(blob)
a.download = 'a.txt'
a.textContent = 'Download'

document.body.appendChild(a)
```
页面上生成了一个超链接，点击它就能下载一个名为 a.txt 的文件，里面的内容是 Hello World!

### Buffer
在nodeJs中，我们还可以通过Buffer类进行buffer => base64的转化,base64是将二进制编码为文本的一种编码规则，将base64的值直接赋给img.src可以实现图片的展示

在JavaScript中，有2个函数分别用来处理解码和编码base64 字符串：

- atob() 解码通过base-64编码的字符串数据
- btoa() 从二进制数据“字符串”创建一个base-64编码的ASCII字符串

nodejs中,我们可以实现filesystem => buffer => base64的转化
```
function readDataAsBase64 (filepath) {
  let buffer = fs.readFileSync(filepath)
  let base64 = Buffer.from(buffer).toString('base64')
  return `data:image/${ext};base64,${base64}`
}
```

### IndexedDB

IndexedDB 就是浏览器提供的本地数据库，它可以被网页脚本创建和操作。IndexedDB 允许储存大量数据，提供查找接口，还能建立索引。

### 格式转换
#### 1. filesystem => blob => file
`blob`对象转`file`对象，`blob`可以看成存放二进制文件的容器，因此可以将系统中的文件读取后转化成`blob`,`blob`又可以转化成`file`对象

```
# nodejs中
function blobToFile(filePath) {
    fs.openSync(filePath, 'w')
    
    let buffer = fs.readFileSync(filePath)
    
    let blob = new Blob([buffer], {type: 'application/octet-stream'})
    let file = new File([blob], filename)

    return file  
}
```

#### 2. filesystem => blob => dom(URL) 
选择本地图片后预览
```
<input type="file" onChange="display(event)">
<img id="img"/>

function display(e) {
  let img = document.getElementById('img')
  img.src = URL.createObjectURL(e.target.files[0])
  img.onload = function () {
    URL.revokeObjectURL(this.src)
  }
}
```

#### 3. filesystem => blob => web 
上传blob & 上传文件

```
function uploadBlob (url, blob, callback) {
  let request = new XMLHttpRequest()
  request.open('POST', url)
  request.send(blob)
  request.upload.onprogress = callback
}
```

```
function uploadFiles (url, files) {
  let request = new XMLHttpRequest()
  request.open('POST', url)
  let formData = new FormData()
  for (let i = 0; i < files.length; i++) {
    formData.append(files[i].name, files[i]
  }
  request.send(formData)
}

```

#### 4. web => blob => dom
从服务器下载一张图片并显示
```
var xhr = new XMLHttpRequest()
xhr.open('get', '1.jpg', true)
xhr.responseType = 'blob'
xhr.onload = function() {
  if (this.status == 200) {
    var blob = this.response
    var img = document.createElement('img')
    img.onload = function(e) {
      window.URL.revokeObjectURL(img.src)
    }
    img.src = window.URL.createObjectURL(blob)
    eleAppend.appendChild(img)
  }
}
xhr.send()
```


#### 5. blob <=> indexedDB

```
# blob => indexedDB
store.put(blob)
# get blob from indexedDB
let request = blob = store.get(key)
request.onsuccess = function () {
  let blob = request.result
}
```

#### 6. blob <=> worker

```
worker.postMessage(blob)
worker.onmessage = function (e) {
  let blob = e.data
}
```
总结: 本文简单地介绍了跟blob相关的一些内容，只是抛砖引玉，大家有错误可以指出

参考链接: 
- https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types
- https://developer.mozilla.org/zh-CN/docs/Web/API/Blob
- https://www.zhangxinxu.com/wordpress/2013/10/understand-domstring-document-formdata-blob-file-arraybuffer/
- http://es6.ruanyifeng.com/#docs/arraybuffer
- https://scarletsky.github.io/2015/01/27/javascript-upload-slice-file-using-blob/
