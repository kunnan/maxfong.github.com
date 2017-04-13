---
layout: post
title:  "MAC RTX 查看大图头像"
date:   2015-09-12 00:00:00
categories: iOS
published: true
---

>RTX是一款实时通信软件，很多企业使用，因为支持二次开发([RTX技术服务中心](http://rtx.tencent.com/rtx/support/index.shtml))，所以很多优秀的开发者开发了很多有意思的插件，还有团队为Window版RTX定制插件，当然这些在`MAC版`都无法使用；  

# 前提
Window版有一款非常好用的查看大图功能的插件；  
得益于公司的规定，RTX上必须放真人头像，每当任务栏上RTX图标闪动，打开是个妹子咨询你问题的时候，都会情不自禁的右击图像查看大图，当然了，这只是为了明确人物然后确定目的，加快办事效率嘛，\\(▔▽▔)/ ；  
然而到了MAC版，一切都变了，没有插件可用了，我们急需一款可用的看图软件；  

# 分析
在MAC RTX的偏好设置中，根据`将收到的文件存储到`选项，定位到了RTX头像存储位置：  
`/Users/[username]/Library/Application Support/RTXC/accounts/[rtxname]/userPhotos`  
目录内的文件正是RTX账号登陆名称和`.bmp`结尾的图片；  
写一款搜索软件，根据名称能搜到则显示图片；  

# 开发
MAC开发本质同iOS开发差不多，都是用Xcode，但是我觉得MAC开发需要掌握的知识更多，毕竟MAC是一个平台，而iOS更多的是展示，以下内容是开发一款小巧App应该掌握的：  

### NSApplicationDelegate
同`UIApplicationDelegate`类似，我们需要掌握`NSApplicationDelegate`里的方法，他控制了应用生命周期中里各阶段的delegate；  
为了图方便，我直接设置`- (BOOL)applicationShouldTerminateAfterLastWindowClosed:(NSApplication *)sender`返回YES，意为点击关闭直接退出应用程序；

### MainMenu.xib
界面的设计，同iOS开发一样，这里需要注意同iOS不同的是Menu，虽然Menu是默认创建的，除非要自定义，一般情况不需要**删除**，特别是`File`和`Edit`，他们控制着当前MAC App的好几个常用快捷键(你可以转移摆放位置)，每一个`NSMenuItem`的`Key Equivalent`是设置快捷键的；  
MAC控件比iOS多，而且很多感觉很熟悉，但是属性和用法可能并不相同；  
就像MAC的`Labe`是`NSTextField`类型，赋值也不是`text`，而是`stringValue`；  
界面设置完成后，通过`IBAction`连接相应事件即可；  

### IBAction
同iOS开发的`IBAction`和`IBOutlet`，因为功能比较小，直接通过`MainMenu.xib`设置界面连线到AppDelegate中了；  
方法实现很简单了，因为RTX是多用户登录，所以`[rtxname]`需要遍历，在目录内通过拼接`userPhotos`和`name.bmp`生成`NSImage`对象并显示；  

### 其他
虽然这个看头像软件没用着，但MAC开发也常用的有：  

#### 恢复显示关闭的窗口
`- (BOOL)applicationShouldHandleReopen:(NSApplication *)theApplication hasVisibleWindows:(BOOL)flag`

#### Dock右击菜单
`- (NSMenu *)applicationDockMenu:(NSApplication *)sender`

#### 开机启动
`LSSharedFileListInsertItemURL`

#### NSTableView
[基于cell-base的NSTableView](http://blog.csdn.net/fengsh998/article/details/18809355)

# 后续
作为程序员，可以写出很多有意思的东西，保持好奇，保持学习；  
Github:[ShowRTXFaceImage](https://github.com/maxfong/ShowRTXFaceImage)  
还有MAC RTX更新进度太慢了，布局适配在搜索的时候就会出现问题，账号不能修改信息，希望能改进的更好用；  