想要确定 feature 是否被overlay
```
可以使用命令
dumpsys overlay

```

在结果中过滤出我们想要查看的 feature
![[Pasted image 20250313161003.png]]
在这个示例中，config_lowPowerStandbySupported 被overlay三次
![[Pasted image 20250313161106.png]]
再根据相关信息就可以看出是哪个apk改到了
