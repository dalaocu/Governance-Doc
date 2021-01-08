# 接口示例

CertManagerService类是证书管理的统一入口，覆盖证书管理的全生命周期，包含如下功能：

*   createRootCert : 生成根证书
*   createRootCertByHexPriKey ：私钥Hex格式作为入参生成证书
*   createCertRequest：生成请求
*   createCertRequestByHexPriKey ：私钥Hex格式作为入参生成请求
*   createChildCert：生成子证书
*   resetCertificate：证书重置
*   queryCertList：证书列表查询
*   queryCertRequestList：请求列表查询
*   queryCertKeyList：证书私钥列表查询
*   queryCertInfoByCertId：根据id查询证书
*   queryCertRequestByCsrId：根据id证书请求
*   exportCertToFile：证书导出

## createRootCert

生成根证书，提供了多种封装接口，可按需使用

```

    public void testCreateRootCert0() throws Exception{
        X500NameInfo issuer = X500NameInfo.builder()
                .commonName("chain")
                .organizationName("fisco-bcos")
                .organizationalUnitName("chain")
                .build();
        String userId = "bob";
        CertVO cert = certManagerService.createRootCert(userId,issuer);
        System.out.println(cert);
    }


    public void testCreateRootCert1() throws Exception{
        X500NameInfo issuer = X500NameInfo.builder()
                .commonName("chain")
                .organizationName("fisco-bcos")
                .organizationalUnitName("chain")
                .build();
        String userId = "bob";
        Date beginDate = new Date();
        Date endDate = new Date(beginDate.getTime() + CertConstants.DEFAULT_VALIDITY);
        CertVO cert = certManagerService.createRootCert(userId,issuer,beginDate,endDate);
    }


    public void testCreateRootCert3() throws Exception{
        X500NameInfo issuer = X500NameInfo.builder()
                .commonName("chain")
                .organizationName("fisco-bcos")
                .organizationalUnitName("chain")
                .build();
        String userId = "bob";
        Date beginDate = new Date();
        Date endDate = new Date(beginDate.getTime() + CertConstants.DEFAULT_VALIDITY);
        KeyUsage keyUsage = new KeyUsage(KeyUsage.dataEncipherment);
        CertVO cert = certManagerService.createRootCert(userId,1,issuer,keyUsage,beginDate,endDate);
    }

     
    public void testCreateRootCert4() throws Exception{
        X500NameInfo issuer = X500NameInfo.builder()
                .commonName("chain")
                .organizationName("fisco-bcos")
                .organizationalUnitName("chain")
                .build();
        String userId = "bob";
        Date beginDate = new Date();
        Date endDate = new Date(beginDate.getTime() + CertConstants.DEFAULT_VALIDITY);
        String pemPriKey = "此处填入私钥";
        CertVO str = certManagerService.createRootCert(userId,pemPriKey,KeyAlgorithmEnums.RSA,issuer,beginDate,endDate);
    }
```

执行过后，会生成根证书并保存

**涉及参数说明**：

- userId: 用户id

- issuer: 签发者信息

- beginDate：证书生效时间

- endDate：证书失效时间

- keyUsage：证书用途

- certKeyId：证书签名私钥id


## createRootCertByHexPriKey

私钥Hex格式作为入参生成根证书

```
     
    public void testCreateRootCertByHexPriKey() throws Exception{
        X500NameInfo issuer = X500NameInfo.builder()
                .commonName("chain")
                .organizationName("fisco-bcos")
                .organizationalUnitName("chain")
                .build();
        String userId = "bob";
        Date beginDate = new Date();
        Date endDate = new Date(beginDate.getTime() + CertConstants.DEFAULT_VALIDITY);
        KeyPair keyPair = KeyUtils.generateKeyPair();
        String hexPriKey = Numeric.toHexString(keyPair.getPrivate().getEncoded());
        CertVO cert = certManagerService.createRootCertByHexPriKey(userId,hexPriKey,KeyAlgorithmEnums.RSA,issuer,beginDate,endDate);
    }
```
执行过后，会生成根证书并保存

**涉及参数说明**：

- userId: 用户id

- issuer: 签发者信息

- beginDate：证书生效时间

- endDate：证书失效时间

- hexPriKey：证书签名私钥Hex格式


## createCertRequest

生成用于生成子证书的请求，提供了两个封装接口，可按需使用

```
     
    public void testCreateCertRequest0() throws Exception{
        X500NameInfo subject = X500NameInfo.builder()
                .commonName("agancy")
                .organizationName("fisco-bcos")
                .organizationalUnitName("agancy")
                .build();
        String userId = "bob1";
        CertRequestVO csr;
        csr = certManagerService.createCertRequest(userId,1, subject);
    }
     
    public void testCreateCertRequest1() throws Exception{
        X500NameInfo subject = X500NameInfo.builder()
                .commonName("agancy")
                .organizationName("fisco-bcos")
                .organizationalUnitName("agancy")
                .build();
        String userId = "bob1";
        KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("ECDSA", "BC");
        ECGenParameterSpec ecGenParameterSpec = new ECGenParameterSpec("secp256k1");
        keyPairGenerator.initialize(ecGenParameterSpec, new SecureRandom());
        KeyPair keyPair = keyPairGenerator.generateKeyPair();
        PrivateKey privateKey = keyPair.getPrivate();

        CertRequestVO csr = certManagerService.createCertRequest(userId, CertUtils.readPEMAsString(privateKey),
                KeyAlgorithmEnums.ECDSA,1,subject);
    }

```
执行过后，会生成请求并保存

**涉及参数说明**：

- userId: 用户id

- subject: 请求方信息

- issuerCertId: 签发证书id

- privateKey：请求签名私钥串

- certKeyId：请求签名私钥id


## createCertRequestByHexPriKey

私钥Hex格式作为入参生成请求

```   
    public void testCreateCertRequestByHexPriKey() throws Exception{
        X500NameInfo subject = X500NameInfo.builder()
                .commonName("agancy")
                .organizationName("fisco-bcos")
                .organizationalUnitName("agancy")
                .build();
        String userId = "bob";
        String hexPriKey = "3500db68433dda968ef7bfe5a0ed6926b8e85aabcd2caa54f8327ca07ac73526";
        CertRequestVO cert = certManagerService.createCertRequestByHexPriKey(userId,hexPriKey,KeyAlgorithmEnums.ECDSA,3,subject);
    }

```
执行过后，会生成请求并保存

**涉及参数说明**：

- userId: 用户id

- subject: 请求者信息

- issuerCertId: 签发证书id

- keyAlg: 密钥算法

- hexPriKey：证书签名私钥Hex格式


## createChildCert

生成子证书

```  
    public void testCreateChildCert() throws Exception{
        String userId = "bob1";
        String child;
        CertVO = certManagerService.createChildCert(userId,4);
    }
```
执行过后，会生成子证书并保存

**涉及参数说明**：

- userId: 用户id

- csrId: 请求id


## resetCertificate

证书重置

```    
    public void testResetCertificate() throws Exception{
        String userId = "bob1";
        Date beginDate = new Date();
        Date endDate = new Date(beginDate.getTime() + CertConstants.DEFAULT_VALIDITY);
        CertVO root = certManagerService.resetCertificate(userId,9,
                new KeyUsage(KeyUsage.dataEncipherment),
                beginDate,endDate);
    }
```

执行过后，会重置证书并保存

**涉及参数说明**：

- userId: 用户id

- certId: 重置证书id

- keyUsage：证书用途

- beginDate：证书生效时间

- endDate：证书失效时间


## queryCertList

证书列表查询，多条件联合查询

```  
    public void testQueryCertList() {
        String userId = "bob";
        List<CertVO> list = certManagerService.queryCertList(
                userId,null,null,null,null,null);
    }
```

执行过后，会得到证书列表

**涉及参数说明**：

- userId: 用户id

- issuerKeyId: 签发私钥id

- pCertId：签发证书id

- issuerOrg：签发机构名

- issuerCN：签发者公共名称

- isCACert：是否ca机构


## queryCertRequestList

证书请求查询，多条件联合查询

```
    public void testQueryCertRequestList() {
        String userId = "bob";
        List<CertRequestVO> list = certManagerService.queryCertRequestList(
                userId,null,null,null,null,null);
    }
```

执行过后，会得到证书请求列表

**涉及参数说明**：

- userId: 用户id

- subjectKeyId: 请求签名私钥id

- pCertId：签发证书id

- subjectOrg：申请机构名

- subjectCN：申请者公共名称


## queryCertKeyList

证书私钥查询，会返回私钥列表，但不返回私钥明文

```
    public void testQueryCertKeyList() {
        String userId = "bob";
        List<CertKeyVO> list = certManagerService.queryCertKeyList(userId);
    }
```

执行过后，会得到证书私钥列表

**涉及参数说明**：

- userId: 用户id


## queryCertInfoByCertId

根据id查询证书

```
    public void testQueryCertInfoByCertId() {
        CertVO certInfo = certManagerService.queryCertInfoByCertId(1L);
    }    
```

执行过后，会得到证书

**涉及参数说明**：

- certId: 证书id

## queryCertRequestByCsrId

根据id查询证书请求

```
    public void testQueryCertRequestByCsrId() {
        CertRequestVO keyRequestVO = certManagerService.queryCertRequestByCsrId(1L);
    }  
```

执行过后，会得到证书请求

**涉及参数说明**：

- csrId: 证书请求id


## exportCertToFile

证书导出

```
    public void testExportCertToFile() throws Exception {
        certManagerService.exportCertToFile(1L,"src/ca.crt");
    }
```

执行过后，证书导出到执行文件目录

**涉及参数说明**：

- certId: 证书id

- filePath: 证书导出路径