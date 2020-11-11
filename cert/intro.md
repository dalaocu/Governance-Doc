# 组件介绍

WeBankBlockchain-Gov-Cert提供了证书生命周期管理的解决方案，规范证书签发流程，支持证书托管，支持多种签名算法，方便个人或企业使用。


## 1. 设计概要
WeBankBlockchain-Gov-Cert包含两个模块，cert-toolkit和cert-mgr，cert-toolkit作为证书生成工具，可作为独立工具包使用，cert-mgr基于cert-toolkit工具包，提供了证书的托管能力，并支持证书的生命周期管理，对签发的流程统一规范。

![](img/cert_framework.png)



## 2. 关键特性

### 1）支持多种密钥和签名算法
支持RSA，EC等密钥算法，支持SM2国密算法
支持SHA256WITHRSA、SHA256WITHECDSA、SM3WITHSM2等签名算法

### 2）支持证书托管
证书和子证书请求会持久化在数据库中，同时证书相关的私钥也会保存，对证书信息统一管理，可对证书进行多维度查询，并支持证书的导出


### 3）支持多级证书签发
证书可进行多级签发，可选择上级证书并请求签发，生成证书链，使用方便，操作便捷

![](img/cert_chain.png)

### 4）支持证书重置
支持对证书进行重置，重置信息包括有效时间、证书用途等


## 3. 场景示例

### 1) 链上节点准入证书管理

链上节点证书的签发统一由WeBankBlockchain-Gov-Cert来完成，Gov-Cert可以集成或者独立部署，并由权威机构来管理服务。在链初始化时可由部署者调用接口完成根证书的生成，新增机构或节点可以通过Gov-Cert提供的查询接口，来查询根证书，并提交子证书请求，根证书管理者可从通过查询请求列表，来获取准入请求，并选择签发子证书，子证书签发完成，下一级证书采取同样逻辑处理
通过WeBankBlockchain-Gov-Cert对于证书的管理，可以规范流程，提升效率，并保证证书安全

![](img/personal_use.png)

### 2) 证书工具包使用

WeBankBlockchain-Gov-Cert中cert-toolkit可作为独立JAVA工具包在项目中引用，代替命令行完成证书的生成和签发。企业或个人项目可集成WeBankBlockchain-Gov-Cert作为证书签发工具包

