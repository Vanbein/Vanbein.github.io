## 初级阶段:

### Objective-C

1. Foundation框架、Catagory、KVC、KVO、Protocol、Block、引用计数等基本知识有有所掌握。
2. 注意代码规范。

### iOS开发的通用控件

1.UIView方面相关控件（UILabel、UIImageView、UIButton、UISlider、UISwitch、UIScrollView（TableView、CollectionView、TextView）、UIWebView、UIWindow、UINavigationBar、UITabBar）。
2.了解UIViewController的生命周期，Navigation的堆栈原理等等。
3.千万别只用代码写UI，把Xib、storyborad搞搞清楚，特别是AutoLayout用好来也很重要。
4.NSNotifaication，怎么说，也是个大头，单独拿出来好好研究下。
5.手势。UITapGestureRecognizer、UIPanGestureRecognizer、UILongPressGestureRecognizer、UISwipeGestureRecognizer、UIRotationGestureRecognizer。
6.屏幕的旋转，这个坑要多走走才过的好。
7.Localizations
8.文件管理，Bundle、NSFileManager。
9.数据存储，UserDefault，KeyChain、NSKeyedArchiver。
10.ARC


### iOS动画：
1.UIView动画封装
2.Controller 相关的TransitionStyle
3.CAlayer。

### 设计模式：
这部分内容研究，建议拿一些开源企业级框架进行学习。本人当时是哪BeeFramework上手，Bee框架算是很好的MVC模式学习框架了。XML UI + Signal的View构建方式也挺优秀的。不过可惜的是现在Bee已经不再维护了，所以就拿来学习吧。
1.MVC
2.代理模式
3.单例
4.观察者
5.工厂模式

### 单元测试：
1.单元测试基础原理
2.XCTest


### 开发技术之外的还包括：
1.项目版本管理：SVN、Git
2.项目包依赖管理：CocoaPods
3.调试各种小技巧。比如断点（条件、全局）、lldb调试基本指令、NSZombieEnabled、
4.一些基本概念的理解，比如进程、线程、同步、异步、队列、串行、并发。



## 中级阶段。



在这个阶段呢，我们应该更多关注性能和业务方面的优化。

### 开发语言方面：
1.Swift：Objective-C与Swift互调。
2.JavaScript：使用Objective-C执行JavaScript。可以多熟悉了解JavaScriptCore。三方框架方面推荐WebViewjavaScriptBridge。
3.C、C++、Objective-C混编。

### iOS方面：
1.动画上熟悉CAAnimation（CABasicAnimation、CAKeyFrameAnimation、CAAnimationgroup\CATransition）、UIDynamics（UIDynamicAnimator、UIDynamicBehavior）
2.Runtime：objc_msgSend、Method Swizzling；
3.正则表达式：NSpredicate、NSRegularExpression。
4.消息推送机制
5.组件开发：创建Framework、打包静态库
6.分清32位和64位编译区别，能够将32位程序迁移到64位（这部分,,,,不强求）。

### 多媒体：
VLC组件使用频率较高，但其中部分不需要的解码库可以适当的剥除以降低库大小，SDWebImage可以细致的去研究他的加载策略缓存策略。CoreAudio、COreGraphics能够调度硬件进行编解码，提升效率多半是Android一时半会达不到的。

1.视频：MediaPlayer、VLCPlayer
2.图片：CoreGraphics、SDWebImage
3.音频：CoreAudio

### 网络交互：

1.NSStream
2.CFNetwork
3.NSURLconnection
4.NSURLSession
5.Json解析(model数据接收导致崩溃，多半在Json解析。)

### 架构设计：
1.设计模式熟悉MVVM等不同的分层模式
2.UML能够绘制和读懂常用模型图吧，可能科班出身的基本都问题不大。

### 应用测试：
还在为应用莫名其妙卡壳而苦恼吗，还在为找不到项目优化点而被产品同批吗？Instrument——你值得拥有。
1.性能测试：instrument（Timer、Allocation、Leak）



### 开发环境与工程框架

个人觉得也是中后期关注比较多的点


在开发环境与工程稳定方面，显然有这样的需求你也不是一只小菜鸟了。此时你对自己的项目有更高的追求了。
那么此时我们应该注意什么样的知识点呢？这部分整理，也是基于我司内部推行的一种Docker思想的整体企业级架构平台。平台集成了代码仓库、应用组建库、自动组件化应用打包。当时也去听了高焕堂老先生关于Docker思想的技术沙龙，感觉受益匪浅。真的开始融入到Docker思想后确实觉得有助于企业优化架构，并且我也完成了手头一个应用从传统架构到这种Docker架构的迁移，大概思路就是先将应用从业务上解耦，后进行组件化封装。通过Cocoapods的podspec进行本地组装，在传至平台组件仓库中打包供应用组装时使用。这个过程中用到了Git、Cocoapods、Jenkins的部分知识，因此多了一份工程框架方面的思考哈。


##### 工程框架


1.包依赖管理：Cocoapods、SwiftPackageManager、Carthage。如果你为了添加一个依赖库，还在手动从Git上面下载，那么就该注意去使用这些包管理工具了。提高效率很多，并且方便团队开发时，快速构建项目框架。
2.持续集成：Jenkins。首先，你先发现了自己在开发过程中对于持续集成的需求，并且发现真的很累...那么此时你就该认真的思考如何通过工具完成这一烦躁的工作了。
3.数据安全：
3.1数据加密：Hash（MD5、SHA1、SHA265）、RSA、AES、3DES、Base63. 
3.2HTTPS与SSL：做开发，必须要学会跳过HTTPS授权，iOS也不例外。
4.打包工具：Command Line Tools、xctool

##### 开发环境

1.Xcode插件：Alcatraz（不说别的，那个会炸开打字效果就值得你一试）、VVDocument、KSImageNamed
2.git：SourceTree 
3.SVN：Versions、Cornerstone。（讲真Versions比Cornerstone好用。）
4.开发者账号申请和管理：
4.1Apple Developer MemberCenter ：证书（发布证书、开发证书、推送证书）、设备管理、配置文件管理
4.2iTurnes Connect：Appstore应用管理、应用上架审核检测、加急审核。
5.热门技术：
5.1支付：微信支付、支付宝支付
5.2社会化分享：微博、微信朋友圈等（此处推荐使用三方库：UMengSocial）
5.3即时通讯：XMPP、VoIP（不知道放这里合不合适哈，做过视频会议，网络电话的人都懂。）
5.4HTML5：虽然我很忠实于原生应用开发，但是H5确实有点可取之处的（举个栗子吧，你做数据图表UI显示时，还是H5给你提供的库最多！），大家不妨关注下Hybrid App。
