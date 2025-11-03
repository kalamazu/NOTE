30a11 增加阳光
PlantsVsZombies.exe+30A11 - 01 88 60550000 - add [eax+00005560],ecx
ida地址：430a00

种植物减少阳光
44f6a6

1de92 场上的植物数量
PlantsVsZombies.exe+1DE92 - 89 46 0C              - mov [esi+0C],eax


PVZ代码定位

1.UPX4.24 解压缩
2.CE找到数值变化
3.CE找到修改数值的汇编代码
4.读取几行汇编字节
5.IDA-serach-sequence-bytes找到相同字节，定位到汇编，反汇编

编写dll

1.安装 i686 工具链（32-bit）； 使用msys2 mingw32终端
pacman -S mingw-w64-i686-toolchain
2.编写Hook.dll
3.编译gcc -m32 -O0 -Wall -Wextra -shared -o HookDllC.dll HookDll.c -Wl,--kill-at


添加依赖

1.使用CFF Explorer修改导入导出表
2.ProcessExplorer监控依赖