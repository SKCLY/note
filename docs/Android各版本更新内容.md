# Android历代版本

#### Android 11

* 在获取地理位置，麦克风和摄像头时增加了一次性权限访问选项
* 位置权限增加了前台和后台的区别。
* 在应用中通过`` PackageManger.getInstalledPackages()``查看设备安装应用包信息，现在只能查看自身应用信息，无法查看第三方应用安装包信息。如果需要应在manifest中单独声明要查询的应用
* 分区存储。在10的基础上变成必需使用分区存储。其他应用只能访问当前应用的照片，视频等媒体文件，其他私有文件都不能访问
* READ_PHONE_STATE权限将只能获取电话相关的状态，不能获取电话号码。如需要获取电话号码，需要额外申请 READ_PHONE_NUMBERS 权限。
* 发布了Android Studio 4.0
  * 混淆代码有智能提示功能
  * Layout Validation，可以预览布局在多个屏幕上的UI效果
  * Layout Insepector，可以查看布局结构和层次，可以优化布局。

