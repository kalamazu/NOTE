#在data，rdata寻找可疑数据

#花指令
>编译不成函数，需要除掉花指令

	C解释为代码，D解释为数据
	U取消定义， P成为一个函数
- 看到奇怪的指令，有可能是花指令
- 看到重复的指令，有可能是花指令
- 看到红色的交叉引用，有可能有花指令
## 常数
#常数
在加密算法中，许多特殊的常数被用于增强安全性和算法性能。以下是一些常见的加密常数及其应用场景：

---

### ​**1. CRC-32 多项式常数**

- ​**`0xEDB88321`**  
    反转后的 CRC-32 多项式，用于以太网、ZIP 等数据校验
    
    1
    
    。

---

### ​**2. SM3 算法的初始值**

- ​**`0x7380166f_4914b2b9_172442d7_da8a0600_a96f30bc_163138aa_e38dee4d_b0fb0e4e`**  
    中国国密算法 SM3 的初始常量，用于消息摘要计算
    
    1
    
    。

---

### ​**3. MD5 算法的幻数**

- ​**`0x67452301`, `0xEFCDAB89`, `0x98BADCFE`, `0x10325476`**  
    MD5 的四个初始缓冲区常量，用于哈希计算
    
    3
    
    。

---

### ​**4. SHA 系列的常数**

- ​**SHA-256 的初始哈希值**  
    如 `0x6A09E667`, `0xBB67AE85` 等，源自平方根和小数部分的前 32 位
    
    3
    
    5
    
    。
- ​**SHA-3 (Keccak) 的轮常数**  
    如 `0x0000000000000001`, `0x0000000000008082`，用于抗差分攻击
    
    6
    
    。

---

### ​**5. AES 的轮常数 (Rcon)**

- ​**`0x01`, `0x02`, `0x04`, `0x08`, `0x10`, ...**  
    用于 AES 密钥扩展，通过 xi−1 在 GF(28) 上生成
    
    4
    
    。

---

### ​**6. RSA 的公共指数**

- ​**`0x10001` (65537)**  
    常见的 RSA 公钥指数，平衡安全性与计算效率
    
    2
    
    。

---

### ​**7. TEA 算法的 δ 常数**

- ​**`0x9E3779B9`**  
    源自黄金分割率的 32 位整数，用于 TEA 加密算法的轮函数
    
    8
    
    。

---

### ​**8. 椭圆曲线密码学 (ECC) 参数**

- ​**NIST 曲线的系数**  
    如 P-256 曲线的素数模数 `0xFFFFFFFF00000001000000000000000000000000FFFFFFFFFFFFFFFFFFFFFFFF`
    
    7
    
    。