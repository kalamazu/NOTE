
### 1. IDA调试
[教程](https://www.cnblogs.com/ddms/p/8820044.html)
1)调试启动：
adb shell am start -D -n com.example.test/.MainActivity

2)启动andriod_server
adb push D:\android_server /data/local/tmp

3)使用forward端口转发：将主机（PC）上的 `23946` 端口收到的数据转发到已连接的 Android 设备上的 `23946` 端口
adb forward tcp:23946 tcp:23946

4)IDA附加调试
在entry point，thread start，library load暂停
 
5)Java Debug Wire Protocol连接到java进程调试
1. ​**远程调试**：通过 TCP/IP 连接到目标 JVM（Java 虚拟机），允许开发者在命令行环境下调试 Java 程序。
2. ​**调试模式**：适用于无 GUI 环境（如服务器）或需要轻量级调试工具的场景。
3. ​**调试已运行的程序**：目标 JVM 需以调试模式启动（监听指定端口），JDB 才能附加（Attach）到该进程。
jdb -connect com.sun.jdi.SocketAttach:hostname=127.0.0.1,port=XXXX

6)**IDA ctrl+S 查看已经载入的so，一定要先保证so已经载入了，再进行调试**



### 2.APK文件调试

1)包名应该和原应用包名一致
2)在main目录下创建jniLibs/arm64-v8a/libxx.so
3)然后即可调用这个so的函数
4)nm -D libxx.so查看可用函数


## 3. 混淆跳转
[教程](https://bbs.kanxue.com/thread-277086.htm)

