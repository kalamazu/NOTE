动态注册加载机制：
Java类加载 
    ↓
System.loadLibrary("xxx")
    ↓
→ linker 加载 libxxx.so
    ↓
→ 调用 JNI_OnLoad()
    ↓
→ 在 JNI_OnLoad() 中执行 RegisterNatives()
    ↓
→ JVM 记录绑定关系：Java方法 <-> native函数指针
    ↓
Java 调用 native 方法时
    ↓
→ JVM 直接跳转到 native 函数执行（无需查找符号）


so加载流程：
so加载
1.systemloadlibrary发起，在标准路径找lib；systemload需要完整绝对路径，常用于非标准位置的so库；接着内核的dlopen启动，启动linker

2.linker解析ELF programheader，将需要的代码段和数据映射到进程的虚拟内存空间，指定读写执行权限；

3.重定位：采用装载时重定位，由Linker集中完成所有符号的解析和修正（使用PIC选项可以使得代码可以加载到内存任意位置执行）

4.初始化，执行initarray指定的构造函数，可在dlsym找到这个so里的符号了，函数名和变量

5.JNI_OnLoad：注册native方法