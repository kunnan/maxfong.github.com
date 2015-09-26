---
layout: post
title:  "Hook Objective-C 类方法"
date:   2015-09-24 00:00:00
categories: iOS
published: true
---

以`NSURL`的`URLWithString:`为例：  

	#import <objc/runtime.h>

	@implementation NSURL (URLWithStringEncode)
	
	+ (void)load {
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^ {
	        const char* className = class_getName([self class]);
	        Class metaClass = objc_getMetaClass(className);
	        
	        Method oldMethod = class_getInstanceMethod(metaClass, @selector(URLWithString:));
	        Method newMethod = class_getInstanceMethod(metaClass, @selector(p_URLWithString:));
	        
	        method_exchangeImplementations(oldMethod, newMethod);
	    });
	}
	
	+ (instancetype)p_URLWithString:(NSString *)URLString {
	    NSString *result = (NSString *)CFBridgingRelease(CFURLCreateStringByReplacingPercentEscapesUsingEncoding(kCFAllocatorDefault, (CFStringRef)URLString, CFSTR(""), kCFStringEncodingUTF8));
	    NSString *newURLString = [result stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
	    return [NSURL p_URLWithString:newURLString];
	}
	
	@end

#相关知识
[iOS Method Swizzling](https://www.baidu.com/s?ie=UTF-8&wd=ios%20method%20swizzling)  
[刨根问底Objective－C Runtime（2）－ Object & Class & Meta Class](http://chun.tips/blog/2014/11/05/bao-gen-wen-di-objective[nil]c-runtime-(2)[nil]-object-and-class-and-meta-class/)