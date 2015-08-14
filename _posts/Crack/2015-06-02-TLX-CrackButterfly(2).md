---
published: true
title: 2. Patch 多网卡限制
layout: post
authors: p74n3_1C4tr0
category: Crack
tags: Crack
---


本章所使用到的工具和环境

>OllyDBG

**绕过多物理网卡的限制**

>1.多物理网卡产生原理

当本地启用多个网络适配器时,蝴蝶的某个后台线程会引发错误,向主程序的错误处理函数发出通知,使之下线并通知用户

错误产生环境:

正常情况下的本地网络适配器的启动数目:

![1](http://ww2.sinaimg.cn/large/005YTBXDgw1ev027fydsqj30fe08o0tq.jpg)

<!-- more -->
蝴蝶产生异常情况下的本地网络适配器的启动数目:

![2](http://ww3.sinaimg.cn/large/005YTBXDgw1ev0282hik7j30fe08ogmt.jpg)

此时蝴蝶程序会弹出窗口,提示错误

![3](http://ww3.sinaimg.cn/large/005YTBXDgw1ev028ierd6j308p04wmxa.jpg)

>2.绕过保护代码

现在我们知道,只要信息框有提示时,就是程序本身限制掉了该功能,在OD 里面如下操作

设置信息框显示断点

![4](http://ww3.sinaimg.cn/large/005YTBXDgw1ev029r2b4qj30ap0aotap.jpg)

找到MessageBoxA

![5](http://ww4.sinaimg.cn/large/005YTBXDgw1ev02a861ebj30930c0myb.jpg)

然后`F9` 执行,不久之后代码停留在MessageBoxA 的入口点

![6](http://ww4.sinaimg.cn/large/005YTBXDgw1ev02ao1g5wj30d906875t.jpg)

然后`CTRL+F9` 执行到函数的最后一句代码,再F8 跳出到MessageBoxA

![7](http://ww1.sinaimg.cn/large/005YTBXDgw1ev02b95acjj30fe091jty.jpg)

这里还不是分析的主体逻辑代码,只是一个调用信息框的封装函数,然后往下执行跳过该函数,跳出去之后就能看到判断的代码了

![8](http://ww1.sinaimg.cn/large/005YTBXDgw1ev02bnu8fhj30fe03q3ze.jpg)

大家看到<code>0x4052AB</code> ,那儿有一个箭头,这里表示代码是从另一个地方跳过来的,于是原本关键if 判断的代码不在这个位置,于是继续往上溯源

![9](http://ww2.sinaimg.cn/large/005YTBXDgw1ev02c3yp9vj30fe072dhk.jpg)

对test – je 指令块的分析,这个判断只是选择语言用的,那么这段代码应该怎么爆破呢?思路就是把整个判断块都patch 掉, 由于程序判断是根据cmp 指令计算的返回值来判断的,那么这句jmp 汇编的意义为无论程序的判断是否正确都要跳转到jmp 中指定的地址

![10](http://ww3.sinaimg.cn/large/005YTBXDgw1ev02cnsznlj30fe0aagnw.jpg)

注意汇编pop eax ,因为堆栈中尚未平衡,于是需要pop 出一次数据才可以正确return 出去

然后再次启动蝴蝶,原本的弹窗提示已经跳过执行,但是程序还是被提示下线,经验告诉我们,应该要在主程序错误处理函数的入口点设置断点

![11](http://ww4.sinaimg.cn/large/005YTBXDgw1ev02d6dk29j30fe08jq4x.jpg)

断点命中之后,继续`CRTL+F9 + F8` 往下跟踪,发现程序执行到这个地方

![12](http://ww1.sinaimg.cn/large/005YTBXDgw1ev02f4g7lej30fd07lwga.jpg)

进去<code>0x407FA0</code> 查看代码,发现这个是下线的函数,于是也要把这里都patch 掉.

![13](http://ww3.sinaimg.cn/large/005YTBXDgw1ev02fg98nxj30fe08btao.jpg)

启动蝴蝶,成功绕过限制

![14](http://ww3.sinaimg.cn/large/005YTBXDgw1ev02fwtvb1j30ti0hswm7.jpg)