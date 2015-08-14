---
published: true
title: 1. Custom 客户端UI
layout: post
authors: p74n3_1C4tr0
category: Crack
tags: Crack
---


本章所使用到的工具和环境

>Visual Studio 2010 (任意版本都可以) ,CFF Explorer

####**在Visual Studio 中导入客户端程序**

在”打开”->”文件” 中打开蝴蝶客户端程序,在Visual Studio 中只支持这种方式来打开程序文件

选中客户端文件

![1](http://ww4.sinaimg.cn/large/005YTBXDgw1ev01ktojywj30fe0aadi6.jpg)

<!-- more -->
然后可以看到许多程序里面的资源文件

![2](http://ww3.sinaimg.cn/large/005YTBXDgw1ev01m9fv6xj306305i74s.jpg)

由于本节着重介绍修改UI ,所以接下来重点关注这几个文件夹:

>Bitmap:图片资源
>Dialog:窗口资源
>Icon:图标资源

1.修改客户端主界面

打开ID 为102 的窗口,这个就是主界面

![3](http://ww3.sinaimg.cn/large/005YTBXDgw1ev01o678toj30710dimxu.jpg)

双击进去,就可以任意设计界面

![4](http://ww1.sinaimg.cn/large/005YTBXDgw1ev01ool5kvj30fd0dzdiy.jpg)

到了这一步相信上过C# 开发的同学们都知道应该怎么做了,但是在设计的时候千万不能够忘记一点,就是把原来界面上的控件删除掉,只能通过设置为Visual 为False 把他们对客户来说不可见.

 

2.添加自定义的图片

在制作和添加图片的时候需要考虑图片的大小,假设设置图片的格式为300 * 135

![5](http://ww2.sinaimg.cn/large/005YTBXDgw1ev01sv7ym6j30fe0f340l.jpg)

然后得出来的画布大小刚好适合

![6](http://ww1.sinaimg.cn/large/005YTBXDgw1ev01todyuej30be092gmi.jpg)

随意画个东西之后记得要以BMP 位图的格式来保存图片,应该选择24 位的位图来保存,这样确保输出的图片的有高的清晰度

![7](http://ww4.sinaimg.cn/large/005YTBXDgw1ev01u7hzjkj30fd04jgma.jpg)

把保存好的图片插入到客户端中

![8](http://ww2.sinaimg.cn/large/005YTBXDgw1ev01unznpcj30aa07jgmt.jpg)

选择导入Bitmap

![9](http://ww4.sinaimg.cn/large/005YTBXDgw1ev01v4wkwwj30bu094758.jpg)

选择刚才画好的图片后,资源就已经被添加到客户端中

![10](http://ww1.sinaimg.cn/large/005YTBXDgw1ev01vj5n8jj30aa08ht9r.jpg)

然后找到主界面中把原来的安腾网络的图片修改掉,注意,此时image 控件是以图片的ID 来显示图片的,原来的位图的ID 为175

![11](http://ww3.sinaimg.cn/large/005YTBXDgw1ev01w36nwhj30a000xglh.jpg)

把他修改成刚才插入的新位图ID 101

![12](http://ww2.sinaimg.cn/large/005YTBXDgw1ev01wjaei7j309q00zq2t.jpg)

主界面就有变化了

![13](http://ww3.sinaimg.cn/large/005YTBXDgw1ev01wzksb5j309407gaav.jpg)

3.修改程序显示图标

最后一步就是修改程序图标文件,如何去画一个好看的ICON 就不讨论啦,下面是怎么把另一个程序的ICON 移植到客户端上

右键选中Shadowsocks.exe ,用CFF Explorer 打开

![14](http://ww3.sinaimg.cn/large/005YTBXDgw1ev01y7fsm0j30at0530th.jpg)

找到ICON 并且保存到本地文件

![15](http://ww3.sinaimg.cn/large/005YTBXDgw1ev01yneu8mj30fe0dkn08.jpg)

用同样的方法打开蝴蝶客户端,并且把刚才保存的图标文件取代原来的图标

![16](http://ww1.sinaimg.cn/large/005YTBXDgw1ev01z4hpyfj30f60c1jvf.jpg)

修改过后保存,效果就出来了

![17](http://ww3.sinaimg.cn/large/005YTBXDgw1ev01zjy3txj30cc00ut8p.jpg)