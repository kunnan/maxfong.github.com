---
layout: post
title:  "QQMusic for Mac V5.0.0.3"
date:   2017-06-08 00:00:00
categories: Mac
published: true
---


# 功能  
1.收听并下载高品质、无损品质的音乐    
2.免费收听收费的音乐  

# 支持版本  
v4.2.3.3  
v5.0.0.3  

# 修改  
使用`Hopper Disassembler`修改以下  

```
-[UrlAudioData currentSongRateType] ret 0
-[DownLoadTask currentSongRateType] ret 0
-[DownloadInfo haveCheckDownloadRight] ret 1
-[DownloadInfo haveDownloadRight] ret 1
-[SongInfo isActionBitSet:forSwitch:] ret 1
```

# 其它  
仅供学习研究交流之用，勿作商业用途  
