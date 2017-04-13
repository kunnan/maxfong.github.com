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
        const char* class_name = class_getName([self class]);
        Class metaClass = objc_getMetaClass(class_name);
        
        SEL originalSelector = @selector(URLWithString:);
        SEL newSelector = @selector(p_URLWithString:);

        Method originalMethod = class_getInstanceMethod(metaClass, originalSelector);
        Method newMethod = class_getInstanceMethod(metaClass, newSelector);
        
        BOOL methodAdded = class_addMethod([metaClass class],
                                           originalSelector,
                                           method_getImplementation(newMethod),
                                           method_getTypeEncoding(newMethod));
        
        if (methodAdded) {
            class_replaceMethod([metaClass class],
                                newSelector,
                                method_getImplementation(originalMethod),
                                method_getTypeEncoding(originalMethod));
        }
        else {
            method_exchangeImplementations(originalMethod, newMethod);
        }
      });
    }

	
	+ (instancetype)p_URLWithString:(NSString *)URLString {
		//TODO:...
	    return [NSURL p_URLWithString:URLString];
	}
	
	@end

#相关知识
[iOS Method Swizzling](https://www.baidu.com/s?ie=UTF-8&wd=ios%20method%20swizzling)  
[刨根问底Objective－C Runtime（2）－ Object & Class & Meta Class](http://chun.tips/blog/2014/11/05/bao-gen-wen-di-objective[nil]c-runtime-(2)[nil]-object-and-class-and-meta-class/)
