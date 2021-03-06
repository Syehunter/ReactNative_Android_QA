# ReactNative_Android_QA

##Android端10个最常见问题
这里逐条记录下最容易遇到的React native android 相关case：

* **1. app启动后，红色界面，unable load jsbundle ：**

1. 解决办法：一般来说就是，你是用dev-serve方式，且你的server没有正确匹配上，如果是用手机跑的话，需要pc和手机在同一个wifi下，且通过menu键设置menu-ip为pc的ip，如果是模拟器，则不需要手动设置ip,设置的话，反倒会出错

* **2. app启动后，红色界面，unRegisteredProject**

1. 提示提示什么，你的app没有在启动时候注册

2. 解决办法：这个后面也是一看就知道的错误，就是你的index.android.bundle中的最下面写的那个 

3. 'componetNameInYourLocalProject'在你的java代码中不是叫这个名字，自己check下，立刻就能修复
`AppRegistry.registerComponent('componetNameInYourLocalProject', () => JSObjAndroid);`

* **3. require（"xxx"）的组件失败**

1. js代码中有时候会出现require（"xxx"）的组件出错
解决办法：检测该node组件是否存在你的服务器上，如果是自己封装的NativeModule话可以直接使用

2. `var CustomMoudle = React.NativeModules.YourCustomModule
CustomMoudle.yourMethodDeclearInYourNative('someparms');`

* **4.调试**

1. 解决办法：可以利用pc端的chrome的 debug工具进行js端的调试，native的调试就只能用logcat跟踪了，目前看到大部分的错误都是自己代码的问题，ReactAndroid本身的Crash较少

* **5.so库的问题**

1. gradle的话，可以通到ndk filter来控制：

`	android {        	
        	defaultConfig {
	    	     ndk {
	 	     abiFilters "x86", "armeabi-v7a"
	            }
         }         `


2. maven的话，可以手动通过libs下的so拷贝来解决问题。

3. 这块有个比较大的坑就是，默认引入的jsc.aar中存在armabi文件夹，但是里面没有jsc.so 。导致在多个地方，去编码源码时ndk方面会报错。

* **6. 关于设备MinSdkVerison**

1. 默认Android要求4.1以上设备(4.0根据网络数据大概占比0.7比例，随着大部分app已经不支持4.0以下设备了，这块倒还可以接受)

2. 刚开始一直使用一个5.0的设备进行ReactAndorid的测试和开发，后来方向，其实搞上一个5.0+的Genymotion模拟器联调起来效率会更高。

* **7.UIExplorer demo问题**

1. 之前一直在看具体接入和代码实现方面的，当大头的工作回过头来看，其实当时应该先从这个UIExploror入手的话，效率和进度应该会有较大提高的。

2. 这块需要编译react源代码，如果遇到了https://github.com/facebook/react-native/issues/3976 的问题，可以使用我在下面回复的方法hook，但是本质原因还是那个armabi jsc.so的问题
![screenshot](http://img4.tbcdn.cn/L1/461/1/dfda3ab17e79df68f00b2ae2c18c24be062186c9)



* **8.能力覆盖范围**

1. 根据团队之前React iOS的经验，跟进主干代码，依赖RN本身提供的UI组件可以满足大部分业务场景。

2. 当然自己如果想复用之前团队沉淀下来的，配合着UIManager和UIModule这块本身工作量到也不算太大。

3. 但是应该尽可能的和团队以后的JS端和iOS端的协议接口保持一致，让React最大的意义发挥出来，“lean once run everywhere”

* **9.数据安全**

1. 0.14之前只支持dev-pc 和assert方式，从0.14.0 realease版本开始支持local file patch加载方式，最新版0.15.1。

2. 因为如果要动态能力，js必定是走网络端下发的，js本身是明文（即使JS做了混淆）,数据防劫持的保护还是必须要做的，这点可以配合https防篡改+sign校验来做

* **10.JNI消息轮训带来的影响**

1. 由于JNI的通信限制，Java层和Native通信是单向的，且为了保证RN的16ms的渲染频率，所有Java-Native-jscore层的通信都是异步的,这样可能对于JAVA层的UI渲染是个性能问题。

2. 当消息量非常大或ListView页面非常复杂时候，每1层Cell的渲染要以Css-ScrowllerView模型需要UI线程的连续绘制，对于瀑布流负责listview等可能会存在性能问题，但是该问题本身肯定是优于H5的体验的
