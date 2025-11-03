https://tool.kanxue.com/index-detail-47.htm
https://gal2xy.github.io/2025/03/31/%E5%8A%A0%E5%A3%B3%E4%B8%8E%E8%84%B1%E5%A3%B3/VMP%E5%85%A5%E9%97%A8%E2%80%94%E2%80%941.81%20Demo%E5%88%86%E6%9E%90/

数据结构：
VIP
VSP
VM stack
VM Register/VM Context
VM Instruction
IMM 立即数
VM handler
VM Opcode
VM handler table
VM Opcode table


1.将真实环境的上下文入栈
2.更加VIP获取下一条虚拟指令操作码，根据操作码跳转handler
3.执行handler，更新指针
4.检查堆栈，空间是否充足
5.执行完全部字节码
6.还原真实环境上下文，退出虚拟机