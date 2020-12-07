# 深入理解--Android Loader

开发 [漫品](http://aimanpin.com/) 客户端的过程中，需要获取到手机目录中所有的txt文件进行展示用于提供给的用户进行
如果使用Java读取目录,目前想到的是递归的方式进行文件获取，但获取过程其实是比较缓慢的,基于获取到的文件再向下进行传递。如果手机文件较多，内容较多的话，这并不是一个好的选中，
也许查找时间会非常的长。
```Java
private void searchBook(ObservableEmitter<File> e, File parentFile) {
    if (null != parentFile && parentFile.listFiles() != null && parentFile.listFiles().length > 0) {
        File[] childFiles = parentFile.listFiles();
        for (int i = 0; i < childFiles.length; i++) {
            if (childFiles[i].isFile() && childFiles[i].getName().substring(childFiles[i].getName().lastIndexOf(".") + 1).equalsIgnoreCase("txt") && childFiles[i].length() > 100 * 1024) {   //100kb
                e.onNext(childFiles[i]);
                continue;
            }
            if (childFiles[i].isDirectory() && childFiles[i].listFiles().length > 0) {
                searchBook(e, childFiles[i]);
            }
        }
    }
}

```

https://blog.csdn.net/u010355144/article/details/46356909

https://segmentfault.com/a/1190000007132972

https://developer.android.com/guide/components/loaders?hl=zh-cn

https://www.jianshu.com/p/8b8197dc2e04

https://www.cnblogs.com/qianguyihao/p/4110510.html
