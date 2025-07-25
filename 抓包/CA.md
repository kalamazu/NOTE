相当于先和CA机构建立安全的通信，再通过CA机构验证服务端的可靠性

### 证书申请：
- 根据证书类型不同，可能需要提供附加材料：
    
    - **DV（域名验证）证书**：验证域名所有权（通过DNS解析、文件验证或邮箱验证）。
        
    - **OV（组织验证）证书**：提供企业营业执照等法律文件。
        
    - **EV（扩展验证）证书**：严格的身份核查（如律师函、电话确认等）。

### 证书签发：
- CA验证通过后，使用自身的**私钥**对申请者的公钥和身份信息进行签名，生成数字证书（通常为X.509格式）。
    
- 证书包含：
    
    - 申请者的公钥和身份信息
        
    - CA的签名算法（如SHA-256 with RSA）
        
    - 有效期（通常1-2年，Let's Encrypt为90天）
        
    - 扩展字段（如密钥用途、CRL分发点等）
分发：
    - - CA将签发的证书（如`certificate.crt`）返回给申请者。
    
	- 申请者需将证书与私钥部署到目标系统（如Web服务器、邮件服务器等）。

### 证书使用：





### 流程：

事前：CA建立信任锚点
服务端向CA验证自己可靠，获得了签发证书，相当于获得CA的认证

服务端内置了受信任的CA根证书，知道了哪些CA可信

事中：
服务端将CA证书发给客户端（经过根私钥加密过的）

客户端利用CA根证书（公钥）验证服务端证书签名是否合法

客户端生成会话密钥，用服务端证书中的公钥加密后传输