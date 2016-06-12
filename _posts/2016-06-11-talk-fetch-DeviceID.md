---
layout: post
title:  "谈谈DeviceID安全的获取方式"
date:   2016-06-11 00:00:00
categories: iOS
published: true
---
 
```
/** 获取设备标识符 */
+ (NSString *)deviceId;
```

大部分App都需要获取设备标识符用于统计及区分设备，必要的情况可以锁掉对应的设备。  

我们现在常用的取值方式有：UDID、MAC Address、OpenUDID、IDFA、IDFV、IMEI等，不同的取值方式各有利弊，不太明白的自己搜索下，如何让一个值一直保存切升级系统都不变呢？一般使用KeyChain，大概的代码如下:  

```
--伪代码
NSString *deviceId = [MFSKeyChain objectForKey:@"deviceId"];
        if (!deviceId.length) {
            deviceId = [[NSUserDefaults standardUserDefaults] objectForKey:@"deviceId"];
            if (!deviceId.length) {
                deviceId = [[UIDevice currentDevice] identifierForVendor].UUIDString;
                if (!deviceId.length) {
                    deviceId = [[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString];
                    if (!deviceId.length) {
                        deviceId = @"备选方案，Token等";
                    }
                }
            }
        }
        if (deviceId.length) {
            [MFSKeyChain setObject:deviceId forKey:keyChain_Key];
            [[NSUserDefaults standardUserDefaults] setObject:deviceId forKey:@"deviceId"];
        }
        return deviceId ?: @"";
        
```

这样的代码用了很长一段时间都没有问题，但是不知道从什么时候起，不同的设备出现了多个deviceId（至今未查出问题所在），并且不好排查，更新逻辑后至今未发现新的问题。  

```
--伪代码(大概是这个样子)
NSString *const kMFSIdentifierCacheDeviceIdKey = @"com.imfong.Device.cacheKey";
NSString *const kMFSIdentifierUserDeviceIdKey = @"com.imfong.deviceId.defaultkey";
NSString *const kMFSIdentifierKeyChainKey = @"com.imfong.deviceId.KeyChain.Key";

+ (NSString *)deviceId
{
    NSString *cacheDeviceId = [cacheManager() objectForKey:kMFSIdentifierCacheDeviceIdKey];
    NSString *userDeviceId = [userDefaults() objectForKey:kMFSIdentifierUserDeviceIdKey];
    //缓存和UserDefaults数据一致，验证通过
    if (cacheDeviceId.length && userDeviceId.length && [cacheDeviceId isEqualToString:userDeviceId]) {
        return cacheDeviceId;
    }
    else {
        NSString *keyChain_Key = kMFSIdentifierKeyChainKey;
        NSString *deviceId = [MFSKeyChain objectForKey:keyChain_Key];
        if (!deviceId.length) {
            deviceId = cacheDeviceId;
            if (!deviceId.length) {
                deviceId = [[UIDevice currentDevice] identifierForVendor].UUIDString;
                if (!deviceId.length) {
                    deviceId = [[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString] ?: @"";
                    if (!deviceId.length) {
                        deviceId = @"devicePushToken值";
                    }
                }
            }
        }
        if (deviceId.length) {
            [MFSKeyChain setObject:deviceId forKey:keyChain_Key];
            [cacheManager() setObject:deviceId forKey:kMFSIdentifierCacheDeviceIdKey duration:-9];
            [userDefaults() setObject:deviceId forKey:kMFSIdentifierUserDeviceIdKey];
        }
        return deviceId ?: @"";
    }
}

#pragma mark -
static MFSCacheManager *cacheManager() {
    static id instance;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[MFSCacheManager alloc] initWithSuiteName:@"com.imfong.Identifier.Cache"];
    });
    return instance;
}

static NSUserDefaults *userDefaults() {
    static id instance;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        instance = [[NSUserDefaults alloc] initWithSuiteName:@"com.imfong.Identifier.userDefaults"];
    });
    return instance;
}
```  

将deviceId加密（最好2种加密方式）后保存到本地的2个地方，需要的时候取出来对比，相同证明没有篡改(全部被篡改就证明你的App太不安全了)直接使用，不同则使用KeyChain或者其他的方法获取并保存本地。  
这种方式在用户正常使用的情况下，不删除App的系统升级也可以正常保持deviceId的正确，在新系统中只要打开一次，KeyChain就会记录新值，删掉App重新下载也是不会变的，当然，漏洞还会有，并不是完美方案。  