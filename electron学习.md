# Electron

## Electron概念及环境安装

> 概念  

Electron是由Github开发，用HTML(结构)，CSS(表现)和JavaScript(行为)来构建跨平台桌面应用程序的一个开源库，Electron通过将Chromium和Node.js合并到同一个运行时环境中，并将其打包为Mac，Windows和Linux系统下的应用来实现这一目的。通过Node它提供了通常浏览器所不能提供的能力。  

> Node.js  

​         Node.js是一个由Ryan Dahl 在2009年创建的编程框架。它提供了一种使用JavaScript来编写服务端程序的方法，并使用基于事件的架构来处理代码的执行，该编程框架整合了V8（一种JavaScript引擎）和libuv（一种编程库），提供了异步的方式来调用操作系统资源。正因如此，通过Node.js执行的JavaScript代码之间可以做到不阻塞对方，这是和其他语言最大的不同点，在其他语言中，一行代码执行完毕后才能执行下一行代码。  

Electron有多个独立的JavaScript上下文，

- 主进程：后端进程负责启动运行应用的视窗（main进程），Electron运行package.json的main脚本的进程被称为主进程。在主进程中运行的脚本通过创建web页面来展示用户界面，一个Electron应用只有一个主进程
- 另外一个负责具体的应用视窗（render进程），它将加载应用视窗的职责委派给JavaScript代码。用户看到的web界面就是由渲染进程描绘出来的，包括html,css,js。

>  Electron源代码结构

![1611818977077](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1611818977077.png)

Electron的架构将chromium源代码和应用切分得很清楚，这样的好处是升级chromium组件很容易，同时意味着通过源代码编译Electron也很容易。  

Atom组件是由C++代码编写的，由四个部分组成

- **App**：包含了由C++和Objective-C编写的代码文件，负责处理Electron启动时的加载工作
- **Browser**：负责处理应用前端部分的交互，比如：初始化JavaScript引擎，界面的交互以及绑定针对不同操作系统的模块。
- **Renderer**：包含运行在Electron的renderer进程中的代码文件
- **Common**：包含工具类代码，这部分代码当应用运行起来以后会被main和renderer进程用到，还包括了处理将Node.js的事件循环整合进Chromium的事件循环的代码

> **Electron运行机制**

在Electron中，不论是想在前端代码中共享后端代码的状态，或者反过来，都需要经过ipcMain和ipcRenderer模块（都是事件分发器，负责在应用的后端代码(ipcMain)和前端代码(ipcRenderer)进程之间进行传递），意味着main进程中的JavaScript上下文和renderer进程中的JavaScript上下文是互相隔离的，但数据可以通过一种显式的方式在两个进程之间传递

![1611821000040](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1611821000040.png)





> 环境搭建及安装

Node.js安装成功基础上

node -v                                                                        查看版本号 (node和“-"之间有空格)

npm config set prefix "创建的node_global文件夹所在路径"           设置全局模块

npm config set cache "创建的node_cache文件夹所在路径"             设置缓存模块

npm install webpack -g                                                                            安装webpack

npm install -g electron                                                                             安装electron

npm install -g electron-packager                                                           安装打包工具

npm config set registry url(源的地址)                                                     设置npm源

npm config get registry                                                                               查看npm源

在工程目录下输入 electron .                                                                       运行工程

在packageson.json的scripts标签中添加 “pack”: "electron-packager . HelloElectron(打包的应用名) --win --out=release --arch=x64 --app-version=1.0.0 --electron-version=v11.0.4 --overwrite"      然后在当前项目根目录中输入 `npm run pack` 即可打包应用

[设置npm源的几种方式](https://cloud.tencent.com/developer/article/1588050)







## javaScript

>  基于类（Java）和基于原型（JavaScript）的对象系统的比较 

| 基于类（Java）                                               | 基于原型（JavaScript)                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 类和实例是不同的事物                                         | 所有对象均为实例                                             |
| 通过类定义来定义类；  通过构造器方法来实例化类               | 通过构造器函数来定义和创建一组对象                           |
| 通过new操作符创建单个对象                                    | 相同                                                         |
| 通过类定义来定义现存类的子类，从而构建对象的层次结构         | 指定一个对象作为原型并且与构造函数一起构建对象的层次结构     |
| 遵循类链继承属性                                             | 遵循原型链继承属性                                           |
| 类定义时指定类的所有实例的所有属性，**无法在运行时动态添加属性**。 | 构造器函数或原型指定实例的初始属性集。**允许动态的向单个对象或者整个对象集中添加或移除属性** |

> **类的概念在JavaScript中的体现**

假设一个文件 `main.js`，里面有个函数 `add(a,b)`，`divide(a,b)`，在其他文件中如果要使用这个函数，可以使用以下方式，

先在`main.js`添加以下代码

```javascript
module.exports={
    add:add,
    divide:divide
}
或者
exports.add=add;
exports.divide=divide;
```

然后在需要使用函数的文件中，通过以下方式来调用

````javascript
const main=require('(main.js所在目录)/main.js');
main.add(2,3);  
main.divide(4,2);
````

这种方式可以将代码组织成小的，可复用的代码库，方便在其他地方调用。Node.js有一条很关键的编程哲学----在开发过程中，宁愿通过组装大量小模块，也不要将所有逻辑都包含在一个大文件中。require函数不仅可用于加载本地文件，也可用于加载模块。