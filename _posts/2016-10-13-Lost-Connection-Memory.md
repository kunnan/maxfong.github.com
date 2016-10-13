---
layout: post
title:  "Xcode8：Lost connection to “iPhone Name”"
date:   2016-10-13 00:00:00
categories: iOS
published: true
---

>Lost connection to “iPhone Name”.
>Restore the connection to “iPhone Name” and run “APP Name” again, or if “APP Name” is still running, you can attach to it by selecting Debug > Attach to Process > “APP Name”.

连接真机调试状态突然提示这个，经过排查是因为升级了Xcode8后，Interface Builder的Opens in 自动设置为`Latest Xcode (8.0)`，会对XIB做了一些版本设置，导致多个View`-[UINit instantiateWithOwner:options:]`时调用`CGDataProviderCreateWithCopyOfData`分配内存过大，引起crash，设置为`Xcode 7.x`后APP运行并调试正常，有相同问题的可以试试，具体原因未查明，知道的告知下，谢谢！
