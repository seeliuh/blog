[toc]
#web技术开发桌面程序技术大纲
一直以来，都想找一个方案，能够比较简单快捷的实现跨平台的桌面程序。之前想过qt--pyqt。可惜不大喜欢，总感觉做出来的程序没有现代感，丑丑的。
直到最近遇到了atom。实在让我惊叹，web技术可以写出这么流畅的桌面程序。UI是我见过最性感的。喜欢atom，同时也让我开始关注atom相关的技术。

对于我来说，选择了解web技术开发桌面程序的原因：
- 大师说过，每个程序员都应学习javascript
- web ui技术是持续发展、活跃的。（肯定比qt活跃）
- 能兼顾学习web前台技术，对于带领复合型技术团队有帮助
- 喜欢atom，喜欢github

经过一番了解，感觉这方面还是有些混乱，牵扯不少技术
##相关技术概述
###node.js
 让javascript能操作本地功能，比如数据库、文件操作等
###webkit
用于渲染html，我理解webkit就是html、css、javascript的“解释器”
###chormium
主要是浏览器相关功能，比如cookie、窗口管理、密码管理等
>**理解WebKit和Chromium: WebKit和Chromium组**成:http://blog.csdn.net/milado_nju/article/details/7300074

###node-webkit
将node.js、webkit、chormium集成在一起的一种方案。国人开发
>用node-webkit开发多平台的桌面客户端http://www.baidufe.com/item/1fd388d6246c29c1368c.html
>使用node-webkit的项目https://github.com/nwjs/nw.js/wiki/List-of-apps-and-companies-using-nw.js
>利用node-webkit生成可执行文件http://www.cnblogs.com/2050/p/3543011.html
>Node-Webkit作者王文睿访问http://www.csdn.net/article/2014-01-08/2818066-Node-Webkit
###electron
将node.js、webkit、chormium集成在一起的的另一种方案
原名Atom-shell
没错，这是Atom的一部分。为了实现Atom，github那帮人写了这个功能，从而带领了一个生态圈。
作为Atom的粉，我必然要选择electron了
http://www.tuicool.com/articles/MrQB3aa
###npm
node.js的包管理器


##环境搭建过程记录
- 安装node.js，从官网上下载对应的包安装
- 在[官网](http://electron.atom.io)下载electron（npm方式没有成功），安装electron
- 编写hello world程序
- 运行程序，参考官网[quickstart](http://electron.atom.io/docs/latest/tutorial/quick-start/)
- 发布目标平台的程序。官方文档有文档说明。我使用了[Awesome Electron](https://github.com/sindresorhus/awesome-electron)。利用[electron-packager](https://github.com/maxogden/electron-packager)可在mac上发布linux、win32、macos三平台的程序。很方便

整个流程打通了，剩下的还需要学习html/javascript/css/

**硬伤，程序体积是硬伤。helloworld程序发布后就是100M。**
