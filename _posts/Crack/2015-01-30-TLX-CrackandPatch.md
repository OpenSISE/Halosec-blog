---
published: true
title: 记一次屏幕录像专家加密视频的破解及Patch经历
layout: post
authors: Chr1sh3ng
category: Crack
tags: Crack
---

有一次在网络上寻找的学术视频资料是加密的,经过了解发现它是由屏幕录像专家这个软件录制并使用自带加密功能进行加密。
于是乎便有了这段破解的经历。
需要用到的工具：Ollydbg
首先打开这个教学视频有如下界面：


![Lesson](http://ww3.sinaimg.cn/large/6a87387agw1euzvwmocaoj205903omx5.jpg)

破解过程：

<!-- more -->

打开工具Ollydbg把资料拖入窗口载入

![OD](http://ww4.sinaimg.cn/large/6a87387agw1euzvxusxy8j20bk07tjs9.jpg)

右键查找程序中的中文字符串

![CSearch](http://ww3.sinaimg.cn/large/6a87387agw1euzvyzrxu4j20bi0c374y.jpg)

![SResult](http://ww1.sinaimg.cn/large/6a87387agw1euzvzzzpb1j20h70g2whq.jpg)

找到密码不对进行跳转判断的相关汇编代码的位置,双击“密码不对”查看汇编代码。

![JudgePass](http://ww2.sinaimg.cn/large/6a87387agw1euzw1pvh1cj20ns0g2adv.jpg)


0040D7CB  |. /0F84 9F000000 je 加密124-.0040D870
找到这段代码,双击
把 je(条件跳转) 改成 jmp(无条件跳转)
这样就破解完成了,然后把修改patch到文件中去。

![Save](http://ww1.sinaimg.cn/large/6a87387agw1euzw2q9fjlj20eo08et9z.jpg)


右键复制到可执行文件

![](http://ww1.sinaimg.cn/large/6a87387agw1euzwbqbdu4j205r024q2y.jpg)

全部复制

![](http://ww4.sinaimg.cn/large/6a87387agw1euzwcgc95pj20ep08tgo8.jpg)

保存,选择以下路径那么一个经过patch的文件就生成在路径下。双击打开它吧^_^

**猛然发现经过patch的资料已经已经实现了任意输入密码即可打开,从而实现爆破。**