### **who is he**
#### 使用CE分析

> 这个Unity游戏的Assembly-CSharp.dll文件中的解密数据和key并不正确，所以需要使用ce分析

> 搜索字符串，找到附近的字符串，确定为真正的key和data

#### 什么是Assembly-CSharp.dll文件？

*Assembly-CSharp.dll文件是Unity游戏的主要程序文件，包含了游戏的逻辑、场景、资源、脚本等。*




### **MFC**
>利用xspy检测窗口信息，探测到一个自定义消息，写脚本获取句柄，然后发送这个消息，得到提示，获得des加密的key，然后句柄有加密字符串，解密得到flag
```c
#include <Windows.h>

#include <stdio.h>

int main()

{
    // 使用宽字符字符串
    HWND h = FindWindowW(NULL, L"Flag就在控件里");

    if (h)

    {
        printf("Found window handle: %p\n", h);

        SendMessage(h, 0x0464, 0, 0);

        printf("success\n");
    }

    else

    {
        printf("failure, window not found\n");
    }

}
```

### **BabyVM**
![[babyvm1.png]]
==虚拟机原理的展示

*将程序的可执行代码转化为自定义的中间操作码，opcode，如果操作码是一个字节，也叫Bytecode，字节码。，opcode通过emulator解释执行

>步骤：
- 分析虚拟机入口
- 搞清虚拟机输入
- 找到opcode位置
- 理清虚拟机结构（dispatch和handler）
- 逆向handler，分析opcode意义
![[babyvm2.png]]
*主函数
![[babyvm3.png]]
*opcode如上
![[babyvm4.png]]
B5F函数细化功能

```python
#根据汇编写脚本逆向

str=[0x69, 0x45, 0x2A, 0x37, 0x09, 0x17, 0xC5, 0x0B, 0x5C, 0x72,

  0x33, 0x76, 0x33, 0x21, 0x74, 0x31, 0x5F, 0x33, 0x73, 0x72]

flag=''

str[15],str[17]=str[17],str[15]

str[14],str[18]=str[18],str[14]

str[13],str[19]=str[19],str[13]

for i in range(32,128):

    if str[8]==((str[10]+2*str[9]+3*i)*str[12])&0xFF:

        str[8]=i

for i in range(32,128):

    if str[7]==((str[9]+2*str[8]+3*i)*str[12])&0xFF:

        str[7]=i

for i in range(32, 128):

    if str[6]==((str[8]+2*str[7]+3*i)*str[12])&0xFF:

        str[6]=i

for j in range(5,-1,-1):

    str[j]=str[j]^str[j+1]

for i in range(len(str)):

    flag+=chr(str[i])

print(flag)
```

### **TPCTF_linuxpdf
> pdf文件中中运行linux，只是个噱头而已。。。

在pdf中找到一大串base64编码，在项目源码中发现解码方法，base64+zlib压缩
然后找到关键文件，直接找到数据，看到有md5，直接爆破
**==diff指令发掘两个二进制文件的差异**

**md5爆破.py    提取文件.py
### TPCTF_portable
> binwalk

exe和elf的结合文件，可以调试出来，也可以binwalk拆开来
```python

f = open("portable_ef3c5f3a7155c5a4c6f0b08606dea575.exe","rb").read()
exe_data=  f[0x9C000:]
open("dump","wb").write(exe_data)
print("end!!!!!!!")


exe_data=  f[0x10000:]
open("dump.exe","wb").write(exe_data)
print("end!!!!!!!")
```

> 一个简单xor脚本
```python
data =[     0x34, 0x2A, 0x42, 0x0E, 0x00, 0x1D, 0x5C, 0x33, 0x5E, 0x44, 
  0x3E, 0x1A, 0x0B, 0x5C, 0x2C, 0x3A, 0x5F, 0x22, 0x03, 0x28, 
  0x36, 0x1B, 0x07, 0x31, 0x8D, 0xDE, 0x10, 0xA2, 0xEB, 0xB2, 
  0xDA, 0xA2, 0xD8, 0x18, 0x0D, 0x17, 0x1C, 0x1F, 0xBD, 0xD9, 
  0x1D, 0xBF, 0xEB, 0xA2, 0xD8, 0x16, 0x0D, 0xA0, 0xF6, 0x30, 
  0xBD, 0xD8, 0x17, 0xBE, 0xDA, 0x0F, 0xAB, 0xC1, 0xAE, 0xEA, 
  0x8D, 0xDE, 0x11, 0x01, 0xA1, 0xC5
]

flag = []
print(len(data))
key = b"Cosmopolitan"
for i in range(66):
    flag.append(data[i]^key[i%12])
flag = b"TPCTF{"+bytes(flag)+b"}"


# 要运行的程序路径
program_path = "./portable_ef3c5f3a7155c5a4c6f0b08606dea575.exe"  # 替换为你的程序路径

# 要发送给程序的输入数据
input_data = b"Hello, World!\n"  # 替换为你的输入数据

print(flag.decode())
```

### **TPCTF_chase

1. 通关得到一个
2. 分析res得到两个


### **NCTF_X1Login

>安卓逆向，native层逆向

需要几个frida脚本：
1. loaddex.js:获取dex
2. getstr.js:获取加密字符串
3. hookdocheck.js:观察docheck函数的参数和输出，推断加密方式
需要几个python脚本：
4. makedex.py：将dex字节变成文件
5. decrypt.py：进行解密
使用了3DES加密，注意大小端的区别
### **NCTF_Safeprogam
> exe逆向，TLS反调试

这题采用了CRC验证完整性的方式来反调试，TLS在程序启动或者线程开始的时候会触发，于是StarAdress函数就用来一直触发新线程，以此不断触发TLS，进行验证

TLS中同时利用异常处理机制来反调试和进行数据干预

采用SM4加密，其中tls替换了Sbox的部分数据，tls还替换了密钥


### **NCTF_ezDOS
> 下载一个DOSBox来进行调试

RC4加密
### **NCTF_gogo

### **XYCTF_dragon[XYCTF题解](https://tkazer.github.io/2025/04/07/XYCTF2025/)
知识点：CRC64，.bc文件bitcode，ll是文本形式的中间代码
给了一个bc文件，bc文件是LLVM字节码文件，可以通过LLVM工具链转化为ll文件，提高可读性
```bash
llvm-dis hello.bc -o hello.ll 
```
llvm是一个开源的的编译器基础设施项目，用于优化编译执行程序的的中间表示IR。

#CRC64
1. 初始化crc=0xFFFFFFFFFFFFFFFF
2. 逐字节处理：
```python
crc ^= (byte << 56)

for _ in range(8):
    if crc & (1 << 63):  # 检查最高位
        crc = (crc << 1) ^ POLY
    else:
        crc <<= 1
    crc &= 0xFFFFFFFFFFFFFFFF 

crc ^= 0xFFFFFFFFFFFFFFFF 

```

核心参数：Poly决定规则，init初值，结果异或，输入反转



### **XYCTF_moon

#cython


### **XYCTF_CrackMe

#win32窗口逆向

ScyllaHide反反调试
peb标志位检测
使用NtSetInformationThread设置线程属性ThreadHideFromDebugger来不让调试器调试
解题方法：：**动态调试**
经验：：分析for循环作用
`0x04C11DB7`（正向）→ 反转后为 `0xEDB88321`：：crc特征常数

### **XYCTF_ezVm

#Unicorn
CPU模拟器框架，将不同架构（如 ARM、ARM64、MIPS、x86、x86 - 64 等）的机器码指令动态地翻译为宿主平台能够理解和执行的指令序列


### **XYCTF_EzObf

#混淆
shift+f2 idc脚本执行
alt+f7 执行外部脚本
> 一个简单混淆的解密脚本
```idc
static NopCode(Addr, Length)
{
    auto i;
    for (i = 0; i < Length; i++)
    {
        PatchByte(Addr + i, 0x90);
    }
}

static rol(value, count, bits = 32)
{
    count = count % bits;
    return ((value << count) | (value >> (bits - count))) & ((1 << bits) - 1);
}

// 搜索真实汇编代码的下一个地址
static FindEnd(Addr)
{
    auto i;
    for (i = 0; i < 0x90; i++)
    {
        auto v = Dword(Addr + i);
        if (v == 0x5153509C)
        {
            return Addr + i;
        }
    }
    return 0;
}

// 搜索最后的jmp rax指令
static FindJmpRax(Addr)
{
    auto i;
    for (i = 0; i < 0x90; i++)
    {
        auto v = Word(Addr + i);
        if (v == 0xE0FF)
        {
            return Addr + i;
        }
    }
    return 0;
}

// 搜索call $+5
static FindCall(Addr)
{
    auto i;
    for (i = 0; i < 0x90; i++)
    {
        auto v = Dword(Addr + i);
        if (v == 0xE8)
        {
            return Addr + i;
        }
    }
    return 0;
}

static main()
{
    auto StartAddr = 0x1401F400D;
    while (1)
    {
        // 搜索真实汇编代码的下一个指令地址
        auto EndAddr = FindEnd(StartAddr);
        if (EndAddr == 0)
        {
            break;
        }
        // 真实汇编代码的字节长度
        auto CodeLength = EndAddr - addr - 13;
        // 搜索Call $+5
        auto CallAddr = FindCall(addr + 13 + CodeLength);
        if (CallAddr == 0)
        {
            break;
        }
        // call $+5的下一条指令地址，即call时push到栈的返回地址
        auto CalcAddr = CallAddr + 5;
        auto ebx = Dword(CalcAddr + 2);
        auto rol_Value = Byte(CalcAddr + 8);
        auto Mode = Dword(CalcAddr + 9);
        ebx = rol(ebx, rol_Value);

        // 搜索最尾部的jmp rax指令地址
        auto JmpRaxAddr = FindJmpRax(addr);
        if (JmpRaxAddr == 0)
        {
            break;
        }
        // 第一部分垃圾指令长度
        auto TrushCodeLength_1 = CallAddr - (addr + 13 + CodeLength);
        // 第二部分垃圾指令长度
        auto TrushCodeLength_2 = JmpRaxAddr - CallAddr + 2;
        // Nop掉无用的所有代码
        NopCode(CallAddr, TrushCodeLength_2);

        NopCode(addr, 13);

        NopCode(addr + 13 + CodeLength, TrushCodeLength_1);
        // 一共两种地址计算，加和减
        if (Mode == 0xffC32B48)
        {
            CalcAddr = CalcAddr - ebx;
        }
        if (Mode == 0xffC30348)
        {
            CalcAddr = CalcAddr + ebx;
        }
        auto JmpCodeAddr = EndAddr;
        // 计算相对跳转地址
        auto JmpOffset = CalcAddr - JmpCodeAddr + 5;
        // 写入jmp指令
        PatchByte(JmpCodeAddr, 0xE9);
        PatchDword(JmpCodeAddr + 1, JmpOffset);
        // jmp的地址为下一次deobf起始地址
        addr = CalcAddr;
    }
}

```

### **XCTF_Summer

#函数式语言Haskell

### **ISCC**
SMID数据操作

S-record数据
![[3b78d43b00d1a6661fa2cb51893f9d94.png]]
可执行的数据

sqlcipher加密的数据库






### **JD**
drillbeam

adbd检测，改变delta的值，影响加密



### **D3
#### d3piano**
init_array段在载入的时候优先执行

#### **d3rpg__rgssgame
脱壳，解压，分析**

#### **d3Alice__Ptrace**
父子进程检测

#### **d3kernel__反调试**
IsDebuggerPresent
CheckRemoteDebuggerPresent
双机调试





### **Harmony**

ohbaby
鸿蒙HDF与HCS
hc-gen解包HCB为HCS配置文件
bzimage提取出chall_service


