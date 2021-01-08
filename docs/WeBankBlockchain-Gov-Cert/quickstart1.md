# cert-toolkit使用

## 功能介绍
cert-toolkit用于证书生成。支持轻量级jar包接入。

支持如下方式：
*   根证书生成
*   证书请求生成
*   子证书生成   
*   证书文件的读写

## 前置依赖

在使用本组件前，请确认系统环境已安装相关依赖软件，清单如下：

| 依赖软件 | 说明 |备注|
| --- | --- | --- |
| Java | JDK[1.8] | |
| Git | 下载的安装包使用Git | |

如果您还未安装这些依赖，请参考[附录](../appendix.md)。



## 部署说明

目前支持从源码进行部署。

### 获取源码

通过git下载源码：

```
https://github.com/WeBankBlockchain/Gov-Cert.git
```

进入目录：
```
cd Gov-Cert/cert-toolkit
```

### 编译源码

方式一：如果服务器已安装Gradle
```
gradle build -x test
```

方式二：如果服务器未安装Gradle，使用gradlew编译
```
chmod +x ./gradlew && ./gradlew build -x test
```

### 导入jar包

cert-toolkit编译之后在cert-toolkit目录下会生成dist文件夹，文件夹中包含cert-toolkit.jar。可以将cert-toolkit.jar导入到自己的项目中，例如libs目录下。然后进行依赖配置。gradle依赖配置如下，然后再对自己的项目进行编译。

```
repositories {
    mavenCentral()
    mavenLocal()
    maven {
        url "http://maven.aliyun.com/nexus/content/groups/public/"
    }
}

dependencies {
    compile fileTree(dir:'libs',include:['*.jar'])
}

```

### 接口说明

cert-toolkit中包含若干类服务接口，如下，接口使用可以通过new对象然后调用

- CertService功能：证书的生成

```

CertService certService = new CertService();

```

CertService提供了三种功能接口：
- createRootCertificate：生成根证书，即自签名证书
- createCertRequest：生成证书请求
- createChildCertificate：生成子证书

为方便调用，针对上述三个接口封装了默认配置（签名算法：SHA256WITHRSA,有效期10年）的生成接口：
- generateRootCertByDefaultConf：生成根证书
- generateCertRequestByDefaultConf：生成证书请求
- generateChildCertByDefaultConf：生成子证书
- generateKPAndRootCert：生成密钥对和根证书


### 示例说明

下面介绍下证书的生成流程

##### 父证书生成

使用generateKPAndRootCert方法：自动生成私钥（默认为RSA），根据生成的私钥自签名生成根证书，并写入指定路径，写入文件默认为ca

示例代码如下：

```
    CertService certService = new CertService();
    X500NameInfo info = X500NameInfo.builder()
            .commonName("chain")
            .organizationName("fisco-bcos")
            .organizationalUnitName("chain")
            .build();
    certService.generateKPAndRootCert(info,"out");
```

执行上述方法，会在控制台打印出证书、私钥文件保存结果和路径，证书会保存在out/ca/ca.cert文件中

##### 子证书csr生成

csr全称为Certificate Signing Request，即证书请求文件，根（父）证书通过其私钥对请求文件签名，颁发子证书。

使用下述可方法快速生成csr

示例代码如下：

```
    CertService certService = new CertService();
    X500NameInfo info = X500NameInfo.builder()
            .commonName("chain")
            .organizationName("fisco-bcos")
            .organizationalUnitName("chain")
            .build();
    //自动生成RSA私钥，KeyUtils为证书组件密钥工具类
    KeyPair keyPair = KeyUtils.generateKeyPair();
    //CertUtils工具提供了证书读写解析的相关能力
    String priStr = CertUtils.readPEMAsString(keyPair.getPrivate());
    String csrStr = certService.generateCertRequestByDefaultConf(info, priStr, "out/child/child.csr");
    System.out.println(csrStr);
```

执行上述方法会在控制台打印出csr文件内容，并写入out/child/child.csr文件中

##### 子证书颁发

通过根证书和其私钥对子证书申请进行签发

入参可以采用多种方式，第一种方式为查看保存证书和私钥相关文件的内容，复制到下述参数中；第二种方式为将对应文件路径作为参数传入，

示例代码如下（选择一种执行即可）：

```
    //第一种方式：参数为生成相关文件路径
    CertService certService = new CertService();
    String childStr2 = certService.generateChildCertByDefaultConf("out/ca/ca.crt","out/child/child.csr","out/ca/ca_pri.key", "out/child/child.crt");
    System.out.println(childStr2);
```

```
    //第二种方式：参数为对应字符串
    CertService certService = new CertService();
    String caKey = "[复制out/ca/ca_pri.key中内容到此处]";
    String caStr = "[复制out/ca/ca.cert中内容到此处]";
    String csrStr = "[复制out/child/child.csr中内容到此处]";
    String childStr = certService.generateChildCertByDefaultConf(caStr,csrStr,caKeym,"out/child/child.crt");
    System.out.println(childStr);

```

执行上述方法会在控制台打印出子证书内容,并写入out/child/child.crt文件中，可从第二步开始，继续下一级证书的签发。



##### 更多使用方式

参照[Java API](./javadoc/toolkitdoc/index.md)
