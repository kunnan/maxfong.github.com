---
layout: post
title:  "谈谈iOS启动连续闪退保护方案"
date:   2016-07-26 00:00:00
categories: iOS
published: true
---

>“如果某个实体表现出以下任何一种特性，它就具备自主性：自我修复、自我保护、自我维护、对目标的自我控制、自我改进。” —— 凯文·凯利

##阅读  
[iOS启动连续闪退保护方案](http://www.infoq.com/cn/articles/ios-booting-protection)

##基于Time的实现方式  

####H文件  
```
#import <Foundation/Foundation.h>

typedef void(^MFSBootingProtectionClearBlock)(void);

extern NSString *const MFSBootingProtectionClearNotification;

@interface MFSBootingProtection : NSObject

/** 开启启动闪退统计，达到3次触发MFSBootingProtectionClearNotification */
+ (void)startWihtClearBlock:(MFSBootingProtectionClearBlock)block;

/** 手动触发检查，默认为NO，YES表示需要清理 */
+ (BOOL)launchClear;

#ifdef DEBUG
/**
 *  触发清理通知，仅供测试
 */
+ (void)triggerClear;
#endif

@end
```

####M文件  
```
NSString *const MFSBootingProtectionClearNotification = @"MFSBootingProtectionClearNotification";
NSString *const kBPCrashCountKey = @"kCrashCountKey";
int const kBPCrashWaitTime = 5;
int const kBPCrashWaitCount = 2;

@implementation MFSBootingProtection

+ (void)startWihtClearBlock:(MFSBootingProtectionClearBlock)block {
    int count = [self launchFailCount];
    if (count >= kBPCrashWaitCount) {
        [[NSNotificationCenter defaultCenter] postNotificationName:MFSBootingProtectionClearNotification object:nil];
        if (block) { block(); }
    }
    else {
        [[NSUserDefaults standardUserDefaults] setObject:@(count++) forKey:kBPCrashCountKey];
    }
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(kBPCrashWaitTime * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [[NSUserDefaults standardUserDefaults] setObject:@(0) forKey:kBPCrashCountKey];
    });
}

+ (int)launchFailCount {
    NSNumber *num = [[NSUserDefaults standardUserDefaults] objectForKey:kBPCrashCountKey];
    return num.intValue;
}

+ (BOOL)launchClear {
    int count = [self launchFailCount];
    return (count >= kBPCrashWaitCount);
}

#ifdef DEBUG
+ (void)triggerClear {
    [[NSNotificationCenter defaultCenter] postNotificationName:MFSBootingProtectionClearNotification object:nil];
}
#endif

@end

```

##启动  
```
//开启闪退保护
    [TCTBootingProtection startWihtClearBlock:^{
        //做一些Clear的事情，参考阅读内容
    }];
```
  
##其他  
1.某些库如hotfix加载脚本，需判断`[TCTBootingProtection launchClear]`才可加载脚本，否则做脚本清理的问题。  
2.MFSBootingProtectionClearNotification的作用是供有单例的库做清理。  
2.无法正确的知道crash问题，只能做一些清理。  
