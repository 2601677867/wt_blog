---
title: PS1游戏汉化研究——TIM图片文件
date: 2022-01-30 15:15:41
categories: PlayStaion汉化研究
cover: https://s2.loli.net/2022/01/14/f3Nvj9omQrKGnx7.png
tags: [ps, ps1, ps1游戏汉化教程, 教程, 游戏汉化]
---

也包括怎么从压缩文件内替换里面的TIM文件。

<!-- more -->

# TIM文件

TIM文件是PlayStaion游戏场景的贴图/图片文件格式，你问我为什么叫TIM？我也不知道

TIM文件的特色是由多个颜色表(CLUT)控制内容，比如说绿色颜色表显示1-6，蓝色颜色表可能显示11-15、或者两种颜色表显示同样的内容。

但是颜色表控制内容是TIM文件独有的特性，最后一版支持直接编辑TIM的插件还是Photoshop 2.0 (1994年)版本。相信我，你不会想用Windows 3.1去处理字库图片的。

所以对于多颜色表控制图片，我们只能将每个状态的图片分别导出，处理后再使用工具合并，有点类似于STR和XA。

但这只限于一些什么字体等特殊贴图，对于一些常规单颜色表图片的话直接转成bmp拖进Photoshop里再转回来就行了！

# 工具

- [jpsxdec](https://github.com/m35/jpsxdec)

与前几次不同，这次我们要以命令行的方式来使用jpsxdec

- [img2tim](http://lameguy64.github.io/img2tim/img2tim_(v0.75).zip)

此命令行工具几乎允许你将任意格式的图片转换成TIM，并允许调整一些高级参数。

{% psw Lameguy64 永远滴神 %}。

- [TIM2View](https://github.com/lab313ru/tim2view)

具有图形界面工具允许你进行TIM合并操作

# 常规图片修改流程(单颜色表)

其实jpsxdec自带了媒体文件替换命令，可以在不解包和重构光盘的情况下实现对游戏文件的修改。

但是我为什么不在STR文件那篇文章中使用这个方案呢？

很简单，因为一旦某个文件替换过程出错，整个光盘都会崩掉！

宁不可能每替换一个文件就把200多m的iso备份一遍吧？

但是贴图文件就不一样了，大部分贴图文件都是被游戏厂商封装在自己制作规则的压缩文件内。

比如说ITEM.BIN，HOMO.DAT等。如果你和冬寂一样，逆向工程经验小于等于0，就不要在尝试编写工具来解包封包了。

好在jpsxdec不仅可以扫描出来每个文件内是否存在TIM图片，还提供了一个替换功能，只不过这个功能只存在于命令行模式中。

## 导出

导出真没啥好说的，我推荐导出为bmp，在Photoshop处理完后再导出为bmp，防止颜色表之类的数据错乱。

## 转换

在此之前需要先了解一件事情，PlayStaion的VRAM(显存)是可以被抽象成一个平面空间，一张4x4大小的贴图可以被放在任意位置。如果将VRAM左下角定位起始坐标原点(0，0)并建立坐标系，那么贴图可以放在(12，36)或(114514,19191810)等位置。

当游戏需要某张图片时，游戏就会读取这张图片的CLUT，数据和坐标等信息。随后这张图片就会被加载到VRAM坐标轴内的指定位置供程序使用。

那么在你生成好TIM文件并与源文件各种格式保持一致后，唯一需要修改的就是显示坐标点了。

这也解释了，为什么我之前尝试用winhex暴力替换压缩文件内的图片，在游戏中会显示其他图片，而不是崩溃的情况。

{% image https://s2.loli.net/2022/01/30/vzCad8Vt3KA4Dkh.png %}

幸运的是坐标信息是储存在TIM文件内的，只需要查看下源文件坐标就可以了。

### 探针法

那我们如何得知一个TIM文件的坐标点呢？迄今为止没有任何一个工具可以直接查看这个参数。但是我们可以根据jpsxdec的报错信息来获取详细数值！

我将其命名为：探针法


使用img2tim随便创建一个tim文件，然后打乱其坐标。

```
img2tim -org 114514 1919810 -o test.tim test.png
```
这会生成一个test.TIM

打开jpsxdec，扫描整个光盘并保存idx索引目录文件，假设您将其命名为re.idx且光盘文件名为Mygame.bin，您要替换的贴图文件在jpsxdec中编号为163

```
java -jar jpsxdec.jar -f Mygame.bin -x re.idx -i 163 -replacetim test.TIM
```

如果你的数值够夸张，jpsxdec肯定会报类似这种的错误

```
New TIM format "320x240 8bpp xy(1, 1) WWidth:160 Len:76812 CLUT[256x1 xy(2, 2) Len:524]" does not match existing TIM format "320x240 8bpp xy(704, 256) WWidth:160 Len:76812 CLUT[256x1 xy(320, 507) Len:524]"
```

这样我们就获得了原坐标xy(704，256)和xy(320，507)

至于第二个坐标是什么我还没研究明白，但img2tim也提供了修改功能。

重新生成TIM并插入

```
img2tim -org 704 256 -plt 320 507 -o Yee.TIM test.png
// 生成Yee.tim
java -jar jpsxdex.jar -f Mygame.bin -x re.idx -i 163 -replacetim Yee.tim
//替换tim文件
```

重新运行光盘试试，你会发现贴图已经被替换了。

{% image https://s2.loli.net/2022/01/30/Jlsu59UiPd3ZrBX.png %}

# 非常规图片修改流程(多颜色表)