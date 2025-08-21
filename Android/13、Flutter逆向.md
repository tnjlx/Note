# 1. Flutter

Flutter是Google构建在开源的Dart VM之上，使用Dart语言开发的移动应用开发框架，可以帮助开发者使用一套Dart代码就能快速在移动iOS、Android上构建高质量的原生用户界面，同时还支持开发Web和桌面应用。Flutter引擎由C/C++编写，它实现了Flutter的核心库，包括动画和图形、文件和网络I/O、辅助功能支持、插件架构，以及用于开发、编译和运行Flutter应用程序的Dart运行时和工具链。引擎将底层C++代码包装成Dart代码，通过dart:ui暴露给Flutter框架层。

# 2. Flutter特征

在assets文件夹下会有dexopt和flutter_assets两个文件夹，lib文件夹会有两个so文件：libapp.so和libflutter.so（flutter动态链接库，与实际业务代码无关）。

# 3. Flutter抓包

* Dart语言标准库的网络请求不走Wi-Fi代理
* Dart SDK在Android平台上强制只信任系统目录下的证书（/system/etc/security/cacerts），通过Dart源码中的runtime/bin/security_context_linux.cc文件实现

## 3.1. 对抗库

* 手动Hook，参考：https://www.52pojie.cn/thread-1951619-1-1.html
* reflutter
* Reqable或者proxyPin

# 4. Flutter反编译

参考：https://www.52pojie.cn/thread-1951619-1-1.html

* 快照
* 静态：解析libapp.so
* 动态：修改libflutter.so并重打包
* blutter
