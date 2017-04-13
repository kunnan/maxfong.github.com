---
layout: post
title:  "谈谈Xcode插件"
date:   2015-09-03 21:30:46
categories: iOS
published: true
---

>同样是使用Xcode，开发效率如何比别人更高效？你需要使用Xcode插件！

不了解的Xcode插件的先看**onevcat**的[Xcode 4 插件制作入门](http://www.onevcat.com/2013/02/xcode-plugin/),详细的介绍了:

* 插件是什么；
* 如何开发一款自己的插件；
* 开发中需要使用的技巧；

###Tests调试
虽然插件无法运行，普通的调试只能通过控制台Log，但实际上你可以自己创建Test，用Test来调试你的功能代码，通过Notification得到数据对象，模拟进行调试，能节省很多时间；

###Notification
插件开发，一定要了解Notification功能，因为Xcode和插件是同一个进程，Xcode的操作发出的通知，插件都能监听到；   
类似的Xcode文本操作，都是监听`IDEEditorDocumentDidChangeNotification`，如[ESJsonFormat-Xcode](https://github.com/EnjoySR/ESJsonFormat-Xcode)，原理是监听了`PBXProjectDidOpenNotification`和`IDEEditorDocumentDidChangeNotification`得到文本路径，弹出窗口对JSON数据进行解析并追加文案到文本中；

###自己的JSON插件
[UtilityTools](https://github.com/maxfong/UtilityTools)是以前写的一个插件，严格来说他并不是一个插件，而只是一个mac软件，借势插入到Xcode中只为了用他的快捷键；  
不同于`ESJsonFormat-Xcode`，他是将所有引用对象生成到独立的.h.m文件中，使用`#import someClass.h`引用，对象重用会方便很多;  
但是最大的弊端是需手动将多个.h.m添加到xcodeProj中，不够自动化，所以直接独立成一个自用的[Mac App-JSONFile](https://github.com/maxfong/JSONFile)了；

###其他
知道将自定义文件通过代码的方式插入到`project.pbxproj`中，类似`fileRef = 4FF8F33F1B400F8B005C73C2`这种值如何获取的，[感谢告知](mailto:devmaxfong@qq.com)；