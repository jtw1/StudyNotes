# Electron

## Electron概念及环境安装

> 概念  

Electron是由Github开发，用HTML(结构)，CSS(表现)和JavaScript(行为)来构建跨平台桌面应用程序的一个开源库，Electron通过将Chromium和Node.js合并到同一个运行时环境中，并将其打包为Mac，Windows和Linux系统下的应用来实现这一目的。通过Node它提供了通常浏览器所不能提供的能力。  

Electron有多个独立的JavaScript上下文，

- 主进程：后端进程负责启动运行应用的视窗（main进程），Electron运行package.json的main脚本的进程被称为主进程。在主进程中运行的脚本通过创建web页面来展示用户界面，一个Electron应用只有一个主进程
- 另外一个负责具体的应用视窗（render进程），它将加载应用视窗的职责委派给JavaScript代码。用户看到的web界面就是由渲染进程描绘出来的，包括html,css,js。

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

