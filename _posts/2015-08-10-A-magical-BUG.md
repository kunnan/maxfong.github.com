---
layout: post
title:  "一个神奇的BUG"
date:   2015-08-10 19:20:56
categories: iOS
---
临近发布，Xcode打出的包在不同的设备效果不同，具体表现有：

- 部分页面的数据不显示  
- 相同的包，安装到不同的设备中，表现不一  
- 不同的设备随机引起crash

对比下来发现一些情况  

- iPhone5s及以上完全正常  
- iphone5及以下设备部分页面闪退及数据显示不完整  

## 排查过程
首先怀疑界面约束问题，因为不同版本的Xcode支持的约束不同，特别排查了那些运行时log提示的那些界面，发现不是根本问题，用`FLEX`排查，发现控件都存在，真的是数据不见了；  

排查中间层返回数据值，JSON一切正常；  

JOSN解析成Entity对象，发现对象4个属性值少了一个，定位到网络解析库；  

网络framework库内创建一个demo可运行app，在iPhone5内解析一段JSON，最终定位到一行代码:

	const char *propertyAttributes = property_getAttributes(property);
	BOOL isReadWrite = YES;
	isReadWrite = !(BOOL)strstr(propertyAttributes, ",R");
	isReadWrite = (BOOL)strstr(propertyAttributes, ",V");
	if (isReadWrite) {
		//...
	}

代码的本意是判断当前属性是否为readonly且不可赋值的属性，不对其进行KVC赋值；  
而且`,R`的代码毫无作用；  
各位看官可能已经看出来了，这段代码有一个非常严重的问题就是**strstr函数**的返回其实是匹配字符串的地址，不匹配返回NULL；  
既然是地址，强转bool其实是风险的，最终代码修改为：

	const char *propertyAttributes = property_getAttributes(property);
	BOOL isReadWrite = (strstr(propertyAttributes, ",V") != NULL);
	if (isReadWrite) {
		//...
	}

<!--详解Property的R、N、V-->

## 解析
查看BOOL在runtime内的定义：

	/// Type to represent a boolean value.
	#if !defined(OBJC_HIDE_64) && TARGET_OS_IPHONE && __LP64__
	typedef bool BOOL;
	#else
	typedef signed char BOOL; 
	// BOOL is explicitly signed so @encode(BOOL) == "c" rather than "C" 
	// even if -funsigned-char is used.
	#endif

在64 bit 为bool类型，其他为char；  
即在32 bit的机器上，任意地址的后2位是00，转换返回NO  
而64 bit，非0就是true；  
这也证明了iphone5及以上正常；

## 检查readonly？
KVC赋值检查对象属性readonly也是因为有次服务端在响应实体对象中，返回了description字段；  
而description对于iOS应该算一个关键字;  
继承自NSObject的子类如果属性有description且没有实现`@synthesize description;`,KVC set的时候会crash，判断attributes是否包含`V`属性证明可赋值和取值；

## 后续
容易忽略的、最基础的知识引起的BUG越可怕，越不容易发现；  
这个BUG也应该是我从业几年遇见的最神奇的BUG，以此纪念！
