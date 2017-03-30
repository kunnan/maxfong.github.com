---
layout: post
title:  "谈谈iPhone Model的获取方式"
date:   2017-03-30 00:00:00
categories: iOS
published: true
---

客户端内获取platformString的方式一般为写死默认数据，这样出现的问题是当apple发布新设备而使用旧的客户端，会无法统计到新的Generation，以下的代码使用`TFHpple`动态的解析`https://www.theiphonewiki.com/wiki/Models`的HTML，能获取到最新platform。

```
#import "TFHpple.h"

+ (void)updateMachinePlatformSource {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSData *data = [NSData dataWithContentsOfURL:[NSURL URLWithString:@"https://www.theiphonewiki.com/wiki/Models"]];
        TFHpple *hpple = [[TFHpple alloc] initWithHTMLData:data];
        NSMutableArray *matrix = [NSMutableArray array];
        NSArray<TFHppleElement *> *wikitables = [hpple searchWithXPathQuery:@"//*[@id=\"mw-content-text\"]/table[@class=\"wikitable\"]"];
        [wikitables enumerateObjectsUsingBlock:^(TFHppleElement * _Nonnull wikitable, NSUInteger idx, BOOL * _Nonnull stop) {
            NSArray<TFHppleElement *> *trs = [wikitable searchWithXPathQuery:@"//tr"];
            NSInteger trCount = trs.count;
            NSInteger thCount = [wikitable searchWithXPathQuery:@"//tr//th"].count;
            NSMutableArray *tmpY = [NSMutableArray array];
            for (int y = 0; y < trCount; y++) {
                NSMutableArray *tmpX = [NSMutableArray array];
                for (int x = 0; x < thCount; x++) {
                    [tmpX addObject:[NSNull null]];
                }
                [tmpY addObject:tmpX];
            }
            [trs enumerateObjectsUsingBlock:^(TFHppleElement * _Nonnull tr, NSUInteger trid, BOOL * _Nonnull stop) {
                NSArray<TFHppleElement *> *tds = tr.children;
                [tds enumerateObjectsUsingBlock:^(TFHppleElement * _Nonnull td, NSUInteger tdid, BOOL * _Nonnull stop) {
                    NSString *content = [td.content stringByReplacingOccurrencesOfString:@"  " withString:@""];
                    if (![content isEqualToString:@"\n"]) {
                        content = [content stringByReplacingOccurrencesOfString:@"\n" withString:@""];
                        NSUInteger index = tdid / 2;
                        id placeObj = tmpY[trid][index];
                        if ([placeObj isKindOfClass:[NSNull class]]) {
                            tmpY[trid][index] = content;
                        }
                        else {
                            NSMutableArray *tmp = tmpY[trid];
                            __block NSInteger nextnullid = -1;
                            [tmp enumerateObjectsUsingBlock:^(id  _Nonnull place, NSUInteger idx, BOOL * _Nonnull placeStop) {
                                if ([place isKindOfClass:[NSNull class]]) {
                                    nextnullid = idx;
                                    *placeStop = YES;
                                }
                            }];
                            if (nextnullid >= 0) {
                                index = nextnullid;
                                tmpY[trid][nextnullid] = content;
                            }
                        }
                        NSDictionary *attributes = td.attributes;
                        NSString *rowspan = attributes[@"rowspan"];
                        if (rowspan.intValue > 0) {
                            for (int i = 1; i< rowspan.intValue; i++) {
                                tmpY[trid+i][index] = content;
                            }
                        }
                    }
                }];
            }];
            [matrix addObject:tmpY];
        }];
        
        if (matrix.count) {
            NSMutableDictionary *dict = [NSMutableDictionary dictionary];
            [matrix enumerateObjectsUsingBlock:^(NSArray *  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
                __block NSUInteger gid = 9999;
                __block NSUInteger iid = 9999;
                [obj enumerateObjectsUsingBlock:^(NSArray *  _Nonnull obj1, NSUInteger idx, BOOL * _Nonnull stop) {
                    if (idx == 0) {
                        gid = [obj1 indexOfObject:@" Generation"];
                        iid = [obj1 indexOfObject:@" Identifier"];
                    }
                    else {
                        if (gid != 9999 && iid != 9999) {
                            [dict setValue:obj1[gid] forKey:obj1[iid]];
                        }
                    }
                }];
            }];
            
            NSLog(@"最终有效键值对：\n%@", dict);
            //存储缓存并设置很长有效时间
            
        }
    });
}
```


### 其他
并不建议直接将这段代码直接写在客户端内，最好是服务端开放接口，使用缓存返回给客户端。  
TD标签解析的时候比较麻烦，因为有`rowspan `属性，所以直接二维数组。  
没有高效可谈，有很多需要优化的，这部分代码只是取值可用。  
