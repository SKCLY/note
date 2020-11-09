# 崩溃分析

1. 第三方对接  bugly，友盟等
2. 本地日志系统
   1. 实现 Thread.UncaughtExceptionHandler 接口，在uncaughtException可以捕获异常，打印并记录下来
   2. 对接阿里云日志服务上传日志信息，方便查找异常。信息内容如下
      * 发生时间
      * 手机型号
      * Android系统版本
      * 应用版本号
      * 异常信息栈
      * 用户行为（记录接口信息，点击页面）
      * onTrimMemory()方法内去释放部分内存
   3. 最好是后台服务日志对比查看
3. 查看trace.txt文件









