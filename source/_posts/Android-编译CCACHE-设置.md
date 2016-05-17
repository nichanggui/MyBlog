---
title: Android 编译CCACHE 设置
date: 2016-05-03 10:33:32
tags: Android编译 CCACHE 
---

#Android 编译CCACHE 设置
**ccache**（“compiler cache”的缩写）是一个编译器缓存，该工具会高速缓存编译生成的信息，并在编译的特定部分使用高速缓存的信息， 比如头文件，这样就节省了通常使用cpp解析这些信息所需要的时间。如果某头文件中包含对其他头文件的引用，ccache会用那个文件的 cpp-parsed版本来取代include声明。ccache只是将最终的文本拷贝到文件中，使得它可以立即被编译，而不是真正去读取、理解并解释其内容。ccache是以空间换取速度，ccache非常适合经常make clean（或删除out目录）后重新编译的情况。

我们在编译Android Rom的时候会需要很大内存空间，并且每一次编译都会花费很长时间，**ccache**则是能缓存一些编译生成的信息，可以使我们在下一次编译过程中，如果文件没有改变，则直接冲**ccache**中拷贝到对应目录即可，大大的节约了编译时间。

**接下来我们设置一下编译环境下的CCACHE**
我的环境是Ubunt12.04。这个环境没关系，设置都是一样的方法。配置如下：
-  1、在~/.bashrc中添加(或者/etc/profile文件中)：
``` java
export USE_CCACHE=1
export CCACHE_DIR=/<path_of_your_choice>/.ccache
```
<!--more-->
 默认情况下cache（缓存）会保存在~/.ccache目录下，如果主目录位于NFS或其他非本地文件系统上， 设置cache目录位置：export CCACHE_DIR=< path-to-your-cache-directory >
注：配置.bashrc后注意source（source ~/.bashrc）改文件，否则cache（缓存）会保存在~/.ccache目录下，而不是你设置的目录。
- 2、使用android源码prebuilts目录下面的ccache工具初始化该文件夹

推荐的cache目录大小为50-100GB，在命令行执行以下命令：
``` java
prebuilt/linux-x86/ccache/ccache -M 50G
```
以上命令需要在你所下载的代码的根目录下面执行
     该设置会保存到CCACHE_DIR中，且该命令是长效的，不会因系统重启而失效。使用ccache第一次编译后能够明显提高make clean以后再次的编译速度。

- 3、查看ccahe使用情况
``` java
$ watch -n1 -d prebuilts/misc/linux-x86/ccache/ccache -s 
```
以上命令需要在你所下载的代码的根目录下面执行
备注:使用ccache之后,第一次编译会时间久一点,之后每次编译速度都会有提升

- 4、清除CCACHE

在使用过一段时间以后，系统的空间会受到影响，比如我的经常会报无空间，导致在编译的时候有时候会失败，内存不足，因为有时候是几个项目都编译过，运行其他软件 比如AS会比较卡顿。所以这时候就需要我们手动清理CCACHE了。

``` java
 prebuilts/misc/linux-x86/ccache/ccache -C #清除
``` 
同理在Android源码根目录执行以上命令即可。

- 5、注意事项
``` java
prebuilt/linux-x86/ccache/ccache
```
这个路径在不同太可能不太一样，可以cd+table进去查找一下即可。

