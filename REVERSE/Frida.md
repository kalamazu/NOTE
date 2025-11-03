


默认监听端口 27042

修改exe文件,传入函数地址，修改传入函数参数
```python
from __future__ import print_function

import frida

import sys

seesion=frida.attach("hello.exe")

script=seesion.create_script("""

   Interceptor.attach(ptr("%s"),{

   onEnter:function(args){

   args[0]=ptr("1234")}});

"""%int(sys.argv[1],16))

def on_message(message,data):

    print(message)

script.on('message',on_message)

script.load()

sys.stdin.read()
```


python hook代码
```python
import sys
import frida
import os
def load_js_file():

    with open("hook.js",'r',encoding='utf-8') as f:

        return f.read()

def on_message(message,data):

    print(f"[Frida] {message}&&{data}")
    

device=frida.get_usb_device()

print(f"已连接设备: {device.name}")

try:

    session = device.attach("TEST")

    print("[+] 附加进程成功")

except frida.ProcessNotFoundError:

    print("[-] 进程未找到，请确保应用已启动")
  
try:

    js_code=load_js_file()

except Exception as e:

    print(f"读取失败:{e}")

    sys.exit()

  
script=session.create_script(js_code)

script.on('message',on_message)

script.load()

  

sys.stdin.read()
```


java层hook js代码
```javascript
 Java.perform(function () {
 var targetclass=Java.use("pakagename")
 
 targclass.targetFunction.implementation=function(param){
 
 }
 
 targclass.targetFunction.overload('int').implementation=function(param){
 
 }

var field = targetclass.getclass().getDeclaredField("someattribute")
field.setAccessible(true)//获得修改实例类的权限
field.set()
field.get()


send(message,data)//发给python端一些信息

 }
```

native层hook js代码
```javascript
Java.perform(function(){
const modules=Process.enumerateModules()
const targetModule = modules.find(m => m.name.includes("libtest.so"));

so_name,fun_name
var targetfunc= Module.findExportByName(so_name, fun_name)


Interceptor.replace(
Module.getExportByName(so_name,fun_name)
new NativeCallback(
function(){  },'返回类型','参数类型'
)



)

})
```


native调试：
```JavaScript
Java.perform(function() {
//Java层hook
    var Check = Java.use('com.primite.drillbeam.Check');

     Check.calc.overload('java.lang.String').implementation = function(input) {

        console.log('[*] calc called with input: ' + input);


        var originalResult = this.calc(input);

        console.log('[*] Original return value: ' + originalResult);

        var modifiedResult = "8c1ee8d42e211d6b649e0b9c33bdc836fc912709";

        console.log('[*] Modified return value: ' +'8c1ee8d42e211d6b649e0b9c33bdc836fc912709');

        return modifiedResult;

    };


//native hook准备
    const libName = "libre0.so";

    const funcOffset = 0x184C; // 函数偏移量

   // 获取模块基址

    const libBase = Module.findBaseAddress(libName);

    const qword_64F0_addr = libBase.add(0x64F0);

    if (!libBase) {

        console.error(`[!] ${libName} not loaded`);

        return;

    }


    const targetFunc = libBase.add(funcOffset);

    console.log(`[+] ${libName} base: ${libBase}`);

    console.log(`[+] sub_184C address: ${targetFunc}`);

  
//native hook
    Interceptor.attach(targetFunc, {

        onEnter: function(args) {

            console.log("\n[=== sub_184C called ===]");

            // 参数列表（ARM64: x0-x3）

            const x0 = args[0];  // a1: 输入数据指针

            const x1 = args[1];  // n: 数据长度（uint64_t）

            const x2 = args[2];  // a3: 密钥指针（__int128*）

            const x3 = args[3];  // a4: 输出长度指针（uint64_t*）


            console.log(`x0=${x0}, x1=${x1}, x2=${x2}, x3=${x3}`);

	          //修改输入
  
           if (!x0.isNull() && x1.toInt32() > 0) {

    Memory.writeByteArray(x0, new Array(x1.toInt32()).fill(0)); // 直接覆写原内存

    console.log("Data overwritten to zeros");

}

  
  
			//读取输入
        if (!x0.isNull() && x1.toInt32() > 0) {

                const inputData = x0.readByteArray(x1.toInt32());

                console.log("Input data:", inputData);

            }

  //修改key

             if (!x2.isNull()) {

    Memory.writeByteArray(x2, new Array(x1.toInt32()).fill(0)); // 直接覆写原内存

    console.log("Data overwritten to zeros");

}

            // 读取密钥（假设 x2 是 128 位密钥指针）

            if (!x2.isNull()) {

                const key = x2.readByteArray(16); // __int128 = 16 bytes

                console.log("Key:", key);

            }

            const delta=qword_64F0_addr.readByteArray(0x10)

            console.log("delta:",delta)

  

            // 保存参数供 onLeave 使用

            this.x0 = x0;

            this.x1 = x1;

            this.x2 = x2;

            this.x3 = x3;

		//读取栈值
           
        const v26_stack_addr = this.context.sp.add(0xC8);

        this.v26_value = v26_stack_addr.readInt();

        console.log(`[*] v26 first value (from stack): 0x${this.v26_value.toString(16)}`);

        },

        onLeave: function(retval) {

            console.log("\n[=== sub_184C returned ===]");

            // 获取返回值（加密后的数据指针）

            const encryptedDataPtr = retval;

            console.log(`Return value (encrypted data ptr): ${encryptedDataPtr}`);

            const inputData = encryptedDataPtr.readByteArray(0x10);

            console.log("output data:", inputData);

  

            const delta=qword_64F0_addr.readByteArray(0x10)

            console.log("delta:",delta)

        }

    });

});
'''

