---
layout: post
title:  "私有Pods集成react-native库"
date:   2017-06-16 00:00:00
categories: iOS react-native
published: true
---

>将react-native移入私有源后，原生语言开发者不再需要安装`Node`、配置`npm`等环境。  

## 获取node_modules

### 安装React开发环境

创建私有库的开发者需要安装React开发环境  

```
brew install node   
brew install watchman  
npm install -g react-native-cli
```

具体参考[React Native Quick Start](http://facebook.github.io/react-native/docs/getting-started.html)

### 使用npm package  

这里使用了0.44.3版本  

```
{
  "name": "Demo1",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "dependencies": {
    "react": "16.0.0-alpha.6",
    "react-native": "0.44.3"
  }
}
```  

执行`npm install`


## 获取React.podspec.json


在`node_modules/react-native`找到`React.podspec`  

```
1.`0.44.3`版本内`cocoapods_version = ">= 1.2.0"`, pod版本不够的需升级  
2.`Core`依赖`Yoga`,需要做好支持
```  

使用`$ pod ipc spec React.podspec >> React.podspec.json`得到JSON文件  
将`source`中的`git`地址改成私有库地址  

```
"source": {
	"git": "git@git.hostxx.com:group/react-native.git",
	"tag": "v0.44.3"
},
``` 

上传`React.podspec.json`到私有repo  


### Yoga  

在`node_modules/react-native/ReactCommon/yoga`内找到`Yoga.podspec`  
使用`pod ipc spec`转成`Yoga.podspec.json`  

修改`source`  

```
"source": {
	"git": "git@git.17usoft.com:wireless-clientapp/react-native.git",
	"tag": "v0.44.3"
},
```

修改`source_files`，路径添加`ReactCommon`  

```
"source_files": "ReactCommon/yoga/**/*.{c,h}"
```

上传`Yoga.podspec.json`到私有repo


## 使用  

原生开发者在`Podfile`中添加 

```
pod 'React', '0.44.3', :subspecs => [
    'Core',
    'DevSupport',
    'RCTText',
    'RCTNetwork',
    'RCTWebSocket',
    #...
]

```

其他开发者执行`pod update`即可正常运行应用，无需再执行前面React环境安装。  

### 嵌入到应用  

```
#import "RCTBundleURLProvider.h"
#import "RCTRootView.h"

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.navigationItem.title = @"这个是RN页面";
    NSURL *jsURL = [NSURL URLWithString:@"http://127.0.0.1:8081/index.ios.bundle?platform=ios&dev=true"];
    RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsURL
                                                        moduleName:@"XXXXXXXX"
                                                 initialProperties:nil
                                                     launchOptions:nil];
    rootView.frame = self.view.frame;
    [self.view addSubview:rootView];
}

```  

原生开发者只需配置好链接，无需再关注react开发者进度。  

博客地址:https://imfong.com/post/Private-Pods-Add-react-native  
