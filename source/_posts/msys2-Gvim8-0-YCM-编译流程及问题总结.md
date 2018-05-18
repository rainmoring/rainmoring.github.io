---
title: msys2+Gvim8.0+YCM 编译流程及问题总结
date: 2018-05-18 09:35:06
tags:
    msy2
    ycm
    gvim

categories:
    work

---

{% blockquote 王家卫,Ashed of Time %}
如果感情是可以分胜负的话，我不知道她是否会赢。但是我很清楚，从一开始，我就输了。
{% endblockquote%}

#前言

本篇文章是我win7 32位系统下配置gvim的YCM的一些经验，主要参考的是[这篇博客]( https://www.cnblogs.com/tracyone/p/4735411.html),基本的流程和这篇博客是一致的，但是也遇到了一些问题，在这里做一个总结，给各位做一个参考，也给自己做一个记录，因为之前弄过一次，但是已经忘记了，导致这一次又弄了好久才好。

#编译流程

LLVM是之前就安装就好的，这里不再说。

##msys2安装

我们使用的环境是msys2，是因为这个shell还不错，拥有很全的linux的工具链。msys2和Cygwin和MinGW的关系很多地方都说过了，前面的链接博客中也有讲到，这里不再多说，可以在网上找找相关文章加深理解。

首先从[msys2](http://www.msys2.org/)下载需要的安装版本，我这里是 `32位系统`，选择`i686`版下载安装,我的安装位置和官网说明一样，都在`C:msys32`中,下载以后更新源，在`C:\msys32\etc\pacman.d`中有三个文件mirrorlist.msys, mirrorlist.mingw32, mirrorlist.mingw64,分别对应安装目录下的msys2.exe，mingw32.exe，mingw64.exe这三个执行文件，后两个很明确，对应32位和64位的操作系统，第一个是什么区别还不是很明确。分别将他们的源换为

mirrorlist.msys：
##
## MSYS2 repository mirrorlist
##

## Primary
## msys2.org
## Server = http://repo.msys2.org/msys/$arch
## Server = http://downloads.sourceforge.net/project/msys2/REPOS/MSYS2/$arch
## Server = http://www2.futureware.at/~nickoe/msys2-mirror/msys/$arch/

Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/$arch
Server = http://mirrors.ustc.edu.cn/msys2/msys/$arch/

mirrorlist.mingw32:
##
## 32-bit Mingw-w64 repository mirrorlist
##

## Primary
## msys2.org
## Server = http://repo.msys2.org/mingw/i686
## Server = http://downloads.sourceforge.net/project/msys2/REPOS/MINGW/i686
## Server = http://www2.futureware.at/~nickoe/msys2-mirror/i686/

Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/i686
Server = http://www2.futureware.at/~nickoe/msys2-mirror/i686/

mirrorlist.mingw64:
##
## 64-bit Mingw-w64 repository mirrorlist
##

## Primary
## msys2.org
## Server = http://repo.msys2.org/mingw/x86_64
## Server = http://downloads.sourceforge.net/project/msys2/REPOS/MINGW/x86_64
## Server = http://www2.futureware.at/~nickoe/msys2-mirror/x86_64/

Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64
Server = http://mirrors.ustc.edu.cn/msys2/mingw/x86_64/

这些源有一个就够用，我这里设置了两个。
然后打开对应的shell，我打开的是mingw32.exe,64位的系统打开mingw64.exe就好，然后使用
pacman -Sy更新源，再然后安装基本的工具
pacman -S --needed filesystem msys2-runtime bash libreadline libiconv libarchive libgpgme libcurl pacman ncurses libintl
如果这个过程中有哪个包没有下载成功，再次下载一下就好，我出现过网速突然断掉导致某个包下载失败的情况。如果是其他情况，我没有遇到过。
注意升级完了以后因为包含pacman的升级，所以这时候源的信息又被覆盖了，重新在修改一次。
然后在安装
pacman -S base-devel
pacman -S msys/msys2-runtime-devel
pacman -S msys2-runtime-2.10.0-2
不要安装msys2的工具链，我也不懂这个工具链是多少位的，为了避免干扰，就不要装了，反正32位的系统，装了估计也没卵用。
上面这个操作都执行一遍，我也不知道哪个是必须的，反正没有坏处。因为我之前遇到过编译的时候没有select.h等很多头文件的问题，用everything 整个设备都找不到这个文件，我试着从外部一个一个拷进来，但是一来文件很多，二来不断报错，最后胡乱的安装包，删文件，导致最后有个什么GPM的校验失败，链pacman都不能在使用，只好重新安装工具链。后来发现在安装这个包的时候，在C:\msys32\usr\include的目录下生成了需要的select.h等等各种各样的头文件，折腾了很久
pacman -S mingw-w64-i686-toolchain
这里面没有cmake需要额外安装
pacman -S mingw32/mingw-w64-i686-cmake
如果要使用cmake-gui，还要qt的库，很麻烦，而且很大，我试过直接在系统上安装一个cmake，就可以使用cmake-gui,但是自己才疏学浅，不太会用，但是在我执行的过程中给我了启发，后面可以再说。
然后可以安装python3.6,也可以安装python2.7,也可以都安装，你怎么高兴怎么来就好，但是后面会讲到这其中的玄机。
至此msys2的准备工作完成，至于加环境变量这些，不是很重要。如果不想外部使用，就不加了，因为我本地安装了python3，加了反而导致和我本地的工具链搞混了。

下面开始安装gvim安装包，我装的是官方预编译好的gvim8.0，我懒得再去折腾，因为官网编译的我觉得就挺好的，python2，python3都支持了，但是区别是官方支持的方式是dyn/python3,dyn/python2,就是说在用的需要动态指定python的信息。另外一个遇到的问题就是我本地的环境中只能安装到c盘中，安装到d盘以后打不开，不知道是不是和单位的设置有关系，同时我在安装目录里面将gvim.exe 名字为vim.exe，vim.exe改为gvim.exe,这样打开的时候就可以直接打开gvim了。安装好以后在家目录下面，也就是C:\Users\willgu下面就有了.vimrc文件，我使用的是spf-13,我将之前下载好的.vim拷进来放在这里就可以使用插件了，整体思路和在ubuntu上一样。

至此准备工作完成了，开始编译YCM
首先要安装LLVM,如果没有就自己先安装一下。最好安装在没有空格的路径下面，不用麻烦的去输入路径空格的转移。
然后进入.vim/bundle下面
git clone https://github.com/Valloric/YouCompleteMe.git 
cd YouCompleteMe 
git submodule update --init --recursive
mkdir build ;cd build

cmake -G "MSYS Makefiles" -DCMAKE_MAKE_PROGRAM=C:/msys32/mingw32/bin/mingw32-make.exe  -DUSE_PYTHON2=OFF -DPYTHON_LIBRARY=D:/python36/python36.dll -DPYTHON_INCLUDE_DIR=D:/python36/include -DPATH_TO_LLVM_ROOT=C:/LLVM . c:/Users/willgu/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp
编译完成。
cmake -G 是生成makefile, 有很多可以选，我们使用的是msys2，就生成MSYS Makefiles，其他的工具链就选择其他的，
-DCMAKE_MAKE_PROGRAM 指定使用的make命令，我们安装的是32位的就是用32位的工具链，如果是64位的选择64对应的make工具就OK，如果不填这个会报错。
 -DUSE_PYTHON2=OFF 是说不要使用python2，如果你使用的python2,就不要这个选项了，因为我使用的python3.6
 ,一开始没有加编译时老是报错，网上找了一下就发现是编译环境使用了python2，但是python2是没有安装了，所以也会提示找不到PYTHON_LIBRARY 和DPYTHON_INCLUDE_DIR这两个错，这时候看网上的说明，有人使用使用cmake-gui的时候发现可以通过去掉python2的√选实现不适用python2，这给我了启发，在网上就找怎么在cmake关闭python2，于是找到了-DUSE_PYTHON2这个选项
-DPYTHON_LIBRARY 这个说的是指定对应的python库，注意一定是要给到.dll结尾，不要只给到目录，因为这个疏忽耽搁了我小半天，找来找去找不到问题。
 -DPYTHON_INCLUDE_DIR 是指的包含的头文件，指定到include目录就ok了
注意：
如果不指定DPYTHON_LIBRARY和DPYTHON_INCLUDE_DIR这两个库也是可以编译过得，这是因为在我的msys2中安装了python3，编译器找到了msys2中的python3而不是我本地的python3。但是编译通过以后打开vim却提示python运行失败，虽然我将msys2中python3的路径加入了win7的path，但是并没有卵用，所以我直接使用系统安装的python3,也才会说path什么的加不加无所谓。
-DPATH_TO_LLVM_ROOT是值得llvm的路径，不用多说
接下来两个路径，.代表的是当前build结果路径，/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp是目标路径
在这个过程中可能出现报错
math.h:91:12:error:'std::hypot' has not been declared
，这时候找到你的python安装路径中的python.h文件，在开头加上
#include "math.h"
mingw32-make ycm_core

搞定，
不知道为什么在golang下面使用YCM的时候，整个系统卡卡的，蛋疼。后面再看看为什么，有可能是因为spf-13太沉重了
