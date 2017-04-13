---
layout: post
title:  "谈谈iOS多工程依赖"
date:   2015-09-26 00:00:00
categories: iOS
published: true
---

>分享自己处理多个工程之间依赖的方案。

# 工程形式
1. `*.xcodeproj`
2. `*.xcworkspace`

`xcworkspace`更好的管理着`Build Products Path`，统一编译结果路径，使用`workspace`的好处之一是能找到编译后的库文件，否则需要手动执行`Search Paths`。

* .a库在`Header Search Paths`和`Library Search Paths`设置里添加`$(BUILT_PRODUCTS_DIR)`即可加载库。
* .framework库需设置`Framework Search Paths`的路径以及在`Header Search Paths`设置`framework`内`.h`文件的路径（如:`"$(SRCROOT)/LibPath/yourLib.framework/Headers/"`），否则`Product`执行`archive`操作将会提示找不到找不到头文件。

而我则更喜欢直接使用`xcodeproj`，库引用路径可以直接通过`*.xcconfig`设置`Build Products Path`指向统一路径，或编写`shell`直接编译成库并移动到指定的目录。

# 常用依赖方式

多个工程依赖方式我了解3种，2种是有Git本身支持，还有一种是使用cocoaPods：

1. git submodule
2. git subtree
3. cocoapods

每个项目根据大小来合理的选择依赖方式；  

## Git Submodule

`git submodule`是以前项目经常使用的依赖方式，git通过`submodule`的形式，设置一些仓库为另一个仓库的子模块，增加依赖关系。  
子模块做为独立的仓库，可以正常的运行、编译。  
主仓库通过引入子模块代码，依赖编译。  

优点是各模块独立仓库，能独立编译，项目的公用部分，直接抽成新的仓库存放，创建新项目时，只需要添`git submodule add 仓库地址 路径`即可。  

缺点是主仓库不仅依赖子模块的代码，还依赖子模块的git提交，每次子模块更新后，需要在主仓库做一次提交，内容就是个`commitid`。  

如果是操作命令行，每次提交需要注意先更新子模块，提交子模块，然后才是主仓库。 
 
`SourceTree`是个不错的Git图形化工具，项目中有子模块，他会根据`.gitmodules`检测到子模块并提示先操作子模块仓库，比较方便而且不容易出错。  

主仓库更新所有的子模块: `git submodule foreach git pull`

如果你抽离出的共用仓库比较少，并且他们之前需要依赖关系，即公用仓库和主仓库**并行**开发且需要实时提交更新，可以试着使用`Git submodule`。  

### 更多git submodule资料
[Git 工具 - 子模块](https://git-scm.com/book/zh/v1/Git-工具-子模块)  
[Git Submodule使用完整教程](http://www.kafeitu.me/git/2012/03/27/git-submodule.html)  
[baidu Git Submodule](http://www.baidu.com/s?ie=UTF-8&wd=git%20submodule)

## Git Subtree

使用`Git Submodule`主仓库会将所有子模块的提交一并更新下来，虽然可以指定分支，但是主仓库的Git仓库还是被子模块污染了。  
`Git Subtree`解决了以上的问题，去掉了依赖，直接将子模块本身文件下载存放到主仓库中，命名`SubTree`，中文叫`子树`。  
没有子模块的概念，因为本身只有主仓库一个，当然也不是由主仓库的Git来管理原来子模块的仓库，因为不需要管理，为什么不需要管理呢，我们需要引入库的概念。  

### 库
iOS开发中，我们经常自己写并且使用在了项目中的就是静态库，常用格式支有`.a`和`.framework`。  
关于库的介绍大家可以参考:[库](http://casatwy.com/ku.html)。  

使用库开发（API开发），要遵守很多，比如已开放的方法名不能在一个版本中删除、更新需要兼容废弃的方法以及定义版本等，当然苹果SDK提供了非常不错的模板，可以参考`NSObjCRuntime.h`内的定义。  

#### framework后缀的静态库
0. 打开Xcode
1. File -> New -> Project -> Framework & Library
2. 选择Cocoa Touch Framework，输入库名称并创建
3. 在Build Settings，搜索 mach
4. 将Mach-O Type修改成Static Library

#### 库联合编译

1. 选中`Target`找到`Build Phases`
2. 选择`Target Dependencies`，添加需要依赖编译的工程
3. `Link Binary With Libraries`，添加依赖库

在同一项目中，库最好不要将依赖库添加到`Link Binary With Libraries`中，因为其他库也有依赖，多个库一起引用可能就会引起冲突。  

好一点的解决方式是

1. 库本身只引用H文件引用，编译不报错
2. 单元测试或测试工程添加依赖库，程序能正确运行

多个库出现冲突，请参考[工程链接静态库的时候，通过删除class来解决重复的符号的错误](http://blog.csdn.net/hherima/article/details/23949413)进行库分解和重新打包。  


### 使用
库的概念明白后，后面的就比较顺通了，我们添加的子树只是一个版本的库，我们添加引用他的功能。  
库更新并不会通知主仓库更新代码。  
主仓库发现子树有BUG或需要使用子树的新功能，则可以自己更新子树。  

使用子树的主要命令有:  

#### 添加   

	$git remote add -f jsonent https://github.com/maxfong/MFSJSONEntity.git 
	$git subtree add --prefix=路径 jsonent master --squash

#### 更新
	
	$git fetch jsonent master 
	$git subtree pull --prefix=test jsonent master --squash

这里没有提交是因为库作为一个独立的仓库，可以独立开发，不需要在其他仓库上修改。  
当然如果是fix BUG的另说，更多`subtree`可以参考:  

[使用GIT SUBTREE集成项目到子目录](http://aoxuis.me/post/2013-08-06-git-subtree)  
[Baidu Git SubTree](https://www.baidu.com/s?ie=UTF-8&wd=git%20subtree)

####子树更新冲突  
根据提示解决冲突，实在无法解决，直接删掉对应的库目录，重新`Clone`，这里需要关注下**--squash**参数的作用。  

## CocoaPods

大部分项目使用`subtree`就能解决依赖问题，但是项目越多越麻烦，比如编译脚本的维护等。   

pod的优秀就不说了，它不仅包含`subtree`所有功能，不产生依赖，并且集成、更新都比较简单，可以自定义下载需要的文件路径，但是大部分项目并没有使用它做工程依赖。  
我觉得`CocoaPods`是每个优秀的iOS程序员必须掌握的技能，所以使用pod的成本应该是比较小的。  

使用`CocoaPods`需要自己创建私有库，网上很多代码存放平台，如果不满足可以自己搭建如`GitLab`这样的开源的代码管理平台。  

#### 参考文档 

[CocoaPods建立私有仓库](http://blog.csdn.net/agdsdl/article/details/45218987)  
[使用Cocoapods创建私有podspec](http://blog.wtlucky.com/blog/2015/02/26/create-private-podspec/)

#### 大一点的项目

一个很大的项目内可能包含多个子项目。  
子项目使用pod添加依赖库，能正常编译运行。  

发布阶段，将多个子项目作为pod库，供主项目依赖，每次子项目完成一次迭代，打一个tag，然后让主项目更新即可。    
对于一个项目有多个模块，开发人员也比较多，推荐这种方式。  

# 结论

文章的内容就是介绍了3种依赖方式。  
我更偏向的依赖方式就是使用cocoaPods建立各种私有库。  
以前我也不喜欢pod，感觉麻烦，直到考虑项目依赖的时候，才发现pod才是绝佳的工具。  
推荐大家都使用起来。  
