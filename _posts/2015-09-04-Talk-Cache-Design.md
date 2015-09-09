---
layout: post
title:  "谈谈移动端Cache设计"
date:   2015-09-04 21:30:46
categories: iOS
published: true
---

>我们的应用都包含了大量的数据，使用缓存可以让应用程序更快速的响应数据展示、用户输入，使程序高效的运行；

#前提
随着项目功能变多变复杂，客户端的存储方式不再统一，功能各不相同；

###存储方式
1. 最常用的是NSUserDefaults
2. 归档的对象需要遵守`NSCoding`协议，使用`NSKeyedArchiver`和`NSKeyedUnarchiver`来归档和解压
3. 数据序列化成`NSData`，`writeToFile`存储至文件
4. DataBase，如SQLite

###功能
1. 数据的时效性
2. 数据安全性
3. 数据的不同类型，存储形式不同
4. 大量数据的效率

###统一
我觉得有必要统一存储方式，只开放一个类，这个类能满足所有的功能；

1. 数据的生命周期
2. 统一缓存空间
3. 数据安全存储
4. 不同的等级
5. 尽可能多的数据类型

#开工

###FileStorageObject

	typedef NS_ENUM(NSUInteger, MFSFileStorageObjectTimeOutInterval) {
	    MFSFileStorageObjectIntervalDefault,
	    MFSFileStorageObjectIntervalTiming,     //定时
	    MFSFileStorageObjectIntervalAllTime     //永久
	};
	
	@interface MFSFileStorageObject : NSObject <NSCoding>
	
	/** 数据String */
	@property (nonatomic, copy, readonly) NSString *storageString;
	/** 数据对象 */
	@property (nonatomic, strong, readonly) id storageObject;
	/** 数据的存储时效性 */
	@property (nonatomic, assign, readonly) MFSFileStorageObjectTimeOutInterval storageInterval;
	
	/** 当前对象的标识符（KEY），默认会自动生成，可自定义*/
	@property (nonatomic, copy) NSString *objectIdentifier;
	
	/** 存储文件的过期时间
	 *  -1表示永久文件，慎用 */
	@property (nonatomic, assign) NSTimeInterval timeoutInterval;
	
	/** 根据（String,URL,Data,Number,Dictionary,Array,Null,实体）初始化对象 */
	- (instancetype)initWithObject:(id)object;
	
	@end

`FileStorageObject`主要作用是存储数据对象，根据`-initWithObject:`创建`FileStorageObject`将所有的数据对象进行转换；  

支持的类型有NSString、NSURL、NSData、NSNumber、NSDictionary、NSArray、NSNull、自定义实体实体，这些类型其实是做了一层转换，都是转换成String，即`storageString`；  

`storageString`本质上是一段JSON，进行AES加密，这样的一个好处是传入的自定义对象不再需要实现`NSCoding`协议，取`storageObject`对象时再进行AES解析(`NSString+MFSEncrypt`里设置`MFSDefaultAESKey`和`AES_IV`)；  

`storageObject`是根据`storageString`这段JSON生成的对象，每次取值都是重新生成，好处是每次获取的新对象改变不会改变原存储的值，如果需要改变，重新存即可；  

`storageInterval`是存储的数据的失效，包含默认、定时和永久，这边定义的普通可被全局清理，定时的数据自动清理，而永久的不允许被全局清理，但是可被自己的对象根据`KEY`清理，这个后面说；  

`objectIdentifier`表示对象标识符，无实际作用，只是区分不同的对象，默认根据`storageString`的`md5`生成；  

`FileStorageObject`实现了`<NSCoding>`协议，让这个对象可被序列化存储；  

###FileStorage

	#import "MFSFileStorageObject.h"
	
	typedef NS_ENUM(NSUInteger, MFSFileStorageType) {
	    MFSFileStorageCache         = 0,    //Memory
	    MFSFileStorageArchiver
	};
	
	@interface MFSFileStorage : NSObject
	
	@property (nonatomic, strong) NSString *suiteName;  //空间
	
	+ (instancetype)defaultStorage;
	
	/** MFSFileStorageType默认为MFSFileStorageArchiver */
	- (void)setObject:(MFSFileStorageObject *)aObject forKey:(NSString *)aKey;
	- (void)setObject:(MFSFileStorageObject *)aObject forKey:(NSString *)aKey type:(MFSFileStorageType)t;
	
	- (MFSFileStorageObject *)objectForKey:(NSString *)aKey;
	
	- (void)removeObjectForKey:(NSString *)aKey;
	
	//删除所有的默认文件，常用方法
	- (void)removeDefaultObjectsWithCompletionBlock:(void (^)(long long folderSize))completionBlock;
	//删除过期的文件
	- (void)removeExpireObjects;
	
	/** 对所有空间做操作 */
	/** 删除所有的默认文件，谨慎操作 */
	+ (void)removeDefaultObjectsWithCompletionBlock:(void (^)(long long folderSize))completionBlock;
	/** 删除过期的文件，谨慎操作 */
	+ (void)removeExpireObjects;
	
	@end

`FileStorage`主要是对`FileStorageObject`对象操作，因为`FileStorageObject`实现了`<NSCoding>`协议，所以此处直接将`FileStorageObject`对象存储到文件中（为了效率，有兴趣的可以试试其他存储方式）；  

使用`- setObject:forKey:`存储`FileStorageObject`对象，下次再通过`-objectForKey:`根据`key`取出`FileStorageObject`对象；  

`FileStorageType`的作用是存储类型，是载入内存还是硬盘，集合`+defaultStorage`使用，在内存中存储`FileStorageObject`对象；  

`remove`方法有3种：  
* `-removeObjectForKey`是根据KEY删除数据，可删除永久级别的数据；  
* `-removeDefaultObjectsWithCompletionBlock:`是删除当前`FileStorage`对象内的所有默认存储数据，返回数据大小供客户端提示；  
* `-removeExpireObjects`删除所有过期的数据，异步的；  

`suiteName`有非常重要的作用，区分不同的类别区域，将数据存储到不同的目录中，使用相同的`KEY`在不同的`FileStorage`对象中取得不同的数据，同`NSUserDefaults`里的SuiteName；  

`+removeDefaultObjectsWithCompletionBlock:`和`+removeExpireObjects`是对整个数据目录进行操作，范围很广；

###CacheManager

	extern NSString * const MFSCacheManagerObject;
	extern NSString * const MFSCacheManagerObjectKey;
	extern NSString * const MFSCacheManagerSetObjectNotification;   //缓存存数据
	extern NSString * const MFSCacheManagerGetObjectNotification;   //缓存取数据
	extern NSString * const MFSCacheManagerRemoveObjectNotification;//移除缓存
	
	@interface MFSCacheManager : NSObject
	
	/** 默认缓存管理器 */
	+ (MFSCacheManager *)defaultManager;
	
	/** nil suite means use the default search list that +defaultManager uses */
	- (instancetype)initWithSuiteName:(NSString *)suitename;
	
	/** 根据Key缓存对象，默认duration为0：对象一直存在，清理后失效，object为nil则removeObject
	 *  @param aObject 存储对象，支持String,URL,Data,Number,Dictionary,Array,Null,自定义实体类
	 *  @param aKey    唯一的对应的值，相同的值对覆盖原来的对象 */
	- (void)setObject:(id)aObject forKey:(NSString *)aKey;
	/** 存储的对象的存在时间，duration默认为0，传-1，表示永久存在，不可被清理，只能手动移除或覆盖移除 
	 *  @param duration 存储时间，单位:秒 */
	- (void)setObject:(id)aObject forKey:(NSString *)aKey duration:(NSTimeInterval)duration;
	
	/** 存储全局临时对象，不序列化，不占用硬盘空间 */
	- (void)setGlobalObject:(id)aObject forKey:(NSString *)aKey;
	
	/** 根据Key获取对象(数据相同内存值不同) */
	- (id)objectForKey:(NSString *)aKey;
	
	/** 根据Key移除缓存对象 */
	- (void)removeObjectForKey:(NSString *)aKey;
	
	/** 异步移除所有duration为0的缓存
	 folderSize单位是字节，转换M需要folderSize/(1024.0*1024.0) */
	- (void)removeObjectsWithCompletionBlock:(void (^)(long long folderSize))completionBlock;
	/** 异步检查缓存(duration大于0)的生命，删除过期缓存，建议App启动使用 */
	- (void)removeExpireObjects;
	
	/** 不区分空间，对所有数据进行删除，影响甚广，谨慎操作 */
	+ (void)removeObjectsWithCompletionBlock:(void (^)(long long folderSize))completionBlock;
	/** 不区分空间，对所有缓存进行检查，谨慎操作 */
	+ (void)removeExpireObjects;
	
	@end

`CacheManager`本质是对`FileStorage`操作；  

开放的方法有注释，一看就明白，主要有几点需要注意：  
1. 开放`+defaultManager`供普通数据存储，可被全局清理；  
2. 需要存储不同空间的，`CacheManager`使用`-initWithSuiteName:`初始化；  
3. `-setObject:forKey:duration:`通过`duration`来控制数据的有效时间，单位是秒，如果传入负数，说明数据永久，只能通过Key使用`-removeObjectForKey:`移除，`-removeObjectsWithCompletionBlock:`无法移除；  
4. 使用`-setGlobalObject:forKey:`的对象不进行文件存储，只存在内存中，他和GlobalManager的区别在于是否需要经过`FileStorageObject`转换；  

###GlobalManager

	@interface MFSGlobalManager : NSObject
	
	/** 根据Key缓存对象，object为nil则removeObject
	 *  @param aObject 存储对象，支持对象类型
	 *  @param aKey    唯一的对应的值，相同的值对覆盖原来的对象 */
	+ (void)setObject:(id)aObject forKey:(NSString *)aKey;
	
	/** 根据Key获取对象(数据相同内存值相同) */
	+ (id)objectForKey:(NSString *)aKey;
	
	/** 根据Key移除缓存对象 */
	+ (void)removeObjectForKey:(NSString *)aKey;
	
	@end	

每个客户端都需要保存一些全局数据，一般的做法是创建一个单例对象来保存，`GlobalManager`就是这个作用；  

#后续
本篇其实不是设计Cache，而是介绍了本人写的CachaManager作用有哪些；  
`CacheManager`和`GlobalManager`包含的功能基本上满足了一个App的数据缓存系统，比如用户信息，可根据UserID设置suiteName，每个用户账号都有不同的数据空间，而最好不要对用户信息使用单例存储；  

MFSCache源码请访问[Github](https://github.com/maxfong/MFSCache)，感谢你提出更好的建议；    
