spawn：启动一个新进程并注入Frida
attach：Hook已经运行的App

```shell
@echo off
REM 1. 检查ADB设备连接
adb devices
if %errorlevel% neq 0 (
    echo 错误：未检测到ADB设备或ADB未配置！
    pause
    exit /b
)

REM 2. 执行adb shell命令并自动输入后续操作
echo 正在执行adb shell命令...
adb shell "su -c 'cd /data/local/tmp && ./fr'"

REM 3. 检查执行结果
if %errorlevel% eq 0 (
    echo 命令执行成功！
) else (
    echo 错误：命令执行失败，请检查设备root权限或文件路径！
)
pause
```


如何编写frida脚本
1.主动调用库函数 new NativeFunction
2.hook 已知运行中库函数 基于地址：Interceptor.attch(addr,{})
3.hook java函数  Java.use+overload+implementation
4.枚举类并枚举方法，找到目标方法 Java.enumerateLoadedClasses+clazz.getDeclaredMethods
5.hook 构造函数，在new一个类的时候改变某些字段或者方法class.$init.overload('').implementation=
6.stalker


当要接管方法时：
func.implementation=function(){

}
or
func.overload("xxx").implementation=function(s,a){

}
of
clazz[func].overload("").implementation=function(s,a){

}

未知参数数量时：this[methodname].apply(this,argumnets)，apply可以展开参数，可以使用arguments来传参