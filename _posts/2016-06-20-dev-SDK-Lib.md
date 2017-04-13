---
layout: post
title:  "谈谈开发SDK的注意点"
date:   2016-06-20 00:00:00
categories: iOS
published: true
---

1. 添加第三方开源的源码库，需修改类名，增加TCT前缀，尽量使用Pod不要直接添加第三方源码。
2. 所有类名、类别名及类别方法名都需要添加前缀。
3. 发现某个方法命名比较困难，那么肯定这个方法耦合度太高，需要再次分解。
4. 需要考虑升级的问题，并且指定某些版本强制升级，不可直接删除旧的方法，只可添加新的替代方法，并旧方法标识(DEPRECATED_MSG_ATTRIBUTE("使用-xxx替代"))。
5. 开放接口，减少对外开放类，只暴露需要使用的类，并且注释及参数一定要写明白。
6. 支持最新特性，64位和Bitcode等。
7. 提供可运行、可测试的Demo。
8. 留一个后门请求接口，用于关闭，防止恶意攻击，或增加统计功能。
9. 使用条件编译就显示，不要运行再提示

```
#warning - Release scheme, this is not work.

#if !__has_feature(objc_arc)
#error xxx requires automatic reference counting
#endif
```
