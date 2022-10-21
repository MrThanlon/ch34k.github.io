---
title: 交叉编译wxWidgets应用
date: 2021-11-01 18:09:35
tags:
  - GUI
  - C++
  - 交叉编译
categories:
  - GUI
  - wxWidgets
---

# 引

众所周知wxWidgets是一个跨平台的GUI库，支持Linux(GTK/X11)，Windows和macOS(cocoa)，与Qt不同的是，他足够轻量，不需要魔改C++语言本身（qmake说的就是你），而且它调用平台原生API实现GUI，而Qt在Windows上实际上是绘制的界面，所以我打算用wxWidgets来做毕设。

众所周知Windows是现阶段的主流操作系统，而我用来写代码的电脑是mac，所以最终我可能仍然需要提供一个Windows版本的程序，但是我实在是不想在虚拟机上再折腾一套开发环境，所以就想着能不能直接在macOS或者Linux上编译出exe呢？答案是可以的，那就是mingw。

<!-- more -->

mingw(**Min**imalist **G**NU for **W**indows)是将GCC编译器和GNU Binutils移植到Win32平台下的产物，它是开源的软件，并且提供了macOS和Linux的版本，所以我们可以很容易的在macOS或者Linux上编译出exe。

接下来就是wxWidgets了，虽然官网提供了适用于mingw的预编译库，但是我自己测试发现不太能用，所以干脆直接从源码编译了。

# 教程

## 安装mingw-w64

首先需要安装mingw-w64，如果是macOS+Homebrew，那么可以直接brew

```shell
brew install mingw-w64
```

如果是Debian/Ubuntu，可以用apt安装，同时还要安装 `gcc-multilib`

```shell
apt-get install mingw-w64 gcc-multilib -y
```

## 下载wxWidgets源码并编译安装

建议从Github克隆仓库下来，因为我们只需要代码文件，不需要历史信息，所以可以加上 `--depth 1` 来减小仓库体积，以v3.1.5版本为例

```shell
git clone --branch v3.1.5 --depth 1 https://github.com/wxWidgets/wxWidgets.git && cd wxWidgets
```

然后我们进入 `wxWidgets` 目录，创建一个目录 `build-win64` 用于保存编译wx时产生的中间文件，以及 `build-win64/install` 存放最终的预编译库

```shell
mkdir -p build-win64/install && cd build-win64
```

我们进入了这个新创建的目录，现在需要运行wx的configure脚本，配置安装路径（`--prefix $PWD/install`）和交叉编译（`--host x86_64-w64-mingw32`，这是64位的，如果要编译32位的版本就使用`i686-w64-mingw32`），我们通常希望最后构建的exe能不需要其他依赖直接运行，所以动态库就没必要了（`--disable-shared`）。

最后的命令是

```shell
../configure --prefix $PWD/install --host x86_64-w64-mingw32 --with-msw --enable-unicode --disable-shared
```

等待configure完成，就可以执行make了

```shell
make -j4 # 4核处理器，如果你的电脑有更多核心的话建议指定更大的数字
```

我自己的mac上是跑了挺久的，不过我在我们工作室的48核服务器上编译只用了3分钟左右。

总之编译完成后就可以安装了，安装命令是

```shell
make install
```

我们之前把安装路径指定到了 `build-win64/install` ，它不是系统路径所以不需要root权限，但是需要注意的是因为没有安装到系统路径，所以像cmake这样的构建工具很可能找不到依赖，所以我们就需要用 `wx-config` 命令来生成编译参数。

## 编译wxWidgets应用

在上一步的 `build-win64` 目录中有一个 `wx-config` 程序，他和 `pkg-config` 很像，可以用来生成编译参数，比如我们的应用是 `main.cpp` ，那么可以这样

```shell
x86_64-w64-mingw32-g++ -o main.exe main.cpp -static `/path/to/wxWidgets/build-win64/wx-config --cxxflags --libs`
```

如果之前编译的32位版本那就把编译器换成`i686-w64-mingw32-g++`。

如果没有问题的话会产生 `main.exe`，他可以直接放到Windows操作系统中运行。

## 使用资源文件

你可能会发现界面很丑，是Windows95的样式，如果你想要比较现代的界面，那么需要使用资源文件，wx提供了一个 `wx.rc`，用它就好了。

首先需要编译 `wx.rc`，用 `windres` 程序，这个程序包含在binutils里，之前可能没有安装，现在我们来安装他。

如果是macOS+Homebrew，可以直接brew

```shell
brew install x86_64-elf-binutils # 64位，如果是32位的话是i686-elf-binutils
```

如果是Debian/Ubuntu，可以用apt安装

```shell
apt-get install binutils-mingw-w64 -y
```

编译rc文件也很简单

```shell
x86_64-w64-mingw32-windres /path/to/wxWidgets/include/msw/wx.rc `/path/to/wxWidgets/build-win64/wx-config --cxxflags` -O coff wx.res
```

于是我们得到了 `wx.res` 文件，现在需要把他链接到程序里，这也很容易，首先我们在编译的时候先不要链接，而是编译到.o文件（注意命令加了个 `-c` ）

```shell
x86_64-w64-mingw32-g++ -o main.o -c main.cpp `/path/to/wxWidgets/build-win64/wx-config --cxxflags`
```

然后在链接这一步加入 `wx.res`

```shell
x86_64-w64-mingw32-g++ -o main.exe main.o wx.res -static `/path/to/wxWidgets/build-win64/wx-config --libs`
```

这样得到的 `main.exe` 就是带资源文件的，有一个wx默认的图标，并且界面风格也比较21世纪了。
