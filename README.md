# nodejs

### 一、浏览器

#### 1、浏览器内核：

​			我们通常说的浏览器内核其实是浏览器的排版引擎（layout engine），也称为浏览器引擎（browser engine）、页面渲染引擎（rendering engine）或样板引擎。

* Gecko：早起被Netscape 和 Mozilla Firfox 浏览器使用
* Trident：微软开发，被 IE4~IE11浏览器使用，但Edge浏览器已经转向Blink
* Webkit：苹果基于KHTML开发、开元的，用于Safari，Google Chome 之前也在使用
* Blink：是Webkit 的一个分支，Google开发，目前用于Google Chome、Edge、Opera 等
* 等等。。。。

##### 1.1、浏览器渲染引擎的工作过程：





<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210206201933296.png" alt="image-20210206201933296" style="zoom:50%;" />

首先解析html（通过html parser）生成DOM树， 同时css也对进行解析（通过css parser），最后附加（attachment） 成为渲染树（render tree）同时生成（layout）布居树 ，然后进行绘制（painting），最终展示（display）

##### 1.2、javascript引擎

* javascript引擎会将我们写的 js 代码解析成二进制文件，供cpu执行
* js执行顺序：高级语言 ==》 汇编语言 ==》 机器语言（二进制文件）
* 在HTML解析的时候，如果遇到了 script 标签，会停止解析HTML，而去加载和执行 javascript 代码
  * 因为 javascript 可以操作我们的 DOM，所以浏览器希望将 HTML 解析的 DOM 和 javascript 操作之后的 DOM 放到一起 生成最终的 DOM树，而不是频繁的去生成新的 DOM 树

为什么需要引擎？

* 实际上我们编写的 javascript 无论交给浏览器还是 node 执行，最终都是被CPU执行的
* 但CPU 只认识自己的指令集，实际上是机器语言 才能被CPU 所执行
* 所以我们需要javascript 引擎帮助我们将 javascript 代码翻译成 CPU 指令来执行

常见的 javascript 引擎：

* SpiderMonkey：第一款javascript 引擎，由 Brendan Eich 开发（js作者）
* Chakra：微软开发，用于IE 浏览器
* JavaScriptCore：Webkit 中的JavaScript 引擎，Apple 公司开发
* V8：Google 开发的强大的 JavaScript 引擎，也帮助Chrome 从众多浏览器中脱颖而出

##### 1.2.1、V8引擎

* V8是用C++编写的Google开源高性能JavaScript 和webAssembly 引擎，它用于 Chrome 和nodejs等
* 它实现ECMAScript 和 webAssembly ， 并在Windows7 或更高版本， macOS 10.12+ 和使用 x64， IA-32， ARM 或MIPS 处理器的Linux 系统上运行
* V8可以独立运行，也可以嵌入到任何 C++ 应用程序中

运行原理：

* 本身源码非常复杂，有超过100W行的c++代码，简单了解其运行原理：

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210206205009363.png" alt="image-20210206205009363" style="zoom:50%;" />

* Parse 模块会将javascript代码转换成AST（抽象语法树），这是因为解释器（Ignition）并不能直接认识javascript代码
  * 如果函数没有被调用，那么是不会转换成AST 的
  * Parse 的 V8 官方文档： https://v8.dev/blog/scanner
* Ignition 是一个解释器，会将AST转换成 ByteCode （字节码）
  * 同时会收集 TurboFan 优化所需要的信息（比如函数参数的类型信息，有了类型才能进行真实的运算）
  * 如果函数只调用一次，Ignition 会解释执行 ByteCode
  * Ignition 的 V8 官方文档 ： https://v8.dev/blog/ignition-interperter
* TurboFan 是一个编译器，可以将字节码编译为CPU可以执行的机器码
  * 如果一个函数被调用多次，那么就会标记为热点函数，那么就会经过TurboFan转换成优化的机器码，提高代码执行性能
  * 但是，机器码实际上也会被还原为ByteCode，这是因为如果后续执行函数的过程中，类型发生了变化（比如 sum函数原来执行的是number类型，后来执行变成了String 类型，之前优化的机器码并不能正确的处理运算，就会逆向的转换成字节码
  * TurboFan 的 V8官方文档： https://v8.dev/blog/turbofan-jit

### 二、node 的使用

#### 1·、安装node和n（node版本工具）

````js
npm install n -g
sudo n lts // 下载lts版本的node
sudo n latest // 下载最新版本的node
可以通过 sudo n 命令来切换node版本
````

#### 2、给node传递参数

````js
node index.js coderYang env=development
可以在node的全局对象process上面看到当前node程序的运行环境
````

![image-20210206211523964](/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210206211523964.png)

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210206211553209.png" alt="image-20210206211553209" style="zoom:50%;" />

console.log() 控制台打印

console.clear() 清空上面代码在控制台上的输出

console.trace()

#### 3、常用的全局对象

* 不能再命令行使用：

  * `__dirname, __filename, exports, module, require()`

* process对象 :

  * 提供 node 进程相关信息

* console对象：

  * log 、clear、 trace 等

* 定时器：

  * setTimtout、setInterval、setImmediate（立即执行）、process.nextTick()

* global对象：

  * 类似浏览器上的 window 对象，但不会将顶层定义的变量放到global中

    * ````js
      var name = 'coderYang'
      console.log(global.name) // undefined
      ````

    

#### 4、前端模块化

what is module?

##### 4.1、exports 导出

````js
// bar.js
const name = 'coderYang';
const age = 18;
let message = 'coderyang';
function sayHello(name) {
  console.log('hello ' + name);
}
exports.name = name;
exports.age = age;
exports.message = message;
exports.sayHello = sayHello;

// main.js
const bar = require('./bar.js')
const { name, age, message, sayHello } = require('./bar.js')
bar.name // coderYang
bar.age // 18
bar.message // ooderyang
bar.sayHello('koby') // hello koby
````

* node 中实现commonJs 的本质就是对象的引用赋值（浅拷贝）
  * exports是一个对象，require会获取这个对象

##### 4.2、module.exports 导出

````js
module.exports.name = 'koby'
````

* node 内部默认 module.exports = exports 
* 如果有一天，module.exports = {} , 那么怎么改exports 都不会改变内部的值，因为 module.exports换了新对象

##### 4.3、require 细节

* require是一个函数， 可以帮助我们引入一个文件（模块）中导出的对象
* require 的查找规则：
  * 导入格式：require(X)
    * 情况1：X是一个核心模块，比如 http、path
      * 直接返回核心模块，并停止查找
    * 情况2：X是以 './'、'../'、'/'开头的
      * 查找本地文件'./' 和 '../' 是相对路径，'/' 会去电脑根目录开始查找
        * 如果有后缀名，按照后缀名查找对应文件
        * 如果没有后缀名
          * 1、直接查找文件X
          * 2、查找 X.js 文件
          * 3、查找 X.json 文件
          * 4、查找 X.node 文件
          * 没有找到对应的文件，将 X 作为一个目录
            * 查找目录下面的 index文件
              * 1、查找 X/index.js 文件
              * 2、查找 X/index.json 文件
              * 3、查找 X/index.node 文件
        * 没找到，报错 not found
    * 情况3：直接写一个X（没路径），并且X不是一个核心模块
      * 去项目 路径中的 node_modules中查找 X
      * 没找到就去上一层的node_modules中找
      * 最终找到电脑根目录中的node_modules中找
      * 找不到 not found

``require加载是同步的``

* @ 模块在被第一次引入时，模块中的 js 文件会被执行一次
* @@ 模块被引入多次时，会缓存，最终只会加载（运行）一次
  * 为什么只会加载运行一次？
    * 每一个模块的 module 都会有一个属性， loaded
    * 为false 时表示没加载，为 true时表示已加载
* @@@ 如果循环引用，加载顺序时怎样的?
  * 图结构
  * 图结构在遍历 的过程中，有深度优先搜索（DFS，depth first search）和广度优先搜索（BFS， breadth first search）
  * Node 采用的是深度优先算法：main -> aaa -> ccc -> ddd -> eee -> bbb
  * <img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210208142256311.png" alt="image-20210208142256311" style="zoom:50%;" />

##### 4.4、commonJs规范缺点

* commonJs 加载模块是同步的：
  * 同步意味着只有等到对应的模块加载完毕，当前模块中的内容才会被执行；
  * 这个在服务器不会有什么问题，因为服务器加载的js文件都是本地文件，加载速度非常快；
* 如果放到浏览器里面：
  * 浏览器加载js文件需要从服务器将文件下载下来，之后在加载运行；
  * 那么采用同步的就意味着后续的js代码都无法正常运行，即使是一些简单的DOM操作；
* 所以在浏览器里面我们通常不使用commonJs规范
  * 当然，在webpack使用commonJs是另外一回事；
  * 因为webpack会将我们的代码转换成浏览器可以执行的代码；
* 早期为了实现前端模块化会使用AMD、CMD



##### 4.5 ES6 Module

###### 4.5.1、常见导出方式

* 第一种：

````js
export const name = 'coderYang';
export function sayHello(name) {
  console.log('你好啊'， name)
}
export const sleap = function (name) {
  console.log(name + '， 正在睡觉')
}
````

* 第二种：

````js
export {} // 这里的{} 不是对象，而是放置要导出的变量的引用列表
````

* 第三种：{}导出时， 给变量起别名

````js
export {
	name as Aname,
  age as Aage,
  sayHello as ASayHello
}
````

###### 4.5.2、常见导入方式

* 1、普通导入方式

````js
import {name, age, sayHello} from './modules/bar.js'
````

* 2、起别名

````js
import {name as AName, age as AAge, sayHello as ASayHello} from './modules/bar.js'
````

* 3、起别名对象接收

````js
import * as bar from './modules/bar.js'
bar.name........
````

###### 4.5.3、import 和 export 结合使用

````js
// foo.js
export {name, age, sayHello}
//index.js
import {name, age, sayHello} from './modules/foo.js'
name.....
````

###### 4.5.4 default的用法

###### 4.5.5 import() 函数

* 通过import 加载一个模块是不可以放在逻辑代码中的

````js
// 会报语法错误
if (true) {
  import App from './App.js'
}

// 报错原因： import 依赖关系是在代码解析的时候就会执行，而js代码执行在 解析-》AST-》二进制-》执行，在解析的时候引擎会不知道该不该引入这个依赖文件所以报错
````

* 有没有办法呢？

````js
if (true) {
  const promise = import('./App.js')
  promise.then(res => {
    console.log(res.name)
    console.log(res.age)
  }).catch(err => {
    console.log(err)
  })
}
````

###### 4.5.6 总结 ES Module 的加载过程

* ES Module加载js文件的过程是编译（解析）时加载的，并且是异步的：
  * 编译时（解析）加载，意味着import 不能和运行时相关的内容放在一起使用
  * 比如from 后面的路径需要动态获取；
  * 比如不能将 import 放到 if 语句的代码块中；
  * 所以我们有时候也称 ES Module 是静态解析的，而不是动态或者运行时解析的；
* 异步意味着：
  * JS引擎在遇到import 时会去获取这个 js 文件，但是这个获取的过程是异步的，并不会阻塞主线程继续执行
  * 也就是说设置了 type="module" script标签引入的文件 相当于在script标签上加上了 async属性
  * 如果我们后面所有普通的 script 标签以及对应的代码，那么 ESModule 对应的 js 文件和代码不会阻塞他们的执行

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210208165405515.png" alt="image-20210208165405515" style="zoom:50%;" />

##### 4.6、CommonJs 和 ES Module交互

* 结论一：通常情况下，commonJs不能加载ES Module
  * 因为CommonJS是同步加载的，但是ES Module必须经过静态分析等，无法在这个时候执行javascript代码
  * 但是这个并非绝对的，某些平台在实现的时候可以对代码进行针对性的解析，也可能会支持
  * Node当中不支持
* 结论二：多数情况下， ES Module 可以加载CommonJS
  * ES Module在加载 CommonJS时， 会将其module.exports导出的内容作为default 发出方式来使用
  * 这个一谈要看具体的实现，比如webpack 中是支持的、Node 最新的Current 版本也是支持的
  * 但在最新的LTS版本中不支持

#### 5、总结

* node中如果想要使用 ES Module
  * 需要把文件的后缀名改为 .mjs
  * 在 package.json 文件中把 type 改成 module



### 三、node 中常见的内置模块

#### 1、path 模块

* 用于对路径或文件进行处理，提供了很多好方法
* 在 Mac OS、Linux、Window上面的路径不一样
  * window上会使用 \ 或者 \\\\ 来作为文件路径的分隔符，当然目前也支持 /
  * 在 MacOS、Linux 的Unix 操作系统上使用 / 来作为文件路径的分隔符
* 如果在 windows 上写的项目部署到 linux 上面可能会出现路径问题
* 所以为了避免这些问题我们需要使用 path 进行路径拼接 path.resolve(__dirpath, 'aaa.txt')

##### 1.1 方法

* resolve 拼接路径的方法 ：`path.resolve('/User/pufy', 'a.txt')`  `/User/pufy/a.txt`
* dirname 获取当前文件所在文件夹路径 `path.dirname('/User/pufy/a.txt')`  `/User/pufy`
* basename 获取当前文件名 `path.basename('/User/pufy/a.txt')`   `a.txt`
* extname 获取当前文件扩展名 `path.extname('/User/pufy/a.txt')`  `.txt`
* join 路径拼接方法： `path.join('/User/pufy', 'a.txt')`  `/User/pufy/a.txt`

Join 和 resolve 的区别

* join 比较笨重，只会拼接字符串路径
* resolve 会根据文件在硬盘的地址进行拼接



#### 2、fs 模块

##### 2.1 stat 获取文件状态

* 同步获取文件状态statSync

````js
const fs = require('fs');
const fileName = './a.txt'
const info = fs.statSync(fileName)
console.log(info)
````

* 异步获取文件状态 stat

````js
fs.stat(fileName, (err, info) => {
  if (err) {
    console.log(err);
    return
  }
  console.log(info)
})
````

* 异步获取文件状态 Promise 方式

````js
fs.promises.stat(fileName).then(res => {
  console.log(res)
}).catch(err => {
  console.log(err)
})
````

##### 2.2 文件描述符

* 在POSIX 系统上（MAC、Linux），对每个进程，内核都维护着一张当前打开着的文件和资源表格。
* 每个打开的文件都分配了一个称为文件描述符的简单数字标识符
* 在系统层，所有文件系统操作都是用这些文件描述符来标识和跟踪每个特定的文件
* Windows 系统是通了一个虽然不同但概念上类似的机制来跟踪资源

##### 2.3 文件的读写

* 简单使用

````js
const content = '你好,爸爸'
fs.writeFile(filePath, content, err => {
    console.log(err)
})
````

* flag：
  * w 打开文件写入，默认值
  * w+ 打开文件进行读写，如果不存在则创建文件
  * r+ 打开文件进行读写，如果不存在那么抛出异常
  * r 打开文件读取，读取时的默认值
  * a 打开要写入的文件，将流 放在文件末尾。如果不存在则创建文件
  * a+ 打开文件以进行读写，将流 放在文件末尾。如果不存在则创建文件

````js
const content = '你好,爸爸'
// 文件写入
fs.writeFile(filePath, content, {flag: 'a'}, err => {
    console.log(err)
})
````

* encoding: 编码

````js
// 文件读取
fs.readFile(filePath, {encoding: 'utf-8'}, (err, data) => {
    console.log(data)
})
// 如果不定义编码格式，data打印的是 Buffer对象
````

* 判断是文件夹还是文件
  * isFile()
  * isDirectory()

##### 2.4、文件夹操作

* 创建文件夹

````js
const dirName = './dir'
// 判断文件夹是否存在 fs.existsSync
if (!fs.existsSync(dirName)) {
    // 创建文件夹
    fs.mkdir(dirName, err => {
        console.log(err)
    })
}
````

* 读取文件夹下面的文件名

````js
// 读取文件夹中的文件
fs.readdir(dirName, (err, list) => {
    console.log(list)
})
````

* 重命名

````js
// 重命名
fs.rename('./dir', './pfy', err => {
    console.log(err)
})
````

* 文件复制案例

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210218111516060.png" alt="image-20210218111516060" style="zoom:100%;" />





#### 3、events模块

##### 3.1、基础方法

````js
const EventEmitter = require('events');
// 1、创建发射器
const emitter = new EventEmitter();
// 2、监听某一个事件
	// on || addListener : 互为别名
emitter.on('click', args => {
  console.log('监听click事件触发', args)
})

// 3、发射事件
emitter.emit('click', '你好啊，我是参数')
````

##### 3.2、获取信息

* emitter.eventNames()  获取监听事件集合
* emitter.listenerCount("click")  获取监听click事件的方法个数
* Emitter.listener("click") 获取监听click事件的具体函数方法



### 四、包管理工具

#### 1、npm

官方文档地址：`npmjs.org`或 `npmjs.com`



* npm install 原理
* <img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210220130130512.png" alt="image-20210220130130512" style="zoom:50%;" />



* 首先在package.json里找到 devDependencies 和 dependencies 
* 查找package-lock.json 文件
  * 没有：构建依赖关系，去registry仓库中下载压缩包，将压缩包存放在缓存中，最后压缩到node_modules文件夹下并生成 package-lock.json 文件
  * 有：检测依赖的一致性
    * 不一致：构建依赖关系，去registry中下载
    * 一致：查找缓存：
      * 没有：去registory中下载
      * 有：将缓存文件添加到node_modules中

* npm uninstall package 卸载包

#### 2、yarn



#### 3、npx



### 五、实现自己的脚手架工具

#### 1、尝试运行coderYang命令

* 首先在文件夹中 ` npm init -y`
* 创建index.js

````js
#!/usr/bin/env node
console.log('你好，爸爸');
````

* 编辑package.json 文件

````js
{
  "name": "nodejs",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "bin": { // 后加上的配置
    "coderYang": "index.js" // 配置命令名称
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
````

* 将文件配置到环境变量中 ` sudo npm link`
* 执行命令 ` coderYang `

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210220154747530.png" alt="image-20210220154747530" style="zoom:50%;" />



#### 2、解析coderYang命令后携带的参数

* 首先安装第三方库 commander ` npm i commander`
* 在index.js文件中使用 commander 

````js
#!/usr/bin/env node
const program = require('commander');
// 查看版本号
program.version(require('./package.json').version)
// 改变命令,使得 -v --version 都生效
program.version(require('./package.json').version, '-v --version')
// 必须写这个,不解析不生效
program.parse(process.argv)
````

![image-20210220155959529](/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210220155959529.png)

![image-20210220160639453](/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210220160639453.png)

### 六、buffer和浏览器的事件循环

#### 1、数据的二进制

* 计算机中所有的内容：文字、数字、图片、音频、视频最终都会使用二进制来标识
* javascript 可以直接去处理非常直观的数据：比如字符串，我们通常展示给用户的也是这些内容
* javascript不可以处理图片：
  * 网页端，图片一直是交给浏览器来处理
  * javascript或者 HTML 只是负责告诉浏览器一个地址
  * 浏览器负责获取图片，并且最终将图片渲染出来
* 对于服务器来说不一样：
  * 服务器要处理的本地文件类型相对较多
    * 例如：一个保存文本的文件并不是使用utf8 进行编码的，而是使用 GBK ，name我们必须读取到他们的二进制数据，在通过GBK 转换成对应的文字
    * 例如：读取的是一张图片数据（二进制），在通过某些手段对图片数据进行二次处理（建材、格式转换、旋转、添加滤镜），node中有一个sharp的库，就是读取图片或者传入图片的 Buffer 对其在进行处理
    * 在node中通过 TCP 建立长连接，TCP 传输的是字节流，我们需要将数据转换成字节在进行传入，并且需要知道传输字节的大小（客户端需要根据大小来判断读取多少内容）

#### 2、buffer和二进制

* 做前端开发会很少和二进制打交道，但是对服务端为了做很多的功能，我们必须去操作他的二进制数据
* 所以node 为了可以方便开发者完成更多功能，提供给我们一个类Buffer，并且是全局的
* 可以吧Buffer看成存储二进制数据的数组（每一项都是8位二进制）
  * 为什么是8位？
  * 在计算机中，很少的情况我们会直接操作一位二进制，因为一位二进制存储的数据是非常有限的；
  * 所以通常会将8位合在一起作为一个单元，这个单元称为一个字节（byte）；
  * 也就是说 1byte = 8bit，1kb = 1024byte， 1M = 1024kb；
  * 比如很多编程语言中的int 类型是4个字节，long类型是 8个字节；
  * 比如 TCP 传输的是字节流，在写入和读取时都需要说明字节的个数；
  * 比如RGB的值分别都是 255， 所以在本质上在计算机中都是用一个字节存储的。

#### 3、buffer和字符串

#### 4、Buffer的使用

* 创建方式1：

````js
let message = 'Hello'
let buffer = new Buffer(message)
console.log(buffer)
````

![image-20210223212002620](/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210223212002620.png)

* 创建方式2：

````js
let message = 'Hello'
let buffer = Buffer.from(message)
console.log(buffer)
````

![image-20210223212206998](/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210223212206998.png)

* 创建方式3：(分配内存空间)

````js
const buffer = Buffer.alloc(8);
console.log(buffer)
buffer[0] = 88;
buffer[1] = 0x88;
console.log(buffer);
````

![image-20210223213308757](/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210223213308757.png)

#### 5、Buffer和文件操作

下载sharp依赖自测

#### 6、事件循环

* 将我们编写的javascript 和浏览器或node建立起来的桥梁
* 浏览器的事件循环是一个我们编写的javascript代码和浏览器API调用（serTimeout/AJAX/监听事件等）的一个桥梁，桥梁之间他们通过回调函数进行沟通
* Node的事件循环是一个我们编写的javascript代码和系统调用（file system、network等）之间的一个桥梁，桥梁之间他们通过回调函数进行沟通

![image-20210223215817560](/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210223215817560.png)

#### 7、进程和线程

抽象理解：

* 进程（process）：计算机已经运行的程序
* 线程（thread）：操作系统能够运行运算调度的最小单位

直观解释：

* 进程：我们可以认为，启动一个应用程序，就会默认启动一个进程（可能是多个进程）
* 线程：每一个进程中，都会启动一个线程用来执行程序中的代码，这个线程被称之为主线程
* 所以我们也可以说进程是线程的容器

例子：

* 操作系统类似一个工厂
* 工厂中有很多车间，车间就是进程
* 每个车间可能有一个以上的忙碌的工人（方法、函数），工人就是线程（方法函数执行）

#### 8、多进程多线程开发

* 操作系统是如何做到同时让多个进程（听歌、写代码、查资料）同时工作的？
  * 这是因为CPU的运算速度非常快，他可以快速的在多地进程之间迅速切换
  * 当我们的进程中的线程获取到时间片时，就可以快速执行我们编写的代码
  * 对于用户来说是感受不到这种快速的切换的

#### 9、浏览器的事件循环

![image-20210223222514836](/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210223222514836.png)

#### 10、宏队列和微队列

队列有两种

* 宏任务队列（macrotask queue）：ajax、setTimeout、setInterval、DOM监听、UI Rendering 等
* 微任务队列（microtask queue）：Promise的 then 回调、Mutation Observer API、queueMicrotask() 等

执行顺序：

* 浏览器默认先执行微任务队列
* 只有微任务队列中的任务全部执行完，才会执行宏队列
* 当宏队列任务执行时产生微任务，回去先去执行微任务队列的方法
* 当且仅当微任务队列彻底执行完，才会执行宏任务队列





#### 11、阻塞IO和非阻塞IO

* 如果我们希望在程序中对一个文件进行操作，那么我们就需要打开这个文件：通过文件描述符
  * javascript可以直接对一个文件进行操作吗？
  * 我们任何程序中的文件操作都是需要通过系统调用（操作系统的文件系统）
  * 事实上对文件的操作，是一个操作系统的系统调用（IO系统，IO是输入、输出）
* 操作系统通常为我们提供了两种调用方式：阻塞式调用和非阻塞式调用
  * 阻塞式调用：调用结果返回之前，当前线程处于阻塞态（阻塞态CPU是不会分配时间片的），调用线程只有在得到调用结果之后才会继续执行
  * 非阻塞式调用：调用执行之后，当前线程不会停止执行，只需要过一段时间来检查一下有没有结果返回即可

但是非阻塞IO也会存在一定的问题：我们并没有获取到需要读取（我们以读取为例）的结果

* 那么就意味着为了可以知道是否读取到了完整的数据，我们需要频繁的去确定读取到的数据是否完整
* 这个过程叫 轮训操作

轮训工作谁来做？

* 如果我们的主线程频繁的去进行轮训的工作，那么必然会大大降低性能
* 并且开发中我们可能不止是一个文件的读写，可能是多个文件
* 而且可能是多个功能：网络IO、数据库IO、子进程调用

libuv提供了一个线程池

* 线程池会负责所有相关操作，并且会通过轮训或者其他的方式等待结果
* 当获取到结果是，就可以将对应的回调放到事件循环（某一个事件队列）中
* 事件循环就可以负责接管后续的回调工作，告诉javascript应用程序执行对应的回调函数

阻塞和非阻塞，同步异步有什么区别？

* 阻塞和非阻塞式对于被调用者来说的
  * 在我们这里就是系统调用，操作系统为我们提供了阻塞调用和非阻塞调用
* 同步异步是对于调用者来说的
  * 在我们这就是自己的程序
  * 如果在发起调用后，不会进行其他任何的操作，只是等待结果，这个过程称之为同步调用
  * 如果我们在发起调用后，并不会等待结果，继续完成其他的工作，等到有回调时再去执行，这个过程就是异步调用
* libuv采用的就是非阻塞异步IO的调用方式

#### 12、node的事件循环的阶段

libuv

#### 13、stream 流

### 七、http模块

#### 1、web服务器

````js
const http = require('http');
// 创建一个web服务器
const server = http.createServer((request, response) => {
    response.end('hello http server haha')
})
// 启动服务器并且指定端口号和主机
server.listen(8888, '0.0.0.0', () => {
    console.log('服务器启动成功!')
    console.log('地址: http://localhost:8888')
})
````

或

````js
const server = new http.Server((req, res) => {
    res.end('hello node server')
})
server.listen(8000, () => {
    console.log('服务器启动成功~  http://localhost:8000')
})
````

两种方法创建启动服务器，本质是一模一样的



* Server 通过listen方法来开启服务器，并且在某一个主机和端口上监听网络请求
  * 也就是当我们通过ip:port 的方式发送到我们监听的web服务器上时
  * 我们就可以对其进行相关的处理
* listen函数有三个参数：
  * 端口port：可以不传，系统会默认分配端口，后续项目中我们会写入到环境变量中
  * 主机host：通常可以传入localhost、IP地址127.0.0.1、或者IP地址0.0.0.0，默认是0.0.0.0
    * localhost：本质上是一个域名，通常情况下被解析成127.0.0.1
    * 127.0.0.1：回环地址（Loop Back Address）， 表达的意思是我们主机自己发出去的包，直接被自己接受
      * 正常的数据包经常 应用层 - 网络层 - 数据链路层 - 物理层
      * 而回环地址，是在网络层直接就被获取到了，是不会经常数据链路层和物理层的
      * 比如我们监听127.0.0.1时，在同一个网段下的主机中，通过IP地址是不能访问的
    * 0.0.0.0：
      * 监听IPV4上所有的地址，再根据端口找到不同的应用程序
      * 比如我们监听0.0.0.0时，在同一个网段下的主机中，通过IP地址是可以访问的



#### 2、request对象

封装了客户端给我们服务器传过来的所有信息

* get请求参数管理：

````js
const http = require('http');
const url= require('url');
const qs = require('querystring')

const server = new http.Server((req, res) => {
    const { pathname, query } = url.parse(req.url)
    const { username, password } = qs.parse(query)
    console.log(pathname, username, password)
    res.end('服务器')
})

server.listen(8000, () => {
    console.log('服务器启动成功~  http://localhost:8000')
})

````

* post请求参数管理：

````js
const http = require('http');
const url= require('url');
const qs = require('querystring');

const server = new http.Server((req, res) => {
    if (url.parse(req.url).pathname === '/login') {
        if (req.method === 'POST') {
            req.setEncoding('utf-8');
            req.on('data', data => {
                const { username, password } = JSON.parse(data)
                console.log(username, password)
            })
        }
    } else  {
        console.log(url.parse(req.url))
    }
    res.end('服务器')
})
server.listen(8000, () => {
    console.log('服务器启动成功~  http://localhost:8000')
})
````

#### 3、文件上传

#### 4、express

````js
const express = require('express');

const app = express();

app.get('/', (req, res, next) => {
    req.setEncoding('utf-8')
    res.end('Hello GET express')
})

app.post('/', (req, res, next) => {
    res.end('Hello POST express')
})

app.listen(8000, () => {
    console.log('成功启动express服务 http://localhost:8000')
})
````

#### 5、中间件

* express本质上是一系列中间件函数的调用

* 中间件本质是传递给express 一个回调函数
* 这个回调函数有三个参数
  * 请求对象（request)
  * 响应对象（response）
  * next 函数（在express中定义的用于执行下一个中间件的函数）

中间件可以执行的任务：

* 执行任何代码
* 更改请求（request) 和响应 （response) 对象
* 结束请求-响应周期（返回数据）
* 调用栈中的下一个中间件

将中间件用到我们的应用程序中：

* express主要提供了两种方式：app/router.use 和 app/router.methods
* 可以是 app , 也可以是 router
* methods指的是畅通的请求方式 app.get  或app.post 等

<strong style="font-size:17px;color:red;">    将匹配上的中间件都放到执行栈中，从头开始执行，若执行到某个中间件时，调用了next()，那么会继续执行下一个中间件，一般会将最后的中间件写上 req.end()</strong>

##### 5.1、最普通的中间件：

````js
const express = require('express');
const app = express();
app.use((req, res, next) => {
    console.log('中间件1')
    next() // 调用next执行下一个执行站中的中间件（中间件2）
})
app.use((req, res, next) => {
    console.log('中间件2')
    next()
})
app.use((req, res, next) => {
    console.log('中间件3')
    res.end('hello world') // res.end必须返回，切只能返回一个
})
app.listen(8000, () => {
    console.log('最普通的中间件服务器启动成功~')
})
````

##### 5.2、链式注册中间件

````js
app.get('/home', (res, req, next) => {
    console.log('中间件1')
    next()
}, (res, req, next) => {
    console.log('中间件2')
    next()
}, (res, req, next) => {
    console.log('中间件3')
    next()
}, (res, req, next) => {
    console.log('中间件4')
    req.end('hello world')
})
````

##### 5.3、内置转换参数测中间件

````js
// 转化json的中间件
app.use(express.json());
// 转化url的中间件 http://127.0.0.1:8000?name=coderYang&password=123456
// 解析urlencoded类型的数据
app.use(express.urlencoded({extended: true}))

app.post('/login', (req, res, next) => {
    console.log(req.body)
})
````

##### 5.4、解析 form-data 数据

````js
const express = require('express');
const multer = require('multer');

const app = express()

const upload = multer()

// 通过form-data传入非文件数据使用这个
app.post('/login', upload.any(), (req, res, next) => {
    console.log(req.body)
    const { username, password } = req.body
    res.end(`爸爸,您的用户名为${username}, 密码为${password}`)
})


app.listen(8000, () => {
    console.log('解析form-data数据服务器启动成功~')
})
````

##### 5.5、通过form-data 上传文件：

````js
const path = require('path');

const express = require('express');
const multer = require('multer');

const app = express()

const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, './uploads/')
    },
    filename: (req, file, cb) => {
        cb(null, Date.now() + path.extname(file.originalname));
    }
})
const upload = multer({
    storage
})


app.post('/upload', upload.single('file'), (req, res, next) => {
    console.log(req.body)
    const { username, password } = req.body
    res.end(`文件上传成功!`)
})
app.listen(8000, () => {
    console.log('解析form-data数据服务器启动成功~')
})

````

<span style="color: red">*永远不要讲multer放在全局中间件中使用</span>

##### 5.6、保存日志

````js
const fs = require('fs');
const express = require('express');
const morgan = require('morgan'); // 第三方库

const app = express();

const writerStream = fs.createWriteStream('./logs/access.log', {
    flags: 'a+'
})
app.use(morgan('combined', {stream: writerStream}))

app.get('/home', (req, res, next) => {
    res.end('hello world')
})

app.listen(8000, () => {
    console.log('保存日志服务器启动成功~')
})
````

##### 5.7、params和 query 传参方式

````js
const express = require('express');
const app = express()

// params 方式
app.get('/products/:id/:user', (req, res, next) => {
    console.log(req.params)
    res.end('Hello big brother')
})

// query 方式
app.post('/login', (req, res, next) => {
    console.log(req.query)
    res.end('用户登录成功~')
})
app.listen(8000, '0.0.0.0',() => {
    console.log('http://127.0.0.1:8000')
})
````

注：res.end() 只能返回 String、Buffer类型的数据，所以返回 JSON格式的数据得通过 JSON.stringfy()

​		res.json() 这个API就可以代替上边的res.end() ， 奥利给



#### 6、express路由

````js
const express = require('express');
const router = express.Router()
router.get('/:id', (req, res, next) => {
  console.log(req.params)
  res.json({name: '123'})
})
module.exports = router
````

````js
const express = require('express');
const userRouter = require('./router/user')
const app = express();
app.use('/user', userRouter)
app.listen(8001, () => {
  console.log("路由服务器启动成功~");
});

````



#### 7、koa

##### 7.1 基本使用

````js
const Koa = require('koa');

const app = new Koa()

app.use((ctx, next) => {
    if (ctx.request.url === '/login') {
        if (ctx.request.method === 'POST') {
            ctx.response.body = '你好啊,欢迎登陆'
        } else {
            ctx.response.body = '请使用POST请求方式'
        }
    } else {
        ctx.response.body = '访问接口不存在'
    }
})

app.listen(8000, () => {
    console.log('koa 服务已启动')
})
````



##### 7.2、koa路由

````js
// 安装koa-router
const Koa = require('koa');
const userRouter = require('./router/user')
const app = new Koa()
app.use((ctx, next) => {
    next()
})
app.use(userRouter.routes());
app.use(userRouter.allowedMethods()) // 找不到对应的请求方式返回 method not allowed
app.listen(8000, () => {
    console.log('koa 服务已启动')
})
````

````js
const Router = require('koa-router');
const router = new Router({prefix: '/users'}) // 前缀
router.get('/', (ctx, next) => {
    ctx.response.body = 'get users'
})
router.post('/', (ctx, next) => {
    ctx.response.body = 'post users'
})
module.exports = router
````

##### 7.3、 koa的 params 和 query 参数

````js
const Koa = require('koa');
const Router = require('koa-router');

const router = new Router({prefix: '/users'});
router.get('/:id', (ctx, next) => {
    console.log(ctx.request.params)
    console.log(ctx.request.query)
    ctx.response.body = 'hello world'
})

const app = new Koa()
app.use((ctx, next) => {
    next()
})

app.use(router.routes());
app.use(router.allowedMethods())

app.listen(8000, () => {
    console.log('koa 服务已启动')
})
````

##### 7.4、form-data、urlencoded、json 格式参数解析

````js
const Koa = require('koa');
const bodyParser = require('koa-bodyparser');
const multer = require('koa-multer');

const app = new Koa();
const upload = multer()

app.use(upload.any()) // 解析form-data
app.use(bodyParser()) // 解析json 和 urlencoded

app.use((ctx, next) => {
    console.log(ctx.request.body) // json 和 urlencoded 数据
    console.log(ctx.req.body) // koa-multer 将数据放到req.body里面了
    const { username, password } = ctx.request.body
    ctx.response.body = { username, password , age: 18 }
})

app.listen(8000, () => {
    console.log('ok')
})
````



### 八、MySQL

#### 1、基础语句

##### 1.1、显示数据库

````sql
show databases;
````

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210303094035647.png" alt="image-20210303094035647" style="zoom:50%; margin-left: 0" />

##### 1.2、创建数据库

````sql
create database <database name>;
````

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210303094615291.png" alt="image-20210303094615291" style="zoom:50%;margin-left:0"/>

##### 1.3、查看当前所使用的数据库

````sql
select database();
````

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210303094839585.png" alt="image-20210303094839585" style="zoom:50%;margin-left:0;" />

##### 1.4、使用数据库

````sql
use coderYang;
````

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210303095020423.png" alt="image-20210303095020423" style="zoom:50%;margin-left:0;" />

##### 1.5、查看当前数据库中的表

````sql
show tables;
````

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210303095150433.png" alt="image-20210303095150433" style="zoom:50%;margin-left:0;" />

##### 1.6、创建表

````sql
create table users(
	name varchar(10),
  age int,
  height double,
  password varchar(30)
);
````

<img src="/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210303095714883.png" alt="image-20210303095714883" style="zoom:50%;margin-left:0" />

##### 1.7、表插入数据

````sql
insert into users (name, age, height, password) values ('张三', 18, 1.88, 'dnjkjfa234jjl23j41fdsanjkkj')
````

![image-20210303100633445](/Users/pufeiyang/Library/Application Support/typora-user-images/image-20210303100633445.png)





























