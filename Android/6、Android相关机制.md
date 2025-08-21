# 1. 安卓虚拟机

参考：https://zhuanlan.zhihu.com/p/146863957

安卓虚拟机存在一个发展史：

## 1.1. JAVA虚拟机

JAVA代码会被编译为class文件，存放着基于栈的字节码，通过JAVA虚拟机执行。JAVA虚拟机不可用于Android系统。

## 1.2. Dalvik虚拟机

通过dx工具，将class文件转换为dex/odex文件，存放着基于寄存器的字节码，通过Dalvik虚拟机执行。

## 1.3. Dalvik虚拟机 + JIT

基于Dalvik虚拟机（libdvm.so），引入JIT机制，将热点函数从字节码转换为native code，加快执行速度。

缺点：每次应用启动均需要再次转换，电池等开销大。

## 1.4. ART虚拟机 + AOT编译

ART虚拟机（libart.so）在应用安装时将dex文件转换为oat文件（ELF格式），加快执行速度。

缺点：安装新应用和升级系统后优化应用，比较费时，且占用额外空间较多。

## 1.5. ART虚拟机 + AOT/JIT混合

应用安装时不再转换dex文件，而是在应用首次运行时使用JIT追踪热点函数，并在系统空闲时使用AOT将热点函数转换为oat文件。

## 1.6. 参考文献：
* ART Hook：https://bbs.kanxue.com/thread-270942.htm#msg_header_h1_4

# 2. 安卓进程管理

## 2.1. 两个特殊进程
* Zygote：所有Android进程的起点，其他APP进程均由该进程fork而来
* system_server：负责管理Android系统服务，如AMS（ActivityManagerService）,PMS（PackageManagerService）

# 3. 安卓权限管理

## 3.1. 权限管理方法
* AndroidManifest.xml：文件中声明权限
* ContextCompat.checkSelfPermission：检查权限
* ActivityCompat.requestPermissions：请求权限
* onRequestPermissionsResult：监听权限结果

## 3.2. 权限等级
* 普通权限：低风险，安装时自动授予，如INTERNET, ACCESS_NETWORK_STATE
* 危险权限：涉及隐私，用户需手动授权，如READ_CONTACTS, CAMERA, LOCATION
* 特殊权限：仅系统应用或特定签名的应用可使用，如MANAGE_EXTERNAL_STORAGE, SYSTEM_ALERT_WINDOW

# 4. APK加载流程
* APK解析：解析AndroidManifest.xml以确定权限、组件等信息
* DEX优化：将dex转换为OAT/ART文件，提升运行效率
* 类加载：使用ClassLoader机制加载Java类（PathClassLoader、DexClassLoader）
* 资源管理：解析resources.arsc以加载UI资源
* Native代码加载：加载lib/目录下的.so本地库
* 执行Application.onCreate()，启动应用
