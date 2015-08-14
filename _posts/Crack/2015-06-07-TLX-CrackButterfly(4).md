---
published: true
title: 4. Sniffing WIFI 监听登陆数据包
layout: post
authors: p74n3_1C4tr0
category: Crack
tags: Crack
---


本章所使用到的工具和环境

>Wireshark ,IDA pro ,无线路由器,集线器

**蝴蝶通信原理**

蝴蝶以UDP 工作在3848 端口上和服务器交互数据,使用Wireshark 抓包并且输入udp.port==3848 开始分析通信过程

![1](http://ww4.sinaimg.cn/large/005YTBXDgw1ev0398txabj30fd03bmyd.jpg)

>蝴蝶登陆过程:

<!-- more -->
![2](http://ww2.sinaimg.cn/large/005YTBXDgw1ev039m0igaj30fe03sgmj.jpg)

登陆数据包(包含帐号密码) -> 服务器
客户端 <- 认证数据包(包含服务器标记16 位KEY)
心跳包(服务器标记KEY) -> 服务器
客户端 <- 心跳包应答

>后台通信过程:

![3](http://ww3.sinaimg.cn/large/005YTBXDgw1ev03b73i26j30fe02bdgg.jpg)

心跳包(服务器标记KEY) -> 服务器
客户端 <- 心跳包应答

蝴蝶退出过程:

![4](http://ww2.sinaimg.cn/large/005YTBXDgw1ev03byf6scj30fe02jwf4.jpg)

  退出数据包(服务器标记KEY) -> 服务器
  客户端 <- 确认数据包

>提取解密函数

从上面简单的数据流分析,我们已经明白蝴蝶通信的大致流程,但是抓取到的数据是加密的,于是必须要通过客户端去发掘解密过程,由于过程较长直接省略,算法如下:

```c++
bool __cdecl packet_decode(char *packet_buffer, int packet_length)
{
  bool result; // eax@2
  int i; // esi@5

  if ( packet_length > 0 )
  {
    if ( packet_buffer )
    {
      for ( i = 0; i < packet_length; ++i )
        packet_buffer[i] = ((unsigned __int8)(((unsigned __int8)packet_buffer[i] >> 5) | packet_buffer[i] & 0x70) >> 2) | 2 * (packet_buffer[i] & 1 | 2 * (packet_buffer[i] & 8 | 4 * (packet_buffer[i] & 4 | 4 * (packet_buffer[i] & 0xFE))));
      result = 1;
    }
    else
    {
      result = 0;
    }
  }
  else
  {
    result = 0;
  }
  return result;
}
```

通信数据包结构如下:

![5](http://ww2.sinaimg.cn/large/005YTBXDgw1ev03e8nbdjj30fe07ewfe.jpg)

把在Wireshare 上截到的数据包放到工程中去显示,最终栈上面的数据如下:

![6](http://ww1.sinaimg.cn/large/005YTBXDgw1ev03eri3ubj30fd04ngmq.jpg)

![7](http://ww4.sinaimg.cn/large/005YTBXDgw1ev03f1ej6oj30fe04rdgx.jpg)

>搭建监听环境

监听环境布置如下:外接网线插入集线器的IN 端,从集线器引出两条线一条连接监听主机另一条接入无线路由即可

![8](http://ww4.sinaimg.cn/large/005YTBXDgw1ev03fxax13j30fd08n75f.jpg)

程序代码 ---- --- -- --