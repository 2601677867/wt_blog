---
title: PS1游戏汉化研究——STR视频文件
date: 2022-01-14 16:01:05
categories: PlayStaion汉化研究
cover: https://s2.loli.net/2022/01/14/f3Nvj9omQrKGnx7.png
tags: [ps, ps1, ps1游戏汉化教程, 教程, 游戏汉化]
---
# 前言

此教程将会带你一步步修改PlayStaion 1游戏中的str视频文件并了解相关知识。

<!-- more -->

我知道你不想听我分析str或者avi是如何让PSX的CPU编码并显示画面的，讲这些又要涉及到一大堆计算机图像学知识，yuv灰度图像rgb图像二进制巴拉巴拉的。

我并不懂什么图像学知识，虽然我以后可能会涉及OpenCV领域，但那都是后话。我所有的教程都是"精讲"——这意味着我将跳过我枯燥乏味的太深奥的学术研究过程，只保留最精彩和你需要的部分。

你能搜到这篇文章大概率是在尝试给PlayStaion上的游戏做汉化{% psw 没有人会在2022年尝试将电影刻录进PSX播放 %}且需要给游戏CG做字幕。

那么这篇"笔记"性质的"教程"将会教你如何从游戏光盘内{% mark 正确的 color:red %}导出至avi格式，通过现代剪辑软件编辑后再转回str。

# 认识PSX中的视频文件

PlayStaion 1的游戏CG普遍使用{% kbd .str %}和{% kbd .mov %}两种视频格式，也有些游戏使用自己自制的格式来压缩视频资源，但psx支持的解码器也就那几种，mpeg2与mpeg总有一个能搞定。有些游戏则会在现有的str和mov之上进行修改，例如《Final Fantasy VI 最终幻想6》

FF6由于在str文件的基础上进行了一些修改，导致转码和再编辑非常困难。此教程只适用于标准str文件，如果你汉化的游戏使用了特殊规则，那只能另辟蹊径了！

jPSXdec的作者m35曾对STR文件进行过深度的研究，或许这篇文章可以帮助你处理特殊STR：[The PlayStation 1 Video (STR) Format](https://github.com/m35/jpsxdec/blob/readme/jpsxdec/PlayStation1_STR_format.txt)

# 前置工作

## 正确提取文件

### 扫整不扫单原则

在2022年，我们使用更现代化的工具[jPSXdec](https://github.com/m35/jpsxdec)来提取psx游戏文件

虽然工具有点英文基础就能使用明白，但当我在提取资源时却犯了一个致命的错误。

得力于现代解压缩软件，大部分都可以直接解压psx游戏最常见的两种格式——iso或bin文件。但这样是完全だめ的！直接解压缩整个光盘文件后再提取str会导致jpsxdec报错：
{% noteblock child:codeblock color:red %}

disc image does not contain raw header - audio may not be detected

{% endnoteblock %}
且会出现str文件丢失音频、扫描不到非bin格式压缩下的附加等各种各样的奇葩问题。

{% image https://s2.loli.net/2022/01/14/LiOqxI7V1rACM8w.png 直接扫描某个bin文件 %}

{% image https://s2.loli.net/2022/01/14/WoklFRhign3pI1X.png 扫描光盘文件 %}

事实证明jpsxdec可以扫描出来Tim Collection无法搜索到被压缩的贴图。

### 导出注意事项

使用jpsxdec导出str时，jpd会自动整合xa与str。但需要注意的是导出选项需要选择AVI——Uncompressed RGB 否则后期mc32和movconv会报错

至于解码质量，以最大视觉效果为优，选择Emulate PSX quality

{% image https://s2.loli.net/2022/01/14/FQeBVHtSImKUNlo.png  %}

关于导出选项，如果你有更好的组合或其他改进建议，请在评论区内comment

### 工具

- [Psy-Q SDK](http://www.psxdev.net/help/psyq_install.html)

Psy-Q在我查阅资料后发现，这个SDK并非出自索尼，而是一个叫SN System的公司所开发。该公司从N64到GBC，为多个游戏机平台开发了拥有offical授权的Windows SDK

该工具不适用于现代操作系统，仅适用于Windows 3.1/95/98 {% psw 真的会有人拿Windows3.1这东西去开发psx游戏吗？我觉得它日常使用都费劲 %}

为什么不用索尼官方Sony Development Tools Collection ？因为网上大部分大佬写的psx程序都是用psy-q写的

- [MC32/MovConv](http://www.psxdev.net/forum/viewtopic.php?f=69&t=957)

实际上MovConv和PSY-Q下的mc32经我测试都是一个东西，如果你的avi格式符合要求，任意一个工具都可以转换成功。

如果上面的mc32最新版不适合你，你可以使用[movconv](https://winningeleven-games.com/showthread.php?tid=8612)

movconv仅在Windows 3.1/95/98上可用！

- [strplay](http://www.psxdev.net/forum/viewtopic.php?f=64&t=507&p=3645#p3645)

一个简单的ps1播放器程序，可以将你生成后的文件拖进iso里测试下游戏外是否可以正常播放。

代码简单到我这个c白痴都能看懂，只需要修改strtest.c即可

- [Windows 98 Vmware](https://winworldpc.com/download/3dc3943c-c386-18c3-9a11-c3a4e284a2ef)

宁不可能自己去先造台符合Windows 98规格硬件的PC，或者将自己的主力机改成98。

使用虚拟机是最棒的选择，VMware对98兼容很好，但95的体验就很拉。

- FFmpeg 与 格式工厂

自己去百度搜

- VirtualDub

上古年代的视频处理软件，好就好在只能处理avi，稍后我会介绍为什么需要它。

# 视频互转——STR

转出来容易转回去难，mc32和movconv对avi格式要求非常严格，有一项不符合工具也会报错。

大概陆陆续续尝试转换了一个多星期均失败后，施科昱大佬一封邮件点醒了我。

首先必须确保avi文件为Uncompressed RGB，也就是未压缩RGB格式，对于纯修改从jpsxdec提取的avi来说基本不用执行什么转换操作。

但如果是在现代操作系统上自制的avi文件或翻译后格式编码遭到修改，那么你需要进行转换处理。

```powershell
ffmpeg -i <infile> -vcodec rawvideo -s 320x240 -r 30 -an <outfile>
```
psx那可怜的芯片最多支持320x240分辨率的视频，最大fps为30fps，如果是从jpsxdec导出的默认15就好。

如果转换后mc32会变得不认识此avi文件，解决办法便是使用VirtualDub将ffmpeg转换后的avi再导出一遍。

文件———另存为旧版AVI文件({% kbd Shift %} + {% kbd T %})

这样mc32和movconv便可顺利识别avi文件了。

{% image https://s2.loli.net/2022/01/14/j2P1dxEfDeSbMcG.png 一个多星期的研究成果 %}

不确定是何种原因造成这个现象，起初我认为是这段ffmepg命令有问题，但在discord上许多人成功转换了文件，同时也没有任何document来解析VirtualDub中另存为旧版AVI的"旧版"到底旧了什么。

我没有尝试转换过其他游戏的cg，或许只有《生化危机:枪下游魂》有这个问题。如果你汉化的游戏也遇到了这种情况，不妨再转换下试试。

{% image https://s2.loli.net/2022/01/14/LlZEorUmsxR7S8z.png 在简易播放器中完美播放 %}

# 音频互转——XA

如果你所汉化的游戏音频与str文件是合并在一起的，那么你可能需要将音频与汉化后视频文件重新合并下。

PlayStaion有两种常见的音频格式，XA与VGA，VGA根据jpsxdec说明书解释为：spu直接处理的特殊的xa音频文件，一般str都是与xa合并在一起，与vga合并的很少很少。


## 对MDEC编码器的设置

像《生化危机：枪下游魂》这种视频与音频分开存放的不必考虑avi转出str时mdec编码器的设置，但要是准备插入音频就要更改一下文件扇区了。

MovConv不具备更改MDEC编码器的功能，请使用Psy-Q SDK下的mc32

{% image https://s2.loli.net/2022/01/20/YKjtD7U3x8pO5zq.png 我忘记在哪个选项卡里可以找到这个窗口了 %}

打开MDEC Parameters窗口，勾选Custom，如果你的视频为30fps就使用4 sectors，如果是15fps就使用8 sectors

## 处理wav

又是一个未知大坑，你可以从原视频文件内直接提取wav，但确保音频为14位且低于44100hz，接着拖到mc32里转换成xa，如果转换后的文件大小不正确或者转换失败的话，请使用ffmepg从视频中提取wav。

```
ffmpeg -i <原视频文件.avi> -acodec pcm_s16le -vn <outfile.wav>

```

## 合并

现在你应该获得了一个str文件和xa文件，打开MC32中的Video+Sound窗口，设置好源文件与目标文件后，仅修改Frame Rate和Number of channels (Data Rate)两个选项卡即可。

{% image https://s2.loli.net/2022/01/20/K6pMGw8JBqxuXRz.png 摸索摸索摸索.... %}

经我测试30帧的视频使用30，1(150 sectors/sec)这个组合可以成功合并文件。

15帧的视频我没有测试，请自行摸索组合。

# 总结

读到这里你就已经学会简单的str处理了，特殊的str{% psw 比如三个视频文件压在一个str里，我称它为三相之力！ %}后续我会发文研究。

合并后的str使用mc32里的预览功能显示已经可以正常播放，但使用strplay似乎会黑屏，不知道为什么。可能是播放器代码的问题，有时间我会修改下并更新文章。

{% image https://s2.loli.net/2022/01/20/YRHciPrqLDExJWX.png 跨平台诈骗！ %}

另外推荐一个面向psx开发者的模拟器————[PCSX-Reduxe](https://pcsx-redux.consoledev.net/)