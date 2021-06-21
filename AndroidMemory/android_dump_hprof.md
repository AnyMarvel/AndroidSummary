
命令行提取hprof文件

1. #adb shell am dumpheap <processname> <FileName>

持续监听：

```bash

while true;  do adb shell dumpsys com.taobao.taobao ; sleep ;

```

2. #adb pull <FileName> <Dir>

3. #/android-sdk/platform-tools/hprof-conv <FileName> <newFileName>.hprof

4. 使用MAT分析(Android studio profile)


