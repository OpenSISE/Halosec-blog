---
published: true
title: 3. Bypass NAT 后认证
layout: post
authors: p74n3_1C4tr0
category: Crack
tags: Crack
---


本章所使用到的工具和环境

>OllyDBG , CheatEngine

**绕过NAT 后认证的限制**

>1.NAT 后认证产生原理

在蝴蝶登陆时会向服务器发送一个特定的数据包:

<!-- more -->
```python
Public Shared Function Login(ByVal server As String, ByVal mac As String, ByVal user As String, ByVal password As String, ByVal ip As String, ByVal service As String, ByVal dhcp As Boolean, ByVal version As String) As Byte()
     Dim pars As New Dictionary(Of Byte, Byte())
     pars.Add(0, New Byte() { 1 })
     pars.Add(7, Convert.HexToBytes(mac))
     pars.Add(1, Convert.StringToBytes(user))
     pars.Add(2, Convert.StringToBytes(password))
     pars.Add(9, Convert.StringToBytes(ip))
     pars.Add(10, Convert.StringToBytes(service))
     pars.Add(14, If(dhcp, New Byte() { 1 }, New Byte(1  - 1) {}))
     pars.Add(&H1F, Convert.StringToBytes(version))
     Return Crypto3848.encrypt(ProtocalUtil.BuildRequestBody(pars))
End Function      
```

注意,数据包里面携带着本机的IP 地址,蝴蝶服务器是通过这个IP 地址来确实是否为NAT 后认证登陆(因为正常的登陆是172.16 的IP ,NAT 后认证的登陆IP 为192.168 ).于是绕过NAT 后认证的思路也出来了:把原来NAT 后的IP 地址用原来分配给本机的IP 填充发送即可

>2.Hook GetAdaptersInfo

红蝴蝶是利用IPHLPAPI.dll 的GetAdaptersInfo 来获取本机的物理网卡地址,IP 地址等信息的,于是这里便给我们提供了突破思路,通过Hook 该函数使其产生我们希望的IP 地址即可,但是再往下思考,必然会遇到几个问题:

>Q1.如何加载DLL
>Q2.如何Hook
>Q3.如何修改自定义IP 地址


 既然谈到如何完美地破解软件限制,那么有必要把蝴蝶启动过程分析下:

  程序初始化环境之后会先进入**WinMain()**

```asm
.text:00426F0D loc_426F0D:                             ; CODE XREF: start+C9j
.text:00426F0D                 push    eax             ; nShowCmd
.text:00426F0E                 push    [ebp+lpCmdLine] ; lpCmdLine
.text:00426F11                 push    esi             ; hPrevInstance
.text:00426F12                 push    esi             ; lpModuleName
.text:00426F13                 call    ds:GetModuleHandleA
.text:00426F19                 push    eax             ; hInstance
.text:00426F1A                 call    _WinMain@16     ; WinMain(x,x,x,x)
```

然后**WinMain()** 直接把所有东西递交给另一个处理函数

```asm
.text:004347FE ; int __stdcall WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd)
.text:004347FE _WinMain@16     proc near               ; CODE XREF: start+DBp
.text:004347FE
.text:004347FE hInstance       = dword ptr  4
.text:004347FE hPrevInstance   = dword ptr  8
.text:004347FE lpCmdLine       = dword ptr  0Ch
.text:004347FE nShowCmd        = dword ptr  10h
.text:004347FE
.text:004347FE                 push    [esp+nShowCmd]
.text:00434802                 push    [esp+4+lpCmdLine]
.text:00434806                 push    [esp+8+hPrevInstance]
.text:0043480A                 push    [esp+0Ch+hInstance]
.text:0043480E                 call    sub_43BC8E
.text:00434813                 retn    10h
.text:00434813 _WinMain@16     endp
```

然后就是先对AFX 和窗口对象进行初始化,成功后转入<code>0x00412CF0</code> (粗体表示)

```asm
.text:0043BC8E sub_43BC8E      proc near               ; CODE XREF: WinMain(x,x,x,x)+10p
.text:0043BC8E
.text:0043BC8E arg_0           = dword ptr  4
.text:0043BC8E arg_4           = dword ptr  8
.text:0043BC8E arg_8           = dword ptr  0Ch
.text:0043BC8E arg_C           = dword ptr  10h
.text:0043BC8E
.text:0043BC8E                 push    ebx
.text:0043BC8F                 push    esi
.text:0043BC90                 push    edi
.text:0043BC91                 or      ebx, 0FFFFFFFFh
.text:0043BC94                 call    ?AfxGetThread@@YGPAVCWinThread@@XZ ; AfxGetThread(void)
.text:0043BC99                 mov     esi, eax
.text:0043BC9B                 call    ?AfxGetModuleState@@YGPAVAFX_MODULE_STATE@@XZ ; AfxGetModuleState(void)
.text:0043BCA0                 push    [esp+0Ch+arg_C]
.text:0043BCA4                 mov     edi, [eax+4]
.text:0043BCA7                 push    [esp+10h+arg_8]
.text:0043BCAB                 push    [esp+14h+arg_4]
.text:0043BCAF                 push    [esp+18h+arg_0]
.text:0043BCB3                 call    sub_440336      ; bool AfxWinInit(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPTSTR lpCmdLine,int nCmdShow);
.text:0043BCB8                 test    eax, eax
.text:0043BCBA                 jz      short loc_43BCF7
.text:0043BCBC                 test    edi, edi
.text:0043BCBE                 jz      short loc_43BCCE
.text:0043BCC0                 mov     eax, [edi]
.text:0043BCC2                 mov     ecx, edi
.text:0043BCC4                 call    dword ptr [eax+84h] ; call 0x0043F9D7
.text:0043BCC4                                         ;
.text:0043BCC4                                         ; CWinApp 对象初始化
.text:0043BCCA                 test    eax, eax
.text:0043BCCC                 jz      short loc_43BCF7
.text:0043BCCE
.text:0043BCCE loc_43BCCE:                             ; CODE XREF: sub_43BC8E+30j
.text:0043BCCE                 mov     eax, [esi]
.text:0043BCD0                 mov     ecx, esi
.text:0043BCD2                 call    dword ptr [eax+50h] ; call 0x00412A10
.text:0043BCD5                 test    eax, eax
.text:0043BCD7                 jnz     short loc_43BCEE
.text:0043BCD9                 mov     ecx, [esi+1Ch]
.text:0043BCDC                 test    ecx, ecx
.text:0043BCDE                 jz      short loc_43BCE5
.text:0043BCE0                 mov     eax, [ecx]
.text:0043BCE2                 call    dword ptr [eax+58h]
.text:0043BCE5
.text:0043BCE5 loc_43BCE5:                             ; CODE XREF: sub_43BC8E+50j
.text:0043BCE5                 mov     eax, [esi]
.text:0043BCE7                 mov     ecx, esi
.text:0043BCE9                 call    dword ptr [eax+68h] ; call 0x00412CF0**
.text:0043BCE9                                         ;
.text:0043BCE9                                         ; 删除CWin 对象,退出程序
.text:0043BCEC                 jmp     short loc_43BCF5
.text:0043BCEE ; ---------------------------------------------------------------------------
.text:0043BCEE
.text:0043BCEE loc_43BCEE:                             ; CODE XREF: sub_43BC8E+49j
.text:0043BCEE                 mov     eax, [esi]
.text:0043BCF0                 mov     ecx, esi
.text:0043BCF2                 call    dword ptr [eax+54h]
.text:0043BCF5
.text:0043BCF5 loc_43BCF5:                             ; CODE XREF: sub_43BC8E+5Ej
.text:0043BCF5                 mov     ebx, eax
.text:0043BCF7
.text:0043BCF7 loc_43BCF7:                             ; CODE XREF: sub_43BC8E+2Cj
.text:0043BCF7                                         ; sub_43BC8E+3Ej
.text:0043BCF7                 call    ?AfxWinTerm@@YGXXZ ; AfxWinTerm(void)
.text:0043BCFC                 pop     edi
.text:0043BCFD                 mov     eax, ebx
.text:0043BCFF                 pop     esi
.text:0043BD00                 pop     ebx
.text:0043BD01                 retn    10h
.text:0043BD01 sub_43BC8E      endp
```

这个<code>0x00412CF0</code> 过程就是整个蝴蝶中最重要的初始化过程,由于这个函数过程太长了,于是直接总结,如下:

```asm
.text:0043BCD2                 call    dword ptr [eax+50h] ; call 0x00412A10
.text:0043BCD2                                         ;
.text:0043BCD2                                         ; bool client_init(void);
.text:0043BCD2                                         ;
.text:0043BCD2                                         ; 该函数主要是初始化客户端
.text:0043BCD2                                         ; 1.初始化线程互斥量
.text:0043BCD2                                         ; 2.检测当前是否同时启动多个程序
.text:0043BCD2                                         ; 3.判断当前用户是否为guest 用户
.text:0043BCD2                                         ; 4.初始化Winsock
.text:0043BCD2                                         ; 5.从IPHLPAPI.DLL 中导出一些函数
.text:0043BCD2                                         ; 6.获取Windows 系统版本
.text:0043BCD2                                         ; 7.加载部分文件(暂未知用意)
.text:0043BCD2                                         ; 8.初始化登陆窗口
.text:0043BCD2                                         ; 9.获取Amt/SAmt 驱动版本
.text:0043BCD2                                         ; 10.进入登陆窗口
.text:0043BCD2                                         ; 11.卸载Winsock
.text:0043BCD2                                         ;
.text:0043BCD2                                         ;
.text:0043BCD2                                         ; WARNING!
.text:0043BCD2                                         ; 1.如果遇到端口不能初始化,则需要重置Winsock ,原因是Winsock 库不能正常初始化,在这里蝴蝶客户端并没有绑定端口
.text:0043BCD2                                         ; 2.如果遇到无法加载IPHLPAPI 函数库,则需要重新复制新的DLL 到system32 目录下,原因是不能正确地从该DLL 中导出函数
.text:0043BCD2                                         ; 3.如果遇到客户端安装失败,"Supplicant Service not Start Ore not Install,Please ReInstall!",则需要重新安装蝴蝶,原因是驱动版本过低(低于V2.2)
.text:0043BCD2                                         ;这里有一处可以利用的地方,就是出IPHLPAPI.DLL 的函数导入
.text:0040DDC0
.text:0040DDC0 sub_40DDC0      proc near               ; CODE XREF: sub_412A10+19Dp
.text:0040DDC0                 push    edi
.text:0040DDC1                 push    offset a222_88_88_62 ; "222.88.88.62"
.text:0040DDC6                 call    ds:__imp_inet_addr
.text:0040DDCC                 push    eax             ; hostlong
.text:0040DDCD                 call    ds:__imp_htonl
.text:0040DDD3                 xor     edi, edi
.text:0040DDD5                 push    offset aIphlpapi_dll ; "iphlpapi.dll"
.text:0040DDDA                 mov     GetAdaptersInfo_FunctionAddress, edi
.text:0040DDE0                 mov     dword_4629C0, edi
.text:0040DDE6                 mov     dword_4629BC, edi
.text:0040DDEC                 mov     dword_4629B8, edi
.text:0040DDF2                 mov     dword_4629B4, edi
.text:0040DDF8                 mov     dword_4629B0, edi
.text:0040DDFE                 mov     dword_4629AC, edi
.text:0040DE04                 call    ds:LoadLibraryA
.text:0040DE0A                 cmp     eax, edi
.text:0040DE0C                 mov     hModule, eax
.text:0040DE11                 jz      loc_40DED1
.text:0040DE17                 push    esi
.text:0040DE18                 mov     esi, ds:GetProcAddress
.text:0040DE1E                 push    offset aGetadaptersinf ; "GetAdaptersInfo"
.text:0040DE23                 push    eax             ; hModule
.text:0040DE24                 call    esi ; GetProcAddress
.text:0040DE26                 mov     GetAdaptersInfo_FunctionAddress, eax
.text:0040DE2B                 mov     eax, hModule
.text:0040DE30                 push    offset aGetnetworkpara ; "GetNetworkParams"
.text:0040DE35                 push    eax             ; hModule
.text:0040DE36                 call    esi ; GetProcAddress
.text:0040DE38                 mov     ecx, hModule
.text:0040DE3E                 push    offset aGetinterfacein ; "GetInterfaceInfo"
.text:0040DE43                 push    ecx             ; hModule
.text:0040DE44                 mov     dword_4629C0, eax
.text:0040DE49                 call    esi ; GetProcAddress
.text:0040DE4B                 mov     edx, hModule
.text:0040DE51                 push    offset aIpreleaseaddre ; "IpReleaseAddress"
.text:0040DE56                 push    edx             ; hModule
.text:0040DE57                 mov     dword_4629BC, eax
.text:0040DE5C                 call    esi ; GetProcAddress
.text:0040DE5E                 mov     dword_4629B8, eax
.text:0040DE63                 mov     eax, hModule
.text:0040DE68                 push    offset aIprenewaddress ; "IpRenewAddress"
.text:0040DE6D                 push    eax             ; hModule
.text:0040DE6E                 call    esi ; GetProcAddress
.text:0040DE70                 mov     ecx, hModule
.text:0040DE76                 push    offset aAddipaddress ; "AddIPAddress"
.text:0040DE7B                 push    ecx             ; hModule
.text:0040DE7C                 mov     dword_4629B4, eax
.text:0040DE81                 call    esi ; GetProcAddress
.text:0040DE83                 mov     edx, hModule
.text:0040DE89                 push    offset aDeleteipaddres ; "DeleteIPAddress"
.text:0040DE8E                 push    edx             ; hModule
.text:0040DE8F                 mov     dword_4629B0, eax
.text:0040DE94                 call    esi ; GetProcAddress
.text:0040DE96                 mov     ecx, GetAdaptersInfo_FunctionAddress
.text:0040DE9C                 mov     dword_4629AC, eax
.text:0040DEA1                 cmp     ecx, edi
.text:0040DEA3                 pop     esi
.text:0040DEA4                 jz      short loc_40DED1
.text:0040DEA6                 cmp     dword_4629BC, edi
.text:0040DEAC                 jz      short loc_40DED1
.text:0040DEAE                 cmp     dword_4629B8, edi
.text:0040DEB4                 jz      short loc_40DED1
.text:0040DEB6                 cmp     dword_4629B4, edi
.text:0040DEBC                 jz      short loc_40DED1
.text:0040DEBE                 cmp     dword_4629B0, edi
.text:0040DEC4                 jz      short loc_40DED1
.text:0040DEC6                 cmp     eax, edi
.text:0040DEC8                 jz      short loc_40DED1
.text:0040DECA                 mov     eax, 1
.text:0040DECF                 pop     edi
.text:0040DED0                 retn
.text:0040DED1 ; ---------------------------------------------------------------------------
.text:0040DED1
.text:0040DED1 loc_40DED1:                             ; CODE XREF: sub_40DDC0+51j
.text:0040DED1                                         ; sub_40DDC0+E4j ...
.text:0040DED1                 xor     eax, eax
.text:0040DED3                 pop     edi
.text:0040DED4                 retn
.text:0040DED4 sub_40DDC0      endp
```

记得要特别注意代码清单里面的粗体代码,标记它们的意义如下:
>1.蝴蝶用LoadLibrary() 加载iphlpapi.dll
>2.所有函数入口点地址都保存在全局变量中
>3.一共导入了7 个函数

至此我们突破NAT 限制的思路已经清晰了:

>1.把LoadLibrary() 重定向加载我们自己写的DLL
>2.DLL 只导出蝴蝶需要用到的iphlpapi.dll 函数

接下来,就是代码时间...

>3.绕过保护代码

下面是DLL 文件的代码:

```c++
#include <memory.h>
#include <string.h>

#include <string>

#include <windows.h>
#include <winsock.h>

#pragma comment (lib,"ws2_32")

using std::string;

#define MAX_ADAPTER_NAME_LENGTH 256
#define MAX_ADAPTER_DESCRIPTION_LENGTH 128
#define MAX_ADAPTER_ADDRESS_LENGTH 8

typedef struct {
    char String[4 * 4];
} IP_ADDRESS_STRING, *PIP_ADDRESS_STRING, IP_MASK_STRING, *PIP_MASK_STRING;
 
typedef struct _IP_ADDR_STRING {
    struct _IP_ADDR_STRING* Next;
    IP_ADDRESS_STRING IpAddress;
    IP_MASK_STRING IpMask;
    DWORD Context;
} IP_ADDR_STRING, *PIP_ADDR_STRING;

 typedef struct _IP_ADAPTER_INFO {  
  struct _IP_ADAPTER_INFO *Next;
  DWORD ComboIndex;  
  char AdapterName[MAX_ADAPTER_NAME_LENGTH + 4];
  char Description[MAX_ADAPTER_DESCRIPTION_LENGTH + 4];
  UINT AddressLength;  
  BYTE Address[MAX_ADAPTER_ADDRESS_LENGTH];
  DWORD Index;
  UINT Type;  
  UINT DhcpEnabled;
  PIP_ADDR_STRING CurrentIpAddress;
  IP_ADDR_STRING IpAddressList;
  IP_ADDR_STRING GatewayList;
  IP_ADDR_STRING DhcpServer;
  BOOL HaveWins;
  IP_ADDR_STRING PrimaryWinsServer;
  IP_ADDR_STRING SecondaryWinsServer;
  time_t LeaseObtained;
  time_t LeaseExpires;
 } IP_ADAPTER_INFO,  *PIP_ADAPTER_INFO;

typedef DWORD (__stdcall *_GetAdaptersInfo)(PIP_ADAPTER_INFO,PULONG);
typedef DWORD (__stdcall *_GetNetworkParams)(void*,long*);
typedef DWORD (__stdcall *_GetInterfaceInfo)(void*,long*);
typedef DWORD (__stdcall *_IpReleaseAddress)(void*);
typedef DWORD (__stdcall *_IpRenewAddress)(void*);
typedef DWORD (__stdcall *_AddIPAddress)(void*,void*,long,long*,long*);
typedef DWORD (__stdcall *_DeleteIPAddress)(long*);

_GetAdaptersInfo  GetAdaptersInfo_=NULL;
_GetNetworkParams GetNetworkParams_=NULL;
_GetInterfaceInfo GetInterfaceInfo_=NULL;
_IpReleaseAddress IpReleaseAddress_=NULL;
_IpRenewAddress   IpRenewAddress_=NULL;
_AddIPAddress     AddIPAddress_=NULL;
_DeleteIPAddress  DeleteIPAddress_=NULL;

HMODULE dll_iphlpapi=NULL;
HANDLE  file_config=INVALID_HANDLE_VALUE;

#define IPV4_LENGTH 16
#define READ_BUFFER_LENGTH 64
#define INPUT_FUNCTION_ADDRESS 0x004629C4

bool send_ip=false;
char send_ip_addr[IPV4_LENGTH]={0};

DWORD __stdcall Hook_GetAdaptersInfo(PIP_ADAPTER_INFO output_data,PULONG output_length);

BOOL APIENTRY DllMain(HANDLE hModule, DWORD ul_reason_for_call, LPVOID lpReserved) {
    if (DLL_PROCESS_ATTACH==ul_reason_for_call) {
        WSADATA startup;
        WSAStartup(1,&startup);

        dll_iphlpapi=LoadLibrary("iphlpapi.dll");
        GetAdaptersInfo_=(_GetAdaptersInfo)GetProcAddress(dll_iphlpapi,"GetAdaptersInfo");
        GetNetworkParams_=(_GetNetworkParams)GetProcAddress(dll_iphlpapi,"GetNetworkParams");
        GetInterfaceInfo_=(_GetInterfaceInfo)GetProcAddress(dll_iphlpapi,"GetInterfaceInfo");
        IpReleaseAddress_=(_IpReleaseAddress)GetProcAddress(dll_iphlpapi,"IpReleaseAddress");
        IpRenewAddress_=(_IpRenewAddress)GetProcAddress(dll_iphlpapi,"IpRenewAddress");
        AddIPAddress_=(_AddIPAddress)GetProcAddress(dll_iphlpapi,"AddIPAddress");
        DeleteIPAddress_=(_DeleteIPAddress)GetProcAddress(dll_iphlpapi,"DeleteIPAddress");

        if (NULL!=dll_iphlpapi && NULL!=GetAdaptersInfo_) {  //  加载iphlpapi.dll 失败时也要跟着失败
            file_config=CreateFile("ip_config.dat",GENERIC_READ,0,NULL,OPEN_EXISTING,0,NULL);  //  读取同路径下的ip_config.dat 文件
            if (INVALID_HANDLE_VALUE!=file_config) {
                char read_buffer[READ_BUFFER_LENGTH]={0};
                unsigned long read_buffer_length=0;
                if (ReadFile(file_config,read_buffer,READ_BUFFER_LENGTH,&read_buffer_length,NULL)) {
                    /*
                        配置文件格式:
                        ipchange=true:ipaddress=123.456.789.000
                    */
                    if (13<=read_buffer_length) {
                        string resolve_string(read_buffer);
                        if (resolve_string.find(':')) {
                            long string_length=resolve_string.length();
                            long point_comment=resolve_string.find(':');
                            char ipchange_buffer[READ_BUFFER_LENGTH/2]={0};
                            char ipaddress_buffer[READ_BUFFER_LENGTH/2]={0};
                            memcpy(ipchange_buffer,resolve_string.c_str(),point_comment);
                            memcpy(ipaddress_buffer,resolve_string.c_str()+point_comment+1,string_length-point_comment-1);
                            
                            resolve_string=ipchange_buffer;
                            string_length=resolve_string.length();
                            point_comment=resolve_string.find('=');
                            memset(ipchange_buffer,0,READ_BUFFER_LENGTH/2);
                            memcpy(ipchange_buffer,resolve_string.c_str()+point_comment+1,string_length-point_comment-1);

                            if (!strcmp(strlwr(ipchange_buffer),"true")) {  //  判断ipchange 是否为true
                                resolve_string=ipaddress_buffer;
                                string_length=resolve_string.length();
                                point_comment=resolve_string.find('=');
                                memcpy(send_ip_addr,resolve_string.c_str()+point_comment+1,string_length-point_comment-1);  //  读取自定义IP 地址
                                send_ip=true;
                            }
                        }
                    }
                }
            }
            return TRUE;
        }
    } else if (DLL_PROCESS_DETACH) {
        FreeLibrary(dll_iphlpapi);
        WSACleanup();
    }
    return FALSE;
}

DWORD __stdcall GetAdaptersInfo(PIP_ADAPTER_INFO output_data,PULONG output_length) {
    DWORD return_code=GetAdaptersInfo_(output_data,output_length);
    if (NULL!=output_data && send_ip)
        memcpy(output_data->IpAddressList.IpAddress.String,send_ip_addr,IPV4_LENGTH);  //  修改原来获取192.168 的NAT 后IP 地址
    return return_code;
}

DWORD __stdcall GetNetworkParams(void* a,long* b) {
    return GetNetworkParams_(a,b);
}
DWORD __stdcall GetInterfaceInfo(void* a,long* b) {
    return GetInterfaceInfo_(a,b);
}
DWORD __stdcall IpReleaseAddress(void* a) {
    return IpReleaseAddress_(a);
}
DWORD __stdcall IpRenewAddress(void* a) {
    return IpRenewAddress_(a);
}
DWORD __stdcall AddIPAddress(void* a,void* b,long c,long* d,long* f) {
    return AddIPAddress_(a,b,c,d,f);
}
DWORD __stdcall DeleteIPAddress(long* a) {
    return DeleteIPAddress_(a);
}
```

写好了DLL 之后,下一步就是到蝴蝶启动代码里面修改相应的数据

![1](http://ww4.sinaimg.cn/large/005YTBXDgw1ev035w8mjij30a905gt93.jpg)

为什么不把我们的DLL 的名字改为iphlpapi.dll 呢?因为Win7 下搜索DLL 默认是从system32 下开始搜索的,不像以前的XP 系统,Win7 之所以这样做是因为要防止DLL 劫持