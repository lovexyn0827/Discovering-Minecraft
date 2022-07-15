---
typora-copy-images-to: media
---

# Minecraft启动流程与常见崩溃的解决

## 前言



## 几个版本的原版客户端运行日志分析

### 1.12.2



### 1.14.4



### 1.16.4



### 1.18.2



### 20w16a



## 源码层面的原版Minecraft运行流程

### 概览



### 客户端的启动



### 内置服务端的启动



### 客户端与内置服务端连接的建立与终止



### 内置服务端的关闭



### 客户端的正常退出



## 各环节中类的加载



## 纯服务端的运行流程



## Mod的加载

### Fabric Loader



### Forge



## 崩溃报告（Crash Report）

### 什么是崩溃报告



### 崩溃报告的位置



### 崩溃报告的结构



### 如何看懂崩溃报告



### 防止从崩溃报告中泄露隐私信息



## 常见崩溃的解决（欢迎补充）

### Java版本过低

#### 标志性现象

启动Minecraft 1.17或更新的版本时游戏直接崩溃，甚至都没有日志文件和崩溃报告，但会有类似于下面的错误信息：

```
Error: A JNI error has occurred, please check your installation and try again
Exception in thread "main" java.lang.UnsupportedClassVersionError: net/minecraft/client/main/Main has been compiled by a more recent version of the Java Runtime (class file version 60.0), this version of the Java Runtime only recognizes class file versions up to 52.0
        at java.lang.ClassLoader.defineClass1(Native Method)
        at java.lang.ClassLoader.defineClass(Unknown Source)
        at java.security.SecureClassLoader.defineClass(Unknown Source)
        at java.net.URLClassLoader.defineClass(Unknown Source)
        at java.net.URLClassLoader.access$100(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.launcher.LauncherHelper.checkAndLoadMain(Unknown Source)
```

#### 问题原因

注意上面错误信息中的这样一句话：

```
net/minecraft/client/main/Main has been compiled by a more recent version of the Java Runtime (class file version 60.0), this version of the Java Runtime only recognizes class file versions up to 52.0
```

翻译过来大概就是：Minecraft被更（四声）新的Java编译过，太新了，现在的Java认不出了，或者说不敢认了。

#### 解决方案

##### Java版本的选择

还是注意一下崩溃报告中的这句话：

```
net/minecraft/client/main/Main has been compiled by a more recent version of the Java Runtime (class file version 60.0)
```

重点注意数字60，这个数字减去44，即为Minecraft需要的Java版本，也就是Java 16。MC在变，Java在变，刻进了JVM官方标准的这一条，永远不变。

还是要给些现成的：

- 如果需要启动1.17.x，安装Java 16。
- 如果需要启动1.18.x或1.19快照（截止到20w16b），安装Java 17。
- 如果要启动更新的版本，还是建议先试一下Java17，毕竟Java17是一个LTS版本，短时间内Mojang应该不打算迁移到更新的Java平台上。

当然，直接下载最新版本的Java也是可以的，甚至还会带来性能上的优化，如果，不会出现什么奇怪的问题的话（从Java 9开始，Java不再保证完全地向前兼容）。

##### 在启动器中进行配置

然后是配置Java运行环境，不同启动器会有所不同，此处只演示两款常见的启动器。

###### 官方启动器

![](media\1650705563765.png)

![](media\1650705632897.png)

将JAVA可执行文件修改成安装的新版本Java中bin文件夹下的java.exe或javaw.exe即可。

###### HMCL

![1650705765798](L:\BOOKS\Discovering-Minecraft\Minecraft启动流程与常见崩溃的解决\media\1650705765798.png)

![1650705841212](L:\BOOKS\Discovering-Minecraft\Minecraft启动流程与常见崩溃的解决\media\1650705841212.png)

也是将JAVA可执行文件修改成安装的新版本Java中bin文件夹下的java.exe或javaw.exe即可。

### 内存溢出

#### 标志性现象

可以是突然弹出一个画面：

![](media\OIP-C.jpg)

也有可能发生崩溃，有崩溃报告产生：

```
---- Minecraft Crash Report ----
// You're mean.

Time: 2020/8/13 下午7:43
Description: Exception in server tick loop

java.lang.OutOfMemoryError: Java heap space
	at java.base/java.util.stream.Nodes$IntArrayNode.<init>(Nodes.java:1327)
	at java.base/java.util.stream.Nodes$IntFixedNodeBuilder.<init>(Nodes.java:1489)
	at java.base/java.util.stream.Nodes.intBuilder(Nodes.java:214)
	略略略...
```

而且有这样一个关键词：***`OutOfMemoryError`***

比较罕见的情况下则连通常的崩溃报告都没有，只在游戏文件夹生成一个名称类似于`hs_err_pid2084.log`的文件（JVM的崩溃报告），而且开头类似于这样：

```
#
# There is insufficient memory for the Java Runtime Environment to continue.
# Native memory allocation (mmap) failed to map 14680064 bytes for G1 virtual space
# Possible reasons:
#   The system is out of physical RAM or swap space
#   The process is running with CompressedOops enabled, and the Java Heap may be blocking the growth of the native heap
# Possible solutions:
#   Reduce memory load on the system
#   Increase physical memory or swap space
#   Check if swap backing store is full
#   Decrease Java heap size (-Xmx/-Xms)
#   Decrease number of Java threads
#   Decrease Java thread stack sizes (-Xss)
#   Set larger code cache with -XX:ReservedCodeCacheSize=
#   JVM is running with Unscaled Compressed Oops mode in which the Java heap is
#     placed in the first 4GB address space. The Java Heap base address is the
#     maximum limit for the native heap growth. Please use -XX:HeapBaseMinAddress
#     to set the Java Heap base and to place the Java Heap above 4GB virtual address.
# This output file may be truncated or incomplete.
#
#  Out of Memory Error (os_windows.cpp:3466), pid=2084, tid=5852
#
# JRE version: Java(TM) SE Runtime Environment (16.0.1+9) (build 16.0.1+9-24)
# Java VM: Java HotSpot(TM) 64-Bit Server VM (16.0.1+9-24, mixed mode, sharing, tiered, compressed oops, compressed class ptrs, g1 gc, windows-amd64)
# No core dump will be written. Minidumps are not enabled by default on client versions of Windows
```

（好吧，如果英语过关能看懂这个就OK了，这个比下面说的还得全面。[笑哭]）

经常是一阵一阵地卡，卡着卡着就崩了

#### 问题原因

物理内存不足或为Java分配的内存不足，那种“一阵一阵地卡”就是内存不足时频繁进行GC的结果。

也有可能有内存泄漏导致一些永远用不到的对象被错误地保留作为“助（主）攻（力）”。

#### 解决方案

可以先试着Minecraft多分配一些内存。

HMCL可以直接在配置界面操作，很简单，不再细说。

官方启动器似乎只能通过编辑JVM参数进行调节，可以在配置菜单中“更多选项”找到。

![1650707079023](L:\BOOKS\Discovering-Minecraft\Minecraft启动流程与常见崩溃的解决\media\1650707079023.png)

修改-Xmx后面的内存大小即可。

如果系统内存本来就不多，则不建议这样做，有时分配的内存越多反而越卡。这时，推荐再加一个内存条而不是配置更多的虚拟内存。

### LWJGL本地库链接异常

#### 标志性现象

Minecraft窗口出现前崩溃：

```
---- Minecraft Crash Report ----
// Uh... Did I do that?

Time: 22-4-21 上午11:43
Description: Initializing game

java.lang.UnsatisfiedLinkError: Failed to locate library: lwjgl.dll
	at org.lwjgl.system.Library.loadSystem(Library.java:147)
	at org.lwjgl.system.Library.loadSystem(Library.java:67)
	at org.lwjgl.system.Library.<clinit>(Library.java:50)
	at org.lwjgl.system.MemoryUtil.<clinit>(MemoryUtil.java:97)
	at org.lwjgl.system.Pointer$Default.<clinit>(Pointer.java:61)
	at com.mojang.blaze3d.platform.Window.checkGlfwError(SourceFile)
	at com.mojang.blaze3d.platform.GLX._initGlfw(SourceFile)
	at com.mojang.blaze3d.systems.RenderSystem.initBackendSystem(SourceFile)
	at net.minecraft.client.Minecraft.<init>(SourceFile)
	at net.minecraft.client.main.Main.main(SourceFile)
```

正常日志输出的最后一行大概率是这样：

```
[12:45:03] [Render thread/INFO]: Backend library: LWJGL version 3.2.1 build 12
```

#### 问题原因



#### 解决方案



### JVM崩溃（并非全部适用）

#### 标志性现象



#### 问题原因



#### 解决方案



### Mod导致的崩溃

#### 标志性现象



#### 问题原因



#### 解决方案



### Minecraft本体导致的崩溃

#### 标志性现象



#### 问题原因



#### 解决方案



## 推荐Mod: Not Enough Crashes