
### FART
[原理](https://xialuohun.top/posts/android/android%E5%A3%B3%E4%B8%96%E7%95%8C/fart%E6%B5%81%E7%A8%8B%E5%92%8C%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90/)
主动调用的方式

在Activity启动时插入钩子，启动脱壳线程
修改方法调用入口，判断是否为FART调用，若是，则dump流程而不是执行

**绕过Libc Hook**​：在Native层执行Dump操作时，FART**直接使用底层的系统调用**

### DexExtractor

修改Android系统底层源码，再Dex文件被加载的关键时刻自动将其从内存抓取出来

修改路径：修改ART/Dalvik虚拟机的 `DexFile.cpp` 中的 ​**`dexFileParse`函数**，这是系统解析Dex文件的必经之路

对抗检测：将Dump下来的Dex数据先进行**Base64编码**再写入文件，以绕过加固方案对直接内存Dump的检测

部署方式：修改system.img
