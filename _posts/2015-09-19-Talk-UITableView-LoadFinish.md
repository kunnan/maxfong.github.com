---
layout: post
title:  "谈谈UITableView LoadFinish"
date:   2015-09-19 00:00:00
categories: iOS
published: true
---
>APM是用来监控和管理应用软件是否有效运行的。-百度百科

# 起因

`UITableView`是我们最常用的控件之一，很多页面的主框架控件就是`TableView`，想要做好APM监控，检测有`TableView`页面的刷新效率、数据加载效率，最好的办法就是得到`TableView`加载完成的效率；

# 过程

### reloadData

可能好多人觉得监控`reloadData`方法的执行时间即可，但其实`reloadData`是异步的，`reloadData`执行完成后，`tableView:cellForRowAtIndexPath:`才开始执行，而我们大部分的渲染以及数据转换工作可能是在`Cell`赋值阶段，我需要知道屏幕上的`Cell`执行完成的时间。  

### 可行的方法

#### 0x01

添加`UITableView+LoadFinish`文件，将`UITableView`的`dataSource`和`delegate`指向自定义对象。  
大概是这样（先拿dataSource做测试）：

	#import "UITableView+LoadFinish.h"
	#import "NSObject+Swizzle.h"
	#import <objc/runtime.h>
	#import "LoadAdapterObject.h"
	
	static char *kLoadAdapterObject = "loadAdapterObject";
	
	@implementation UITableView (LoadFinish)
	
	+ (void)load {
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^
	                  {
	                      SEL setDataSource = @selector(setDataSource:);
	                      SEL mfs_setDataSource = @selector(mfs_setDataSource:);
	                      [self swizzleInstanceSelector:setDataSource withNewSelector:mfs_setDataSource];
	                  });
	}
	
	- (void)mfs_setDataSource:(id)dataSource {
	    LoadAdapterObject *loadAdapterObject = objc_getAssociatedObject(self, kLoadAdapterObject);
	    if (!loadAdapterObject) {
	        loadAdapterObject = LoadAdapterObject.new;
	        objc_setAssociatedObject(self, kLoadAdapterObject, loadAdapterObject, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
	    }
	    loadAdapterObject.dataSource = dataSource;
	    
	    [self mfs_setDataSource:loadAdapterObject];
	}

#### 0x02

将`dataSource`指向`LoadAdapterObject`的实例后，让`LoadAdapterObject`对象来实现TableView的`UITableViewDataSource`和`UITableViewDelegate`所有协议方法。  

`LoadAdapterObject`的实现大概是：

	@implementation LoadAdapterObject
	
	- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
	    if ([self.dataSource respondsToSelector:@selector(tableView:numberOfRowsInSection:)]) {
	        return [self.dataSource tableView:tableView numberOfRowsInSection:section];
	    }
	    return 0;
	}
	
	- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
	    if ([self.dataSource respondsToSelector:@selector(tableView:cellForRowAtIndexPath:)]) {
	        return [self.dataSource tableView:tableView cellForRowAtIndexPath:indexPath];
	    }
	    return nil;
	}
	
	//UITableViewDataSource和UITableViewDelegate的另外方法
	//...
	
	@end

其主要目的是转发所有的`delegate`和`dataSource`实现方法。  

#### 0x03

在`LoadAdapterObject`中，我们可以获取：  
* tableView的高度
* Cell的高度
* Cell执行完成的时间
* Section的数量
* Head和Foot的高度和数量

#### 0x04

经过一系列的计算，可以得出当前屏幕的TableView的函数执行时间，也就是TableView真实渲染和数据加载完成的时间；

### Cell的图片

`Cell`的图片一般都是异步加载，所以我们无法在`Cell`操作完成前检测到。
但其实，我们不需要检测这个时间，因为异步下载图片对页面渲染显示影响比较小，这部分时间不需要统计到APM页面加载中，需要的话，可以单独建立图片下载时间统计。  

### 花絮

如果`TableView`的`dataSource`没有实现`tableView:cellForRowAtIndexPath:`方法，`UITableView`会自动进入在`delegate`中查找`tableView:cellForRowAtIndexPath:`是否实现，如果都为实现，才会crash，有兴趣的看下这个项目[TableView Load Finish Semifinished product](https://github.com/maxfong/UITableViewLoadFinish)。  

	@implementation LoadAdapterObject
	
	- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
	    return 5;
	}
	
	//- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
	//    UITableViewCell *cell = [tableView dequeueReusableHeaderFooterViewWithIdentifier:@"adapterCell"];
	//    if (!cell) {
	//        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"adapterCell"];
	//    }
	//    cell.textLabel.text = @"adapterCellValue";
	//    return cell;
	//}
	
	@end

注释掉`LoadAdapterObject`的`tableView:cellForRowAtIndexPath:`,TableView会查找`delegate`中是否实现`tableView:cellForRowAtIndexPath:`（dataSource不实现`tableView:numberOfRowsInSection:`会直接crash）。  

	- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
	    return 1;
	}
	
	- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
	    UITableViewCell *cell = [tableView dequeueReusableHeaderFooterViewWithIdentifier:@"originCell"];
	    if (!cell) {
	        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"originCell"];
	    }
	    cell.textLabel.text = @"originCellValue";
	    return cell;
	}

上述代码运行后，`TableView`内会有**5**个`Cell`显示**`originCellValue`**。
有兴趣的可以再研究下。  

# 后续
也许有更好的方法，欢迎指出。  
