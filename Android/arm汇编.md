
```text
<opcode> {<cond>} {S} <Rd>,<Rn>,<shifter_operand>
```
- `opcode`: 为指令，在我们第一句的指令是 `sub` ,表示减法。
- `Rd`: 为指令操作目的寄存器，在我们第一句中是 `sp` 寄存器。
- `Rn`: 为指令第一源操作数，在我们第一句中是 `sp` 寄存器
- `shifter_operand`: 数据处理指令，这里我们第一句是立即数寻址，即 `#32`。
33个寄存器 31个通用寄存器  sp，pc
32位：w0-w30
64位：x0-x30
第31个专用寄存器的访问方式有4种:
- 当将其作为 `32bit` 栈帧指针寄存器（`stack pointer`) 的时候，使用 `WSP` 来引用它。
- 当将其作为 `62bit` 栈帧指针寄存器（`stack pointer`) 的时候，使用 `SP` 来引用它。
- 当将其作为 `32bit` 零寄存器( `zero register` )的时候，使用 `WZR` 来引用它。
- 当将其作为 `62bit` 零寄存器( `zero register` )的时候，使用 `ZR` 来引用它。

x0保存返回值，以及传参
x1-x7传参
x8也可以保存返回值
x9-x28通用
x29 fp 保存栈底地址
x30 lr 保存返回地址
x31 sp 保存栈顶地址
x31 zr 恒为0
x32 pc 指向当前指令地址

r0-r15
r13栈指针
r14链接寄存器LR
r15程序计数器

专用寄存器：CPSR SPSRs 
CPSR32位寄存器，存储程序状态，ALU标志，条件码，中断使能位置，执行模式位
SPSR保存中断前CPSR的值

bl调用函数
st开头从寄存器存到内存去
ls开头从内存存到寄存器来

B Label
BX Rm

BEQ 
BNE

BL Label函数调用，返回地址保存到LR R14

MRS Rd，CPSR
MSR CPSR ，Rn