# Re

## ez_reverse
> 加了3.96UPX壳的exe文件，查壳工具显示UPX（modified），使用十六进制编辑器打开，发现UPX被换为NEX，修改后使用IDA打开
> 查看strcmp的调用，迅速定位到TlsCallback_1_0，是程序的主程序
> ![alt text](image.png)
> Str1是输入，Str2内置的数据，是一串base64编码的字符串 'JGxBJHtXJWxsX1lvNF97bG9HX1hocGFBJV9Mb2dBbV9zNm9SYWNlX01lY2BBbGlDbF9Zbl9EJXR0a30='
> 解码得到$lA${W%ll_Yo4_{loG_XhpaA%_LogAm_s6oRace_Mec`AliCl_Yn_D%ttk}，一眼换表
> 看看编码表的交叉引用，在TlsCallback_0_0被修改
> ![alt text](image-1.png)
> 获得新编码表
> ![alt text](image-2.png)
> 

## 安？
> 这个题目花了好长时间，但很值，学会了ida调试so，虽然未知原因没有调试成功。
> 反编译java层代码，MainActivity的代码非常混乱，但关键的区域很明显
> ![alt text](image-3.png)
> 通过frida hook 应用，发现interfaceC0050e08存放了输入，interfaceC0050e09存放了提示框的内容，interfaceC0050e010存放了输入框上方的VIP状态
> 然而关键在于Hello方法，于是开始so层的逆向

> 但发现so层没有Hello方法，通过网上搜索，了解了在JIN_OnLoad动态注册,发现JIN_Fun就是Hello方法
> 一看这个Fun，格外混乱，函数一层包一层，一点看不懂，所以刚开始就想调试，毕竟这个apk就叫app debug
> 然后从零学如何IDA调试native层的库，做了很多操作，改了AndroidMainfest.xml的几个属性，试了两种调试策略，启动前开始调试，还有运行中调试，都失败了，甚至尝试新建一个android测试项目，来引入这个库调试，还找了很多反调试的痕迹，都没找到。最后归咎于我的设备问题，因为ida调试的时候，库都加载了，但线程出现问题，有些线程提前退出了，导致调试失败。
> 于是转向Frida调试，这个很顺利，也很熟练，但是不足以解出题目，只能再看回Fun
> ![alt text](image-4.png)
> 前面的是重点，后面对提示语的加密并不关心，通过字数就能判断是哪句
> 我很疑惑这个函数是怎么做到那么复杂的，一个简单的操作里面一层一层包了几十个函数，所以给不了ai分析，自己也一片混乱，希望出题的师傅指点一下是怎么做到的
> 后来直接猜，不进去函数里面了，sub_248C4就是比较两个字符串，sub_2465C就是个vector，等等。这样加密就变得清晰了，把输入的字符串依次和5777c异或，5777c在sub_246C8更新，最后和xmmword_1672E地址开始的43个字符比较
> 于是先hook出5777c的初始值，初值是0x49a1
> ![alt text](image-5.png)
> 然后写脚本解密
>![alt text](image-6.png)
> 运行得到flag{Th!5_is_7h3_f!rst_i_u5e_Ndk_aurfgkybi}
