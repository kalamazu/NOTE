#字符串00
绕过长度检测

#栈溢出
输入溢出，或者strcpy溢出

#换行符 
10

#canary
通过输出buf泄露canary

#Linux绕过
[绕过](https://blog.csdn.net/m0_67844671/article/details/133239381)
1.1分号绕过 ；
分号可以区分不同命令

1.2管道符 |
Linux所提供的管道符“|”将两个命令隔开，管道符左边命令的输出就会作为管道符右边命令的输入。

1.3 &
 **command1 & command2**
 &放在启动参数后面表示设置此进程为后台进程，上面的命令会先执行command2，再执行command1
 >ping -c 1 baidu.com & ls -l
 先执行ls，再ping

1.4&&
前一条成功才执行下一条

1.5||
> **跟&&符号相反，前面那条命令为假才会执行后面的命令**
> **只要有一个命令返回真（命令返回值 $? == 0），后面的命令就不会被执行。–直到返回真的地方停止执行。**


1.6绕过空格
```bash
{cat,etc/passwd}
cat${IFS}flag.txt
cat$IFS$9flag.txt
cat${IFS}flag.txt
cat<flag.txt
cat<>flag.txt
```
1.7关键字
转义
c\a\t<>f\l\a\g.t\x\t
空绕过
cat fla''g.php

#patchelf
#readelf
#nm
nm -D cat
分析导入了什么符号
nm -a cat
分析全部符号
#objcopy
修改段
#strip 
除去一些东西、
#echo$'\x34'