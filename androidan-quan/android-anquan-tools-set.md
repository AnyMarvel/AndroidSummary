#测试工具

- Appie(https://manifestsecurity.com/appie) – 轻量级的软件包, 可以用来进行基于Android的渗透测试, 不想使用VM的时候可以尝试一下.
- Android Tamer(https://androidtamer.com/) – 可以实时监控的虚拟环境, 可以用来进行一系列的安全测试, 恶意软件检测, 渗透测试和逆向分析等.
- AppUse(https://appsec-labs.com/AppUse/) – AppSec Labs开发的Android的虚拟环境.
- Mobisec(http://sourceforge.net/projects/mobisec/) – 移动安全的测试环境, 同样支持实时监控
- Santoku(https://santoku-linux.com/) – 基于Linux的小型操作系统, 提供一套完整的移动设备司法取证环境, 集成大量Adroind的调试工具, 移动设备取证工具, 渗透测试工具和网络分析工具等.

#逆向工程和静态分析

- APKInspector(https://github.com/honeynet/apkinspector/) – 带有GUI的安卓应用分析工具
- APKTool(http://ibotpeaches.github.io/Apktool/) – 一个反编译APK的工具，能够将其代码反编译成smali或者java代码，并且能后对反编译后的代码重新打包
- Dex2jar(https://github.com/pxb1988/dex2jar) – Dex2jar可以将.dex文件转换成.class文件或是将apt文件转换成jar文件.
- Oat2dex(https://github.com/testwhat/SmaliEx) – Oat2dex顾名思义和上一个工具类似, 用以将.oat文件转化为.dex文件.
- JD-Gui(http://jd.benow.ca/) – 用来反编译并分析class,jar
- FindBugs(http://findbugs.sourceforge.net/) ＋ FindSecurityBugs(http://findbugs.sourceforge.net/) – FindSecurityBugs是FindBugs的拓展, 可以对指定应用加载各种检测策略来针对不同的漏洞进行安全检查.
- YSO-Mobile Security Framework(http://findbugs.sourceforge.net/) – Mobile Security Framework (移动安全框架) 是一款智能、一体化的开源移动应用(Android/iOS)自动渗透测试框架，它能进行静态、动态的分析.
python manage.py runserver 127.0.0.1:1337
- Qark(https://github.com/linkedin/qark) – LinkedIn发布的开源静态分析工具QARK,该工具用于分析那些用Java语言开发的Android应用中的潜在安全缺陷.
- AndroBugs(https://github.com/AndroBugs/AndroBugs_Framework) – AndroBugs Framework是一个免费的Android漏洞分析系统，帮助开发人员或渗透测试人员发现潜在的安全漏洞, AndroBugs框架已经在多家公司开发的Android应用或SDK发现安全漏洞, Fackbook、推特、雅虎、谷歌安卓、华为、Evernote、阿里巴巴、AT&T和新浪等
- Simplify(https://github.com/CalebFenton/simplify) – Simplify可以用来去掉一些android代码的混淆并还原成Classes.dex文件, 得到.dex文件后可以配合Dex2jar或者JD-GUI进行后续还原
ClassNameDeobfuscator(https://github.com/HamiltonianCycle/ClassNameDeobfuscator) – 可以通过简单的脚本来解析smali文件
动态调试和实时分析

- Introspy-Android(https://github.com/iSECPartners/Introspy-Android) – 一款可以追踪分析移动应用的黑盒测试工具并且可以发现安全问题。这个工具支持很多密码库的hook，还支持自定义hook.
Cydia Substrate(http://www.cydiasubstrate.com/) – Cydia Substrate是一个代码修改平台.它可以修改任何主进程的代码,不管是用Java还是C/C （native代码）编写的, 一款强大而实用的HOOK工具
- Xposed Framework(http://forum.xda-developers.com/xposed/xposed-installer-versions-changelog-t2714053) – Xposed框架是一款可以在不修改APK的情况下影响程序运行（修改系统）的框架服务，基于它可以制作出许多功能强大的模块，且在功能不冲突的情况下同时运作.
CatLog(https://github.com/nolanlawson/Catlog) – Adroind日志查看工具, 带有图形界面
- Droidbox(https://github.com/nolanlawson/Catlog) – 一个动态分析android代码的的分析工具
- Frida(http://www.frida.re/) – Frida是一款基于python javascript 的hook与调试框架，通杀android\ios\linux\win\osx等各平台，相比xposed和substrace cydia更加便捷.
- Drozer(https://www.mwrinfosecurity.com/products/drozer/) – Drozer 是一个强大的app检测工具，可以检测app存在的漏洞和对app进行调试。

# 网络状态分析和服务端测试

- Tcpdump(http://www.androidtcpdump.com/) – 基于命令行的数据包捕获实用工具
- Wireshark(https://www.wireshark.org/download.html) – Wireshark（前称Ethereal）是一个网络封包分析软件。网络封包分析软件的功能是撷取网络封包，并尽可能显示出最为详细的网络封包资料
- Canape(http://www.contextis.com/services/research/canape/) – 可以对任何网络协议进行测试的工具
- Mallory(https://intrepidusgroup.com/insight/mallory/) – 中间人(MiTM)攻击工具, 可以用来监视和篡改网络内的移动设备和应用的网络流量数据. A Man in
- Burp Suite(https://portswigger.net/burp/download.html) – Burp Suite 是用于攻击web 应用程序的集成平台。它包含了许多工具，并为这些工具设计了许多接口，以促进加快攻击应用程序的过程。所有的工具都共享一个能处理并显示HTTP 消息，持久性，认证，代理，日志，警报的一个强大的可扩展的框架
- Proxydroid(https://play.google.com/store/apps/details?id=org.proxydroid) – Android ProxyDroid可以帮助的你设置Android设备上的全局代理（HTTP / SOCKS4 / SOCKS5）.
绕过Root检测和SSL的证书锁定

- Android SSL Trust Killer(https://github.com/iSECPartners/Android-SSL-TrustKiller) – 一个用来绕过SSL加密通信防御的黑盒工具, 功能支持大部分移动端的软件.
- Android-ssl-bypass(https://github.com/iSECPartners/android-ssl-bypass) – 命令行下的交互式安卓调试工具, 可以绕过SSL的加密通信, 甚至是存在证书锁定的情况下
- RootCoak Plus(https://github.com/devadvance/rootcloakplus) – RootCloak隐藏root是一款可以对指定的app隐藏系统的root权限信息.
其他安全相关的库

- PublicKey Pinning(https://www.owasp.org/images/1/1f/Pubkey-pin-android.zip) – 公钥锁定,
- Android Pinning(https://github.com/moxie0/AndroidPinning) – 一个独立开发的用于实现Android证书锁定的库. A standalone library project for certificate pinning on Android.
- Java AES Crypto(https://github.com/tozny/java-aes-crypto) – 一个用来加解密字符串的Android类, 目的是防止开发整使用不恰当的加密方式从而导致的安全风险
- Proguard(http://proguard.sourceforge.net/) – ProGuard是一个压缩、优化和混淆Java字节码文件的免费的工具，它可以删除无用的类、字段、方法和属性。可以删除没用的注释，最大限度地优化字节码文件。它还可以使用简短的无意义的名称来重命名已经存在的类、字段、方法和属性。常常用于Android开发用于混淆最终的项目，增加项目被反编译的难度.
- SQL Cipher(https://www.zetetic.net/sqlcipher/sqlcipher-for-android/) – SQLCipher是一个开源的SQLite扩展, 提供使用256-bit的AES加密来保证数据库文件的安全.
- Secure Preferences(https://github.com/scottyab/secure-preferences) – 用来加密Android上的Shared Preferences防止安全防护不足的情况下被窃取.
- Trusted Intents(https://github.com/guardianproject/TrustedIntents) – Library for flexible trusted interactions between Android apps.
