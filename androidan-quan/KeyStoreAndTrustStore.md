# 简介

KeyStore 和 TrustStore是JSSE中使用的两种文件。这两种文件都使用java的keytool来管理，他们的不同主要在于用途和相应用途决定的内容的不同。

这两种文件在一个SSL认证场景中，KeyStore用于服务器认证服务端，而TrustStore用于客户端认证服务器。

>比如在客户端(服务请求方)对服务器(服务提供方)发起一次HTTPS请求时,服务器需要向客户端提供认证以便客户端确认这个服务器是否可信。
这里，服务器向客户端提供的认证信息就是自身的证书和公钥，而这些信息，包括对应的私钥，服务器就是通过KeyStore来保存的。
当服务器提供的证书和公钥到了客户端，客户端就要生成一个TrustStore文件保存这些来自服务器证书和公钥。

KeyStore 和 TrustStore的不同，也主要是通过上面所描述的使用目的的不同来区分的，在Java中这两种文件都可以通过keytool来完成。不过因为其保存的信息的敏感度不同，KeyStore文件通常需要密码保护。

正是因为 KeyStore 和 TrustStore Java中都可以通过 keytool 来管理的，所以在使用时多有混淆。记住以下几点，可以最大限度避免这些混淆 :

- 如果要保存你自己的密码，秘钥和证书，应该使用KeyStore，并且该文件要保持私密不外泄，不要传播该文件;

- 如果要保存你信任的来自他人的公钥和证书，应该使用TrustStore，而不是KeyStore;

- 在以上两种情况中的文件命名要尽量提示其安全敏感程度而不是有歧义或者误导

  比如使用KeyStore的场景把文件命名为 truststore.jks,或者该使用TrustStore的情况下把文件命名为keystore.jks之类，这些用法都属于严重误导随后的使用者，有可能把比较私密的文件泄露出去；

- 拿到任何一个这样的文件时，确认清楚其内容然后决定怎样使用；

因为 KeyStore 文件既可以存储敏感信息，比如密码和私钥，也可以存储公开信息比如公钥，证书之类，所有实际上来讲，可以将KeyStore文件同样用做TrustStore文件,但这样做要确保使用者很明确自己永远不会将该KeyStore误当作TrustStore传播出去。

## KeyStore

内容
一个KeyStore文件可以包含私钥(private key)和关联的证书(certificate)或者一个证书链。证书链由客户端证书和一个或者多个CA证书。

KeyStore类型
KeyStore 文件有以下类型，一般可以通过文件扩展名部分来提示相应KeyStore文件的类型:

JCEKS
JKS
DKS
PKCS11
PKCS12
Windows-MY
BKS
以上KeyStore的类型并不要求在文件名上体现，但是使用者要明确所使用的KeyStore的格式。

## TrustStore
内容
一个TrustStore仅仅用来包含客户端信任的证书，所以，这是一个客户端***所信任的来自其他人或者组织的信息***的存储文件,而不能用于存储任何安全敏感信息，比如私钥(private key)或者密码。

客户端通常会包含一些大的CA机构的证书，这样当遇到新的证书时，客户端就可以使用这些CA机构的证书来验证这些新来的证书是否是合法的。

相关资料

[java-keystore-truststore-difference](https://www.baeldung.com/java-keystore-truststore-difference)

[KeyStores and TrustStores](https://docs.oracle.com/cd/E19509-01/820-3503/ggffo/index.html)

[Difference between keystore and truststore](https://www.pixelstech.net/article/1488632244-Difference-between-keystore-and-truststore)

[Different types of keystore in Java – Overview](https://www.pixelstech.net/article/1408345768-Different-types-of-keystore-in-Java----Overview)

[Java Cryptography Architecture Standard Algorithm Name Documentation for JDK 8](https://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html)
