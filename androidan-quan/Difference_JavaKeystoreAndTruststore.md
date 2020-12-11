 Java keystore 和Java truststore 区别与联系

- 对称加密：指的就是加、解密使用的同是一串密钥，所以被称做对称加密。 对称加密只有一个密钥作为私钥。 常见的对称加密算法：DES，AES等。

- 非对称加密：指的是加、解密使用不同的密钥，一把作为公开的公钥，另一把作为私钥。



在大多数情况下，当我们的应用程序需要通过SSL / TLS进行通信时，我们将使用java keystore和java truststore。

通常，这些是受密码保护的文件，与我们正在运行的应用程序位于同一文件系统上。

在Java 8之前，这些文件的默认格式为JKS(android .keystore 也是jsk格式的证书)。

从Java 9开始，默认的密钥库格式为PKCS12。

- JKS : 二进制格式，同时包含证书和私钥，一般有密码保护，只能存储非对称密钥对（私钥 + x509公钥证书）
- PKCS#12是公钥加密标准，通用格式（rsa公司标准）。微软和java 都支持。它规定了可包含所有私钥、公钥和证书。其以二进制格式存储，也称为 PFX 文件，在windows中可以直接导入到密钥区，注意，PKCS#12的密钥库保护密码同时也用于保护Key。

<font color=red>JKS和PKCS12之间的最大区别是JKS是Java专用的格式，而PKCS12是存储加密的私钥和证书的标准化且与语言无关的方式。</font>
![](assert/androidDebugKeystoreWarning.jpg)



[Java Cryptography Architecture Standard Algorithm Name Documentation for JDK 8](https://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html)


![](assert/KeyStoreTypes.jpg)


JKS证书创建 XXXXXX为证书密码

```bash
keytool -genkeypair -storetype JKS -keystore test.jks -storepass XXXXXX
```

PKCS12创建方式 XXXXXX为证书密码
```bash
keytool -genkeypair -keystore myKeystore.p12 -storetype PKCS12 -storepass XXXXXX


```
查看的keystore 证书公钥命令 XXXXXX为证书密码

```bash
keytool -list -rfc -keystore HOSAnyMarvel.jks -storepass XXXXXX
```

查看csr证书内容，csr为文本格式可直接打开，或使用openssl打开验证

```bash
openssl req  -text -in HOSAnyMarvel.csr
```
查看cer证书内容，cer为文本格式可直接打开，或使用openssl打开验证

```bash
keytool -printcert -file AnyMarvelHOS.cer
```

pkcs7 格式的内容查看

```bash
openssl pkcs7 -inform DER -in AnymarvelHOSDebug.p7b  -print_certs -text
```

文本格式查看信封内部的其他内容

```bash
vim HOSAnyMarvel.p7b
:%!xxd
```
