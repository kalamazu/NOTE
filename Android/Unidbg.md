https://blog.csdn.net/m0_68075044/article/details/130156575


初始化：模拟器，dalvikvm nativeso
设置日志和断点
调用native方法获取加密结果
提供资源销毁接口


系统库补环境：直接调用setLibraryResolver  某些库函数调用在 unidbg 中只是 stub，不一定有完整行为

JNI补环境vm.setjni重写需要的jni方法

IOresolver提供虚拟文件系统

其他so依赖，加载依赖，手动触发jnionload


