公钥、私钥、数字签名(签名)、数字证书(证书) 的关系(图文)

1. 鲍勃有两把钥匙，一把是公钥，另一把是私钥。

![](https://upload-images.jianshu.io/upload_images/8031371-c57a039a914ce1b7.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

2. 鲍勃把公钥送给他的朋友们----帕蒂、道格、苏珊----每人一把。

![](https://upload-images.jianshu.io/upload_images/8031371-ede0d9c0046b8b9c.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

3. 苏珊要给鲍勃写一封保密的信。她写完后用鲍勃的公钥加密，就可以达到保密的效果

![](https://upload-images.jianshu.io/upload_images/8031371-32369164a35ab2bb.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

4. 鲍勃收信后，用私钥解密，就看到了信件内容。这里要强调的是，只要鲍勃的私钥不泄露，这封信就是安全的，即使落在别人手里，也无法解密。


![](https://upload-images.jianshu.io/upload_images/8031371-6de0479bd9ee00cb.png?imageMogr2/auto-orient/strip|imageView2/2/w/600/format/webp)

5. 鲍勃给苏珊回信，决定采用** "数字签名"**。他写完后先用Hash函数，生成信件的摘要（digest）


![](https://upload-images.jianshu.io/upload_images/8031371-2d8c8ef0afb7c7e7.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

6. 然后，鲍勃使用私钥，对这个摘要加密，生成"数字签名"（signature）。

![](https://upload-images.jianshu.io/upload_images/8031371-d2caa5b33c5d549d.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

7. 鲍勃将这个签名，附在信件下面，一起发给苏珊。


![](https://upload-images.jianshu.io/upload_images/8031371-e069eef89b0add41.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

8. 苏珊收信后，取下数字签名，用鲍勃的公钥解密，得到信件的摘要。由此证明，这封信确实是鲍勃发出的。

![](https://upload-images.jianshu.io/upload_images/8031371-d9b73d96e653880f.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

9. 苏珊再对信件本身使用Hash函数，将得到的结果，与上一步得到的摘要进行对比。如果两者一致，就证明这封信未被修改过。


![](https://upload-images.jianshu.io/upload_images/8031371-dceb30e4ce1bdec2.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

10. 复杂的情况出现了。道格想欺骗苏珊，他偷偷使用了苏珊的电脑，用自己的公钥换走了鲍勃的公钥。此时，苏珊实际拥有的是道格的公钥，但是还以为这是鲍勃的公钥。因此，道格就可以冒充鲍勃，用自己的私钥做成"数字签名"，写信给苏珊，让苏珊用假的鲍勃公钥进行解密。

![](https://upload-images.jianshu.io/upload_images/8031371-95aff68208e7e904.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

11. 后来，苏珊感觉不对劲，发现自己无法确定公钥是否真的属于鲍勃。她想到了一个办法，要求鲍勃去找"证书中心"（certificate authority，简称CA），为公钥做认证。证书中心用自己的私钥，对鲍勃的公钥和一些相关信息一起加密，生成"数字证书"（Digital Certificate）。

![](https://upload-images.jianshu.io/upload_images/8031371-6efe1fe10f1bc2b7.png?imageMogr2/auto-orient/strip|imageView2/2/w/650/format/webp)

12. 鲍勃拿到数字证书以后，就可以放心了。以后再给苏珊写信，只要在签名的同时，再附上数字证书就行了。

![](https://upload-images.jianshu.io/upload_images/8031371-b53f062a5453f230.png?imageMogr2/auto-orient/strip|imageView2/2/w/549/format/webp)

13. 苏珊收信后，用CA的公钥解开数字证书，就可以拿到鲍勃真实的公钥了，然后就能证明"数字签名"是否真的是鲍勃签的。

![](https://upload-images.jianshu.io/upload_images/8031371-bb5f8192a86a8dde.png?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)
