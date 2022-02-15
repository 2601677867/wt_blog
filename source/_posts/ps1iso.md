---
title: PS1游戏汉化研究——iso游戏光盘修改
date: 2022-01-25 19:05:40
categories: PlayStaion汉化研究
cover: https://s2.loli.net/2022/01/14/f3Nvj9omQrKGnx7.png
tags: [ps, ps1, ps1游戏汉化教程, 教程, 游戏汉化]
---

当你想把修改好的游戏文件写回游戏光盘内实机运行看下效果，你不能简单的在软碟通里粘贴替换文件就完事了。

<!-- more -->

# ISO-9660与验错码

PlayStaion使用CD-ROM媒介来储存游戏文件，光盘自然也使用了ISO-9660(CDFS)文件系统标准。

这个文件系统储存文件时会把文件划分成三类：CD-ROM Mode 1，CD-ROM Mode 2 Form 1与Form 2

PS使用了Mode 1储存游戏程序和数据文件，Mode 2 Form 2来储存音频与视频等游戏资源。两者都有纠错码与验错码来防止数据异常导致执行文件错误，所以不做任何处理直接复制粘贴的文件必会被验错码检测到。

如果修改后的文件大小等于或小于源文件大小，纠错码会纠错一部分字节来让光盘正常运行。但如果大于源文件大小，文件扇区分配被更改的情况下纠错码也无能为力，这时我们就需要重新构建整个游戏光盘。

# 重新构建

## psximager 工具

施珂昱大佬在教程中提到过EMU-ZONE的汉化论坛里有人曾分享了一篇重构PS游戏光盘文件的方法，但可惜EMU-ZONE网站早已关站。

不过我们有更为现代化的开源工具：psximager

{% ghcard cebix/psximager theme:dark %}

但是工具需要自己用mingw或者gcc编译一下，如果你懒得自己手动编译，可以下载这个已经被编译过适用于Windows平台的旧版:[下载链接](https://github.com/paulsapps/psximager/releases/download/v2.0_bins/psximager.zip)

Mac OS或者Linux发行版的同学请自行编译吧！

需要注意的是src文件夹下还有三个工具需要单独编译，最后需要自行手动添加环境变量。

## 工具说明

psximager有三个功能

### psxrip 解包并生成目录树文件

psxrip可以从原版光盘文件(BIN/CUE)中以正确的方式解包并生成文件目录树用于重新构建或注入修改版文件。

生成的目录树为{% kbd .cat %}文件并附带一个系统数据源的{% kbd .sys %}文件{% psw 其实这数据源就是索尼的PS许可信息文件 %}。

```
psxrip [OPTION...] <input>.bin/cue <output_dir>

 -l, --lbns 将所有文件和目录起始扇区号一起写入目录树文件内
 -t, --lbn-table 打印LBN表
 -v, --verbose 详细
 -V, --version 显示版本信息并退出
 -?, --help 显示此帮助消息

```

示例：我想解包《生化危机:枪下游魂》原版游戏光盘文件bioharzard-gunsurvival.bin到regs文件夹内，并生成bioharzard-gunsurvival.cat目录树文件

```
psxrip bioharzard-gunsurvival.bin /regs

```

### psxinject 不重建替换文件

前面说过，纠错码的能力是有限的。修改后文件不允许使用更多扇区既大于原版文件大小。但允许小于或等于。

```
psxinject [OPTION...] <input>.bin/cue <repl_file_path> <new_file>

  -v, --verbose 详细
  -V, --version 显示版本信息并退出
  -?, --help 显示此帮助消息
 
```

示例：替换《生化危机:枪下游魂》原版游戏光盘文件bioharzard-gunsurvival.bin中的CAPCOM30.STR

```
psxinject bioharzard-gunsurvival.bin CAPCOM30.STR C:\psxproject\str\CAPCOM30.STR

```

### psxbuild 重构光盘文件

根据cat目录树创建新的游戏光盘文件

```
psxbuild [OPTION...] <input>.cat <output>.bin

  -c, --cuefile 创建.cue文件
  -v, --详细 详细
  -V, --version 显示版本信息并退出
  -?, --help 显示此帮助消息
  
```
如果你想要向游戏光盘内添加原版镜像没有的新文件，那么你需要手动修改下目录树。因为我的汉化工作仅限于修改不涉及添加，所以还请有需求的各位自行阅读psximager的readme.md目录树部分。

# 总结

一套流程下来就是：先rip生成目录树———把文件替换好后在psxbuild。

有些游戏对CG或者图片大小没有做限定要求，因此可以整一些奇奇怪怪的活来玩。
