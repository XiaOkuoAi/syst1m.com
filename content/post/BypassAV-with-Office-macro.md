---
title: "BypassAV With Office Macro"
date: 2021-01-16T12:24:45+08:00
draft: false
tags: [Fishing]
categories: [RedTeam]
comment: true
---
BypassAV With Office Macro
<!--more-->

# office宏

**钓鱼攻击是攻防演练中常见的打点手段，那么安全防护当然也少不了，那么相对而言攻击手法就有必要提高，这里只说明一种简单实用的方法做参考**

- 首先cs生成宏代码

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105173712.png)

- 粘贴进去执行文档，不用说，大哥直接杀了

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105173811.png)

- 复制CS生成的宏代码

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105184902.png)


- 把恶意代码删除掉，替换成弹出框

```
Sub Auto_Open()
    MsgBox("欢迎您~") 
Sub AutoOpen()
    Auto_Open
End Sub
Sub Workbook_Open()
    Auto_Open
End Sub

```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105185245.png)

啊这～，三个空函数都给我报毒了，我太难了

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105185400.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105185314.png)

经过我一番瞎JER测试，最后把最后的`Workbook_Open`给删除掉了，再次尝试，MMP，我的弹窗出来了，欢迎您！

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105185751.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105185840.png)

接着来一个一个看，把原来的CS复制进去，然后我删掉最后一个函数，只留下前两个函数然后粘贴进去保存

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105192113.png)

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105192216.png)

然后360查杀一下，WTF ?咋没查杀出来，我点击一下宏文档，竟然他妈的上线了。啊这~

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105192431.png)

- 在线查杀一下，很多报毒的，不过，360没报毒，就达到目的了

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20210105193851.png)