# 龙芯Fedora28配置Jetbrains全家桶并解决兼容性问题

2020-12-11

## 缘起

尽管Jetbrains全家桶是纯Java开发的，但是其中包含的一些插件是以二进制的形式发布的，且从2020.2开始，其依赖的Java环境最低只支持openjdk11。

这里均以2019.3版为准。

编译好的libs： [传送门](Jetbrain-libs.tar.bz2)

## 共性的问题

主要问题出现在 ``fsnotifier`` 和 ``pty4j-native`` 两个插件上，他们均是以二进制的形式出现在 ``lib`` 目录下的。一般解决了这两个问题就可以正常使用了，一些我遇到的问题的不完全的解决办法附在之后。

+ fsnotifier

下载[该页面](https://github.com/JetBrains/intellij-community/blob/master/native/fsNotifier/linux/) 源码并运行 ``./make.sh`` 编译，获得 ``fsnotifier-mips64`` ，复制到 ``/bin`` 目录下（和自带的fsnotifier和fsnotifier64在通个目录）。并在clion配置文件夹 ``config`` 目录下新建 ``idea.properties`` 文件；或在菜单选择 ``Help->Edit Custom Properties`` 输入以下内容：


```
# custom CLion properties

idea.filewatcher.executable.path = fsnotifier-mips64
```

具体参考官方[链接](https://confluence.jetbrains.com/display/IDEADEV/Compiling+File+Watcher)

+ pty4j-native

问题主要体现在无法打开终端。

clone整个项目并编译：

```sh
https://github.com/JetBrains/pty4j.git
cd pty4j/native
make
```

编译获得的链接库在 ``os/linux/mips64el`` 中，复制它到 ``lib/pty4j-native/linux/x86_64`` 中覆盖即可。


## clion

+ tool-chain

由于无法使用build in的编译器，在设置中设置系统中的编译器即可。

```sh
sudo dnf install cmake gcc gcc-g++
```

+ clang-tiny

我没有成功编译clangd和clang-tiny，试图使用系统的clangd但是没有找到设置的方法。只能设置clang-tiny和使用内建代码补全。这里需要安装clang。

```sh
sudo dnf install clang
```

在 ``File->Settings->Language & Frameworks->c/c++`` 中设置Clangd关闭，设置Clang-Tiny为使用外部的Clang-Tiny，路径为 ``/usr/bin/clang-tidy`` 。

具体参考官方[链接](https://www.jetbrains.com/help/clion/settings-languages-cpp-clangd.html)，目测并不十分适合老版本。

## pycharm

配置如clion一节中fsnotifier和pty4j-native两部分，其他无需过多配置。

Pycharm似乎不会创建桌面链接，这里附之。是使用clion的文件改的。 ``Icon`` 和 ``Exec`` 段按照现实目录修改。

```sh
#jetbrains-pycharm.desktop
[Desktop Entry]
Version=1.0
Type=Application
Name=Pycharm
Icon=/home/loongson/.Jetbrains/pycharm-2019.3.5.edit/bin/pycharm.svg
Exec="/home/loongson/.Jetbrains/pycharm-2019.3.5.edit/bin/pycharm.sh" %f
Comment=A cross-platform IDE for python
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-pycharm
```

## idea

配置如clion一节中fsnotifier和pty4j-native两部分，其他无需过多配置。

## rider

+ 外部链接

[龙芯.NET](http://www.loongnix.org/index.php/Dotnet)
[.NET Core 3.1下载页面](http://www.loongnix.org/index.php/Dotnetcore31)
[.NET Core安装说明](http://www.loongnix.org/index.php/Install-dotnet-core)

+ 环境变量

在 ``~/.bash_profile`` 中添加：

```sh
export DOTNET_ROOT=$HOME/dotnet
export PATH=$PATH:$HOME/dotnet
```




todo:

what is libdbm64.so