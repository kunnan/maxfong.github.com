---
layout: post
title:  "谈谈App如何获取当前时间"
date:   2016-07-04 00:00:00
categories: iOS
published: true
---

>[NSDate date]获取的是设备时间，我们如果获取到当前正确的时间呢？  

本文的方式是通过接口（可以放在App所有response的HEAD）获取服务端当前时间，和[NSDate date]的时间生成间隔，正确的时间就是[NSDate date]加间隔（可能是负）。  

```
@interface MFSNetworkDate : NSObject

/** 网络服务器时间 */
+ (NSDate *)date;

/** 根据时间间隔生成新的Date */
+ (NSDate *)dateWithTimeIntervalSinceNow:(NSTimeInterval)secs;

/** 当前时间和传入时间的间隔 */
+ (NSTimeInterval)timeIntervalSinceNowToDate:(NSDate *)date;

@end

```  

###实现方式的步骤  
1. 获取服务接口的rspTime，计算成serverDate（`[NSDate dateWithTimeIntervalSince1970:[headDictionary[@"rspTime"] doubleValue]/1000]`）。  
2. 根据服务端的Date和当前设备时间生成timeInterval（`serverDate.timeIntervalSinceNow`）。  
3. [MFSNetworkDate date]获取时间时，判断serverDate是否存在，存在则返回当前时间（`[NSDate dateWithTimeIntervalSinceNow:timeInterval];`），否则返回设备时间(`[NSDate dateWithTimeIntervalSinceNow:0];`)。  

###时间扩展方法实现   
```
+ (NSDate *)dateWithTimeIntervalSinceNow:(NSTimeInterval)secs {
    return [NSDate dateWithTimeInterval:secs sinceDate:[self date]];
}

+ (NSTimeInterval)timeIntervalSinceNowToDate:(NSDate *)date {
    return [date timeIntervalSinceDate:[self date]];
}

```

###其他  
1. 不要使用`Swizzle`的方式替换[NSDate data]的实现方式，因为App在启动的时候，已经使用了[NSDate data]获取的设备时间，当服务端返回新的值时，[NSDate data]新值和久值之间的间隔可能是负数，这能引起很多莫名其妙的问题，比如手势长按变成了点击。
2. 使用`[NSDate dateWithTimeIntervalSinceNow:0]`就是为了防止第一条的递归。  
3. 每次请求新的接口，响应时都会更新serverDate和timeInterval，所以哪怕在App启动后修改系统时间，刷新页面，请求任意接口，都能返回正确的时间。  
4. 服务端使用类似[[NSDate date] timeIntervalSince1970]的方式，排除时区的问题。  
5. [MFSNetworkDate date]和[NSDate date]分开实现，各取所需。  


