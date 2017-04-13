---
layout: post
title:  "初识On-Demand Resources"
date:   2015-09-17 00:00:00
categories: iOS
published: true
---

>今天刚用上Xcode 7，发现target tab出现Resources Tags(苹果文档部分配图是Asset Tags)，立马查询一番。

参考[On_Demand_Resources_Guide](https://developer.apple.com/library/prerelease/ios/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/index.html)发现`Resources Tags`是Xcode7新增，主要作用是给资源文件添加标签，集合iOS9新增的API`NSBundleResourceRequest`，达到按需下载资源的目的。  


#流程
1. 确定资源分级，哪些是必须的，哪些是延迟加载的，哪些是可以远程的资源。
2. Xcode的Resource Tags选项中，添加Tag，并在`Prefetched`设置优先级，`Images.xcassets`里的图片也可以设置Tags。  
3. 使用`NSBundleResourceRequest`根据`tags`获取资源，它是iOS9新增类，具体参考[NSBundleResourceRequest_Class](https://developer.apple.com/library/prerelease/ios/documentation/Foundation/Reference/NSBundleResourceRequest_Class/index.html)，重点有初始化、下载、优先级、`progress`属性和结束。
4. 配置存放资源的服务端。
4. 通过`Xocde Archive`工具导出`Asset Packs`，将`asset packs`拷贝到自己的服务端。
5. Xcode `Build Settings`里设置服务端`asset URL`。

#思考
苹果文档中拿了游戏作为例子，在第一关的时候，下载第二关的资源文件，对游戏而言，真实的减少了包的大小。  
有大资源的应用可以根据版本使用起来，少量资源文件的应用还想到有哪些使用场景？根据不同情况加载不同的资源包？  
`On-Demand Resources`支持`Data file`、`Image`、`OpenGL shader`、`SpriteKit *`、`WatchKit complication`以及**`Apple TV Image Stack`**，感觉Apple在下一盘很大的棋。  

#其他
一篇翻译一半的中文版资料：[按需加载资源开发指南-@BenBeng](http://benbeng.leanote.com/post/On-Demand-Resources-Guide)  
实战Demo待补充，还没来得及写。  