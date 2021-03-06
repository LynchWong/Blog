---
title: "iOS视频播放"
date: 2015-10-14 18:18:18
categories: 
- iOS
tags: 
- 音频
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

离上一篇博客发布又是两周时间过去了，心里总是感觉空空的。两周没有发布一篇博客，感觉就像是虚度了。另外一个原因就是实在是不知道写点什么，有些内容兴趣不大就搁置了。其次就是之前的天坑真是没有动力去填了，比如CoreData，就放在那吧。上篇博客也提到了这篇做视频播放器，所以这篇就是讲讲视频播放器的。
<!--more-->

上篇博客也提到了一些资源：

[iOS开发系列--音频播放、录音、视频播放、拍照、视频录制](http://www.cnblogs.com/kenshincui/p/4186022.html)
[iOS音频播放 (一)：概述](http://msching.github.io/blog/2014/07/07/audio-in-ios/)
[AVFoundation Programming Guide](https://developer.apple.com/library/ios/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/00_Introduction.html)

第一个资源基本所有的东西都讲到了，音频、视频等。简单易懂，入门级学习都够了，也能满足基本的需求。
第二个资源主要讲解音频的，很专业。
第三个资源是苹果官方的编程指南，具有很好的指导意义，建议大家阅读。

其实基本知识点前面的这些资源都讲到了，我这里再讲也就没什么意思了。

所以我们这里就做一个完整功能的播放器，播放，暂停，快进，全屏等功能。

接下来我们就开始吧，先说下我的开发环境。

* OS X EI Capitan(Version 10.11)
* Xcode(Version 7.0.1)
* 使用Swift作为开发语言

# 建立项目 #
如下图所示，我们选择*Single View Application*，项目名称叫做*LWPlayer*，语言选择*Swift*。
![alt text](/img/iOSLWPlayer/1.png)

![alt text](/img/iOSLWPlayer/2.png)

然后我们将项目的*Deployment Target*修改为8.0。然后我们删除*Main.storyboard*文件，同时*Main Interface*里面的Main也去掉。因为我们删除了storyboard文件，如果不删除Main的话程序无法启动直接崩溃。

现在项目的界面应该如下所示：

![alt text](/img/iOSLWPlayer/3.png)

这里我们删除了storyboard文件，我的打算是用代码手写界面，不使用storyboard和xib文件。大家可能都知道用代码写布局很繁琐，所以这里使用第三方的框架*SnapKit*。这个第三方框架其实就是OC版本的*Masonry*，这里就不细讲布局的事情了，参考资料[ SnapKit ](https://github.com/SnapKit/Masonry)和[ Masonry ](https://github.com/SnapKit/Masonry)。

现在项目编译运行之后就是一片漆黑，之后我们要自己做一些初始化的工作。首先我们先添加第三方库，及*SnapKit*。我们使用依赖管理器来添加第三方库。可以使用*CocoaPods*，当然你也可以使用*Carthage*，如果是Swift以及iOS8.0以上推荐使用*Carthage*来管理。这里我们使用*Carthage*，[ Carthage ](https://github.com/Carthage/Carthage#adding-frameworks-to-an-application)，安装很简单，请自行了解。

*Carthage*的参考资料：

[Carthage](https://github.com/Carthage/Carthage#adding-frameworks-to-an-application)
[Carthage：去中心化的Cocoa依赖管理器](http://www.cocoachina.com/ios/20141204/10528.html)
[Cocoa 新的依赖管理工具：Carthage](http://www.isaced.com/post-265.html)

第一篇是官方的，第二篇相当于是官方的翻译，大家自行参考学习。

我们进入到项目目录执行如下命令，截图如下：

![alt text](/img/iOSLWPlayer/4.png)

进入到项目目录后我们创建了*Cartfile*文件，然后在里面添加需要的第三方库。如下所示：

![alt text](/img/iOSLWPlayer/5.png)

然后*:wq*保存退出，然后执行*carthage update*命令进行安装编译。

这里就算是安装编译完成了，我们可以进到工程目录里面看下编程生成的二进制framework文件。如下图所示：

![alt text](/img/iOSLWPlayer/6.png)

不仅有iOS的，还有Mac的。现在我们还差最后一步，就是将framework添加到我们的工程。设置*Framework Search Paths*是最简单的方法，告诉工程到那里去找这些framework。设置为*$(SRCROOT)/Carthage/Build/iOS*，如果是OS X就把iOS改成Mac。如下所示：

![alt text](/img/iOSLWPlayer/7.png)

这一步和官方说明的不一样，按照官方的方法，每次添加了新的framework都要自己到Xcode里面进行手动添加。

第三方库算是添加完成了，现在我们要修改*AppDelegate*文件。修改*func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool*方法如下：

    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        // Override point for customization after application launch.
        
        //appearance
        UINavigationBar.appearance().barTintColor = UIColor(red: 56.0/255.0, green: 62.0/255.0, blue: 72.0/255.0, alpha: 1.0)
        UINavigationBar.appearance().tintColor = UIColor.whiteColor()
        
        //rootViewController
        let rootVC = ViewController()
        
        window = UIWindow(frame: UIScreen.mainScreen().bounds)
        window!.rootViewController = rootVC
        window?.makeKeyAndVisible()
        
        return true
    }

这里代码很简单，就不讲解了。运行启动后屏幕仍然是黑的，我们继续做些修改。将*ViewController*的代码修改为如下所示：

    import UIKit
    import SnapKit

    class ViewController: UIViewController {

        override func viewDidLoad() {
            super.viewDidLoad()
            // Do any additional setup after loading the view, typically from a nib.
            view.backgroundColor = UIColor.whiteColor()
            
            let aView = UIView()
            aView.backgroundColor = UIColor.orangeColor()
            view.addSubview(aView)
            
            aView.snp_makeConstraints { (make) -> Void in
                make.edges.equalTo(view).offset(UIEdgeInsets(top: 10, left: 10, bottom: -10, right: -10))
            }
        }

        override func didReceiveMemoryWarning() {
            super.didReceiveMemoryWarning()
            // Dispose of any resources that can be recreated.
        }


    }

我们首先导入*SnapKit*模块，然后修改视图的背景色为白色，然后添加了一个距离父视图编剧为10的橙色子视图。布局的代码，*SnapKit*的使用方法就不讲解了。

编译运行后会发现如下错误，

![alt text](/img/iOSLWPlayer/8.png)

这里是因为之前修改*Framework Search Paths*导致的，其实前面的这一步貌似并没有效果。所以我们还是得按照官方的来，如下所示：

![alt text](/img/iOSLWPlayer/9.png)

我们在*Embedded Binaries*这里将*Carthage/Build/iOS/SnapKit.framework*添加进来，然后编译运行，截图如下：

![alt text](/img/iOSLWPlayer/10.png)

# 项目结构 #

首先我们删除*ViewController*，新建*LWPlayerViewController*，代码如下：

    import UIKit
    import SnapKit

    class LWPlayerViewController: UIViewController {

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.
            let aView = UIView();
            aView.backgroundColor = UIColor.orangeColor();
            view.addSubview(aView)
            
            aView.snp_makeConstraints { (make) -> Void in
                make.edges.equalTo(view)
            }
        }

        override func didReceiveMemoryWarning() {
            super.didReceiveMemoryWarning()
            // Dispose of any resources that can be recreated.
        }

    }

很简单就不讲解了，同时我们还需要修改*AppDelegate*的*func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool*方法，代码如下：

    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        // Override point for customization after application launch.
        
        //appearance
        UINavigationBar.appearance().barTintColor = UIColor(red: 56.0/255.0, green: 62.0/255.0, blue: 72.0/255.0, alpha: 1.0)
        UINavigationBar.appearance().tintColor = UIColor.whiteColor()
        
        //rootViewController
        let rootVC = LWPlayerViewController()
        
        window = UIWindow(frame: UIScreen.mainScreen().bounds)
        window!.rootViewController = rootVC
        window?.makeKeyAndVisible()
        
        return true
    }

运行后截图如下：

![alt text](/img/iOSLWPlayer/11.png)

你可能发现我们设置了导航栏的外观，所以我们接下来就新建*LWNavigationController*继承自*UINavigationController*，代码如下：

    import UIKit

    class LWNavigationController: UINavigationController {
        
        override init(rootViewController: UIViewController) {
            super.init(rootViewController: rootViewController)
        }
        
        override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: NSBundle?) {
            super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
        }

        required init?(coder aDecoder: NSCoder) {
            fatalError("init(coder:) has not been implemented")
        }

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.
        }

        override func didReceiveMemoryWarning() {
            super.didReceiveMemoryWarning()
            // Dispose of any resources that can be recreated.
        }
        
        // MARK: - StatusBar

        override func preferredStatusBarStyle() -> UIStatusBarStyle {
            return UIStatusBarStyle.LightContent
        }

    }

这里的代码也没什么好讲解的，主要就是一些初始化方法，有些是必需的，没有会报错。之后我们修改*AppDelegate*，首先添加：

	var navigationVC: LWNavigationController!

的属性，然后修改*func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool*方法：

    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        // Override point for customization after application launch.
        
        //appearance
        UINavigationBar.appearance().barTintColor = UIColor(red: 56.0/255.0, green: 62.0/255.0, blue: 72.0/255.0, alpha: 1.0)
        UINavigationBar.appearance().tintColor = UIColor.whiteColor()
        
        //rootViewController
        let rootVC = LWPlayerViewController()
        navigationVC = LWNavigationController(rootViewController: rootVC)
        
        window = UIWindow(frame: UIScreen.mainScreen().bounds)
        window!.rootViewController = navigationVC
        window?.makeKeyAndVisible()
        
        return true
    }

运行后模拟器截图：

![alt text](/img/iOSLWPlayer/12.png)

# 视频资源

说来惭愧，没有在网上找到可以用来进行远程播放的视频资源，所以我们要自己弄个。使用**CocoaHTTPServer**第三方库，让App内嵌一个HTTP server。

[VKVideoPlayer](https://github.com/viki-org/VKVideoPlayer)

这里有一个开源的播放器，功能很完善，包括加载字幕、广告等等，大家可以参考学习下。

这里我们先来添加第三方库**CocoaHTTPServer**，前面我们使用的是Carthage来管理依赖的，貌似 这个第三方库不行。但是CocoaPods可以，但是我们这里就不使用CocoaPods，估计这里是可以用的，这两个依赖管理应该不会冲突，这里我们就不测试了，直接使用最原始的方法：拖放文件。

[CocoaHTTPServer](https://github.com/robbiehanson/CocoaHTTPServer) ，这里是下载地址，下载完成后我们将**Core**，**Extensions**，**Vendor**放到新建的文件夹**CocoaHTTPServer**中，其他的东西我们不需要了。然后将这个文件夹拖放到工程中，如下图所示：

![alt text](/img/iOSLWPlayer/13.png)

接下来我们添加**libxml2.tbd**，不添加会报找不到文件的错。添加后如下图所示。

![alt text](/img/iOSLWPlayer/14.png)

因为这个第三方库是OC，如果要混编的话我们需要添加桥接头文件，**LWPlayer-Bridging-Header.h**。修改文件内容的代码如下：

	#import "HTTPServer.h"

然后我们还要配置下头文件，如下所示：

![alt text](/img/iOSLWPlayer/15.png)

然后我们参考[ VKVideoPlayer ](https://github.com/viki-org/VKVideoPlayer)的**HTTPStreamingServer**，代码如下：

    import UIKit

    let HTTPStreamingServerSingleton = HTTPStreamingServer()

    class HTTPStreamingServer: NSObject {
        
        var httpServer: HTTPServer!
        
        class var sharedInstance: HTTPStreamingServer {
            get {
                return HTTPStreamingServerSingleton
            }
        }
        
        override init() {
            super.init()
            initialize()
        }
        
        func initialize() {
            httpServer = HTTPServer()
            httpServer.setType("_http._tcp.")
            httpServer.setPort(12345)
            let webPath = NSBundle.mainBundle().resourcePath?.stringByAppendingString("/web")
            httpServer.setDocumentRoot(webPath)
        }
        
        func start() {
            do {
                try httpServer.start()
                print("Started HTTP Server on port \(httpServer.listeningPort())")
            } catch {
                print("Error starting HTTP Server")
            }
        }
        
        func stop() {
            httpServer.stop()
        }

    }

定义了一个全局的常量，然后使用类计算型属性实现单列。其他的代码很简单，就不讲解了。

然后我们在**func applicationDidBecomeActive(application: UIApplication)**方法中加入**HTTPStreamingServer.sharedInstance.start()**及**func applicationWillTerminate(application: UIApplication)**方法中添加**HTTPStreamingServer.sharedInstance.stop()**，运行之后在控制台输出**Started HTTP Server on port 12345**。

然后我们将一些视频资源拖到项目中，这里我们就用**VKVideoPlayer**的资源，将**web**这个文件夹拖到我们项目中，注意我截图中的箭头，如下所示：

![alt text](/img/iOSLWPlayer/16.png)

# 播放器 #

新建一个类**LWPlayerView**继承自**UIView**，代码如下：

    import UIKit
    import AVFoundation

    class LWPlayerView: UIView {
        
        var player: AVPlayer {
            set {
                (layer as! AVPlayerLayer).player = newValue
            }
            get {
                return (layer as! AVPlayerLayer).player!
            }
        }

        override class func layerClass() -> AnyClass {
            return AVPlayerLayer.classForCoder()
        }

    }

如果你看过苹果的编程指南或者苹果给的Demo，都是这样实现的，可以算是最佳实践吧。

接下来我们就来实现播放器，到**LWPlayerViewController**。首先我们添加如下所示的属性：

    let videoPath = "http://localhost:12345/wifi_640_index.m3u8"//视频播放地址
    let radio = CGFloat(375.0 / 238.0)//缩放比例
    let width = UIScreen.mainScreen().bounds.size.width//屏幕宽度
    
    let playContentView = UIView()//播放器容器视图
    let toolContentView = UIView()//播放器工具容器视图
    
    var player: AVPlayer!//播放器
    var playerView = LWPlayerView()//播放视图
    
    var hideTool = false

后面都有注释，最后一个属性用来表示是否隐藏状态栏和工具栏。然后我们添加初始化播放器的方法：

    func initPlayer() {
        let url = NSURL(string: videoPath)!
        let playerItem = AVPlayerItem(URL: url)
        player = AVPlayer(playerItem: playerItem)
    }

然后是初始化视图的方法：

    func initView() {
        
        playContentView.backgroundColor = UIColor.blackColor()
        view.addSubview(playContentView)
        playContentView.snp_makeConstraints { (make) -> Void in
            make.top.left.right.equalTo(view)
            make.width.equalTo(playContentView.snp_height).multipliedBy(radio)
        }
        
        playerView.player = player
        playerView.addGestureRecognizer(UITapGestureRecognizer(target: self, action: Selector("sigleTapAction")))
        playContentView.addSubview(playerView)
        playerView.snp_makeConstraints { (make) -> Void in
            make.edges.equalTo(playContentView)
        }
        
        toolContentView.backgroundColor = UIColor(red: 56.0/255.0, green: 62.0/255.0, blue: 72.0/255.0, alpha: 1.0)
        playContentView.addSubview(toolContentView)
        toolContentView.snp_makeConstraints { (make) -> Void in
            make.left.bottom.right.equalTo(playContentView)
            make.height.equalTo(30.0)
        }
    }

然后我们实现**sigleTapAction()**方法：

    func sigleTapAction() {
        if hideTool {
            hideTool = false
            navigationController?.navigationBar.hidden = false
            toolContentView.hidden = false
        } else {
            hideTool = true
            navigationController?.navigationBar.hidden = true
            toolContentView.hidden = true
        }
    }



然后修改**func viewDidLoad()**方法，当点击的时候会隐藏状态栏和工具栏：

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = UIColor.whiteColor()
        initPlayer()
        initView()
        player.play()
    }

运行后截图如下：

![alt text](/img/iOSLWPlayer/17.png)

大家都知道HTTP请求在iOS9里面有变化，所以有的同学可能没有看到视频播放的画面；同样的**VKVideoPlayer**你可能也看不到视频播放，都是相同的原因，大家搜索都能解决。设置下**.plist**文件就可以了，如下图所示：

![alt text](/img/iOSLWPlayer/18.png)

注意这两个字段的类型，一个是字典类型，一个是布尔类型。

## 播放器界面 ##

接下来我们就来完成播放器的界面，让它开起来更像一个播放器。首先我们添加如下所示的属性：

    let playButton = UIButton()//播放按钮
    let muteButton = UIButton()//静音按钮
    let progress = UISlider()//播放进度
    let srtsButton = UIButton()//字幕按钮
    let fullButton = UIButton()//全屏按钮

接下来我们就来布局这些按钮，修改**initView()**方法，代码如下，直接替换即可：

    func initView() {
        
        playContentView.backgroundColor = UIColor.blackColor()
        view.addSubview(playContentView)
        playContentView.snp_makeConstraints { (make) -> Void in
            make.top.left.right.equalTo(view)
            make.width.equalTo(playContentView.snp_height).multipliedBy(radio)
        }
        
        playerView.player = player
        playerView.addGestureRecognizer(UITapGestureRecognizer(target: self, action: Selector("sigleTapAction")))
        playContentView.addSubview(playerView)
        playerView.snp_makeConstraints { (make) -> Void in
            make.edges.equalTo(playContentView)
        }
        
        toolContentView.backgroundColor = UIColor(red: 56.0/255.0, green: 62.0/255.0, blue: 72.0/255.0, alpha: 1.0)
        playContentView.addSubview(toolContentView)
        toolContentView.snp_makeConstraints { (make) -> Void in
            make.left.bottom.right.equalTo(playContentView)
            make.height.equalTo(30.0)
        }
        
        playButton.setImage(UIImage(named: "pause_icon_vertical"), forState: UIControlState.Normal)
        playButton.setImage(UIImage(named: "play_icon_vertical"), forState: UIControlState.Selected)
        muteButton.setImage(UIImage(named: "sound_on_vertical"), forState: UIControlState.Normal)
        muteButton.setImage(UIImage(named: "sound_off_vertical"), forState: UIControlState.Selected)
        srtsButton.setImage(UIImage(named: "chinese_english_shift_vertical"), forState: UIControlState.Normal)
        srtsButton.setImage(UIImage(named: "english_shift_vertical"), forState: UIControlState.Selected)
        fullButton.setImage(UIImage(named: "full_screen_icon_vertical"), forState: UIControlState.Normal)
        
        progress.minimumValue = 0.0
        progress.maximumValue = 1.0
        progress.continuous = false
        progress.setThumbImage(UIImage(named: "VKScrubber_thumb_vertical"), forState: UIControlState.Normal)
        progress.value = 0.0
        progress.minimumTrackTintColor = UIColor(red: 10 / 255, green: 137 / 255, blue: 153 / 255, alpha: 1.0)
        progress.maximumTrackTintColor = UIColor(red: 10 / 255, green: 137 / 255, blue: 153 / 255, alpha: 0.4)
        
        playButton.addTarget(self, action: Selector("playAction:"), forControlEvents: UIControlEvents.TouchUpInside)
        muteButton.addTarget(self, action: Selector("muteAction:"), forControlEvents: UIControlEvents.TouchUpInside)
        srtsButton.addTarget(self, action: Selector("srtsAction:"), forControlEvents: UIControlEvents.TouchUpInside)
        fullButton.addTarget(self, action: Selector("fullAction:"), forControlEvents: UIControlEvents.TouchUpInside)
      
        toolContentView.addSubview(playButton)
        toolContentView.addSubview(muteButton)
        toolContentView.addSubview(progress)
        toolContentView.addSubview(srtsButton)
        toolContentView.addSubview(fullButton)
        
        playButton.snp_makeConstraints { (make) -> Void in
            make.left.equalTo(10.0)
            make.centerY.equalTo(toolContentView.snp_centerY)
            make.height.width.equalTo(20.0)
        }
        
        muteButton.snp_makeConstraints { (make) -> Void in
            make.left.equalTo(playButton.snp_right).offset(13.0)
            make.centerY.equalTo(toolContentView.snp_centerY)
            make.height.width.equalTo(20.0)
        }
        
        progress.snp_makeConstraints { (make) -> Void in
            make.left.equalTo(muteButton.snp_right).offset(22.0)
            make.right.equalTo(srtsButton.snp_left).offset(-16.0)
            make.centerY.equalTo(toolContentView.snp_centerY)
            make.height.equalTo(2.0)
        }
        
        srtsButton.snp_makeConstraints { (make) -> Void in
            make.right.equalTo(fullButton.snp_left).offset(-14.0)
            make.centerY.equalTo(toolContentView.snp_centerY)
            make.height.width.equalTo(20.0)
        }
        
        fullButton.snp_makeConstraints { (make) -> Void in
            make.right.equalTo(-10.0)
            make.centerY.equalTo(toolContentView.snp_centerY)
            make.height.width.equalTo(20.0)
        }
    }

代码很长，但是基本没有难点，大都是布局的代码，这里就不讲解了。然后我们添加按钮相应的Action：

    func playAction(sender: UIButton) {
        if player.rate == 0 {
            player.play()
            sender.selected = true
        } else {
            player.pause()
            sender.selected = false
        }
    }
    
    func muteAction(sender: UIButton) {
        if player.muted {
            player.muted = false
            sender.selected = false
        } else {
            player.muted = true
            sender.selected = true
        }
    }
    
    func srtsAction(sender: UIButton) {
        
    }
    
    func fullAction(sender: UIButton) {
        
    }

然后我们将**viewDidLoad()**方法里面的播放的代码**player.play()**去掉，运行之后截图如下：

![alt text](/img/iOSLWPlayer/19.png)

## 强制横屏 ##

首先我们修改横屏按钮要调用的方法，代码如下：

    func fullAction(sender: UIButton) {
        UIDevice.currentDevice().setValue(NSNumber(integer: UIInterfaceOrientation.LandscapeRight.rawValue), forKey: "orientation")
        UIViewController.attemptRotationToDeviceOrientation()
    }

接下来就是要调整下横屏时的视图，代码如下：

    override func willRotateToInterfaceOrientation(toInterfaceOrientation: UIInterfaceOrientation, duration: NSTimeInterval) {
        if UIInterfaceOrientationIsLandscape(toInterfaceOrientation) {
            playContentView.snp_remakeConstraints { (make) -> Void in
                make.edges.equalTo(view)
            }
            
            toolContentView.snp_remakeConstraints { (make) -> Void in
                make.left.bottom.right.equalTo(playContentView)
                make.height.equalTo(52.0)
            }
            
            playButton.setImage(UIImage(named: "pause_icon_landscape"), forState: UIControlState.Normal)
            playButton.setImage(UIImage(named: "play_icon_landscape"), forState: UIControlState.Selected)
            muteButton.setImage(UIImage(named: "sound_on_landscape"), forState: UIControlState.Normal)
            muteButton.setImage(UIImage(named: "sound_off_landscape"), forState: UIControlState.Selected)
            srtsButton.setImage(UIImage(named: "chinese_english_shift_landscape"), forState: UIControlState.Normal)
            srtsButton.setImage(UIImage(named: "english_shift_landscape"), forState: UIControlState.Selected)
            fullButton.setImage(UIImage(named: "full_screen_icon_landscape"), forState: UIControlState.Normal)
            progress.setThumbImage(UIImage(named: "VKScrubber_thumb_landscape"), forState: UIControlState.Normal)
            
            playButton.snp_remakeConstraints { (make) -> Void in
                make.left.equalTo(25.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.width.equalTo(24.0)
            }
            
            muteButton.snp_remakeConstraints { (make) -> Void in
                make.left.equalTo(playButton.snp_right).offset(32.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.width.equalTo(24.0)
            }
            
            progress.snp_remakeConstraints { (make) -> Void in
                make.left.equalTo(muteButton.snp_right).offset(30.0)
                make.right.equalTo(srtsButton.snp_left).offset(-31.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.equalTo(4.0)
            }
            
            srtsButton.snp_remakeConstraints { (make) -> Void in
                make.right.equalTo(fullButton.snp_left).offset(-26.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.width.equalTo(24.0)
            }
            
            fullButton.snp_remakeConstraints { (make) -> Void in
                make.right.equalTo(-27.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.width.equalTo(24.0)
            }
        } else {
            playContentView.snp_remakeConstraints { (make) -> Void in
                make.top.left.right.equalTo(view)
                make.width.equalTo(playContentView.snp_height).multipliedBy(radio)
            }
            
            toolContentView.snp_remakeConstraints { (make) -> Void in
                make.left.bottom.right.equalTo(playContentView)
                make.height.equalTo(30.0)
            }
            UIApplication.sharedApplication().statusBarHidden = true
            playButton.setImage(UIImage(named: "pause_icon_vertical"), forState: UIControlState.Normal)
            playButton.setImage(UIImage(named: "play_icon_vertical"), forState: UIControlState.Selected)
            muteButton.setImage(UIImage(named: "sound_on_vertical"), forState: UIControlState.Normal)
            muteButton.setImage(UIImage(named: "sound_off_vertical"), forState: UIControlState.Selected)
            srtsButton.setImage(UIImage(named: "chinese_english_shift_vertical"), forState: UIControlState.Normal)
            srtsButton.setImage(UIImage(named: "english_shift_vertical"), forState: UIControlState.Selected)
            fullButton.setImage(UIImage(named: "full_screen_icon_vertical"), forState: UIControlState.Normal)
            progress.setThumbImage(UIImage(named: "VKScrubber_thumb_vertical"), forState: UIControlState.Normal)
            
            playButton.snp_remakeConstraints { (make) -> Void in
                make.left.equalTo(10.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.width.equalTo(20.0)
            }
            
            muteButton.snp_remakeConstraints { (make) -> Void in
                make.left.equalTo(playButton.snp_right).offset(13.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.width.equalTo(20.0)
            }
            
            progress.snp_remakeConstraints { (make) -> Void in
                make.left.equalTo(muteButton.snp_right).offset(22.0)
                make.right.equalTo(srtsButton.snp_left).offset(-16.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.equalTo(2.0)
            }
            
            srtsButton.snp_remakeConstraints { (make) -> Void in
                make.right.equalTo(fullButton.snp_left).offset(-14.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.width.equalTo(20.0)
            }
            
            fullButton.snp_remakeConstraints { (make) -> Void in
                make.right.equalTo(-10.0)
                make.centerY.equalTo(toolContentView.snp_centerY)
                make.height.width.equalTo(20.0)
            }
        }
    }

运行之后截图如下：

![alt text](/img/iOSLWPlayer/20.png)

现在只能横屏，还不能返回到竖屏的状态，接下来修改**fullAction(sender: UIButton)**方法，代码如下：

    func fullAction(sender: UIButton) {
        if UIDevice.currentDevice().orientation == UIDeviceOrientation.Portrait {
            UIDevice.currentDevice().setValue(NSNumber(integer: UIInterfaceOrientation.LandscapeRight.rawValue), forKey: "orientation")
            UIViewController.attemptRotationToDeviceOrientation()
        } else {
            UIDevice.currentDevice().setValue(NSNumber(integer: UIInterfaceOrientation.Portrait.rawValue), forKey: "orientation")
            UIViewController.attemptRotationToDeviceOrientation()
        }
    }

## 播放进度 ##

其实实现播放器进度很简单，如果看了最开始的那些资源的话。几行代码就可以解决了：

    func addProgressObserver() {
        let playerItem = player.currentItem!
        player.addPeriodicTimeObserverForInterval(CMTimeMake(1, 1), queue: dispatch_get_main_queue()) { (time: CMTime) -> Void in
            let current = CMTimeGetSeconds(time)
            let total = CMTimeGetSeconds(playerItem.duration)
            let progress = current / total
            self.progress.setValue(Float(progress), animated: true)
        }
    }

获取当前播放器的播放资源，然后得出总的时间和当前的时间，然后做一个百分比，最后设置**UISlider**的值。这个方法在我之前做进度条的时候就用到了，那里唯一不同的地方就是每秒调用10次，这里是每秒一次，设置**CMTimeMake(1, 1)**为**CMTimeMake(1, 10)**就是每秒调用10次。

然后我们在初始化播放器的方法里面添加调用该方法的代码：

    func initPlayer() {
        let url = NSURL(string: videoPath)!
        let playerItem = AVPlayerItem(URL: url)
        player = AVPlayer(playerItem: playerItem)
        addProgressObserver()
    }

运行之后如下所示:

![alt text](/img/iOSLWPlayer/21.png)

## 拖动进度 ##

首先我们添加一个属性，如下所示：

	var timeObserver: AnyObject!

然后修改**addProgressObserver()**方法，如下：

    func addProgressObserver() {
        let playerItem = player.currentItem!
        timeObserver = player.addPeriodicTimeObserverForInterval(CMTimeMake(1, 1), queue: dispatch_get_main_queue()) { (time: CMTime) -> Void in
            let current = CMTimeGetSeconds(time)
            let total = CMTimeGetSeconds(playerItem.duration)
            let progress = current / total
            self.progress.setValue(Float(progress), animated: true)
        }
    }

然后添加一个新的方法：

    func removeProgressObserve() {
        player.removeTimeObserver(timeObserver)
    }

苹果说这两个方法最好成对出现，你Cmd点击这些方法名就能看见苹果给的注释和描述了。

现在我们就来实现主要的逻辑了，但是首先还是要给我们的**UISlider**添加Action，代码如下：

	progress.addTarget(self, action: Selector("updateProgress:"), forControlEvents: UIControlEvents.ValueChanged)

然后我们实现**updateProgress(sender: UISlider)**方法，这里就是主要逻辑了，代码如下：

    func updateProgress(sender: UISlider) {
        removeProgressObserve()
        let progress = Float64(sender.value)
        let seekTime = CMTimeGetSeconds(player.currentItem!.duration) * progress
        player.seekToTime(CMTimeMakeWithSeconds(seekTime, 1), toleranceBefore: kCMTimeZero, toleranceAfter: kCMTimeZero) { (finished: Bool) -> Void in
            if finished {
                self.addProgressObserver()
            }
        }
    }

首先我们移除掉进度监视的方法，避免在拖动的时候还在设置进度的值。然后我们算出拖动的值对应的要播放的时间，然后我们就调用**AVPlayer**的方法就可以了，然后在结束的时候再添加进度监视就OK了。经测试，不调用这两个方法也是OK的。

运行后截图如下：

![alt text](/img/iOSLWPlayer/22.png)

![alt text](/img/iOSLWPlayer/23.png)

第一张图启动后没有点击播放，然后我们拖动进度条，拖动后的效果就是第二张图。这个进度条不太好拖动，如果你是模拟器，用鼠标点击圆点的左上角比较好拖动。

播放器到此就基本完成了，当然功能不够完善，比如音量不能调节，只能静音开关。还剩下一个字幕的功能也没有做，这里就不再继续了。字幕的实现主要难点就是解析字幕文件了，留给大家了，不会就参考[ VKVideoPlayer ](https://github.com/viki-org/VKVideoPlayer)。

[ 完整源码 ](https://github.com/LynchWong/LWPlayer)，这里我把图片资源去掉了，不方便贡献给大家，见谅。
