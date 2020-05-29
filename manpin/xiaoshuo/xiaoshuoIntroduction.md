漫品客户端 全站开源 开源地址:

https://github.com/AnyMarvel/ManPinAPP

欢迎start

小说模块简介

APP本地小说仅支持txt格式，将小说按章节分割存入数据库，在线小说来源是通过手机模拟小说网站的GET请求，获取网站源码，根据不同网站来源进行数据解析，获取相应数据存入数据库显示（已经匹配了近30个网站），同时为了提升阅读体验，章节内容做了二级缓存处理。同时也增加了离线加载功能。

![](/manpin/assert/manpinxiaoshuo.png)

免费小说阅读器,主打精简，UI精简但不失优雅，功能精简但不失体验，根据功能界面划分为：

- 书架模块：包含本地书籍，以及网络在线书籍。
- 书城：书城分为导航栏,推荐页及主页三部分数据,为保证APP的稳定性,这三部分都为动态加载模块,获取网络链接可用状态则展示当前网站链接内容,获取失败则更换,能够最大限度的保证在线阅读的可用性.最后以一个网站为数据来源，解析其主页数据，筛选以及封装数据以Android原生界面的形式展现出来。
- 网络小说离线功能：通过提前设置任务队列，通过Service后台获取章节数据。
- 本地小说：将手机本地的txt小说导入应用。
- 小说阅读模块。
　　本APP所有数据来源于第三方小说网站，不具备自身后台，通过JSoup对xml进行数据解析，来完成用户对小说内容的获取。

#开发难点

##一. 本地超大txt小说数据处理

手机直接读取超大文本时，不做好优化是很可能OOM的。

本地小说的处理方式是:

![](/manpin/assert/xiaoshuoOOM.png)

可查看ImportBookModelImpl类,查看具体处理方式

##二. 小说阅读模块

小说阅读页面基于自定义ViewGroup实现，最多只有3个页面，分别是当前页，上一页，下一页。当滑动到下一页时，上一页移除，当前页指向下一页，同时再新增下一页。保证UI布局数量不会越来越多，杜绝因为View过多而产生的OOM。

　　同时阅读时，章节内容数据优先从内存读取，随后是数据库，都没有的话，再通过章节的网络地址去请求新的章节再解析最后返回数据，存入缓存以及数据库中。

## 三. 目标网站不稳定问题

小说模块中目前内置30多个小说网站,基于漫品APP获取小说的途径主要由以下四个途径:

1. 搜索
目前漫品客户端包含30+个网络,遍历搜索相关内容进行展示,流程如下

![](/manpin/assert/xiaoshuosousuo.png)

内置网站搜索集合,逐个获取关键字搜索内容,获取失败或超时则使用下一个搜索引擎进行搜索,直到搜索成功为止

2. 书城页面导航推荐

书城页面导航页面为单个小说网站导航页面内容,导航内容为动态绘制,导航绘制分为以下几个步骤:
- 获取可用导航网站资源(导航网站列表或的定时替换)
- 绘制当前导航

3. 主页及推荐页面

逻辑与导航推荐逻辑相符

效果图如图所示

![](https://github.com/AnyMarvel/ManPinAPP/blob/master/pictures/xiaoshuo_4.jpeg)![](https://github.com/AnyMarvel/ManPinAPP/blob/master/pictures/xiaoshuo_3.jpeg)![](https://github.com/AnyMarvel/ManPinAPP/blob/master/pictures/xiaoshuo_2.jpeg)![](https://github.com/AnyMarvel/ManPinAPP/blob/master/pictures/xiaoshuo_1.jpeg)


漫品客户端 全站开源 开源地址:

https://github.com/AnyMarvel/ManPinAPP

欢迎start
