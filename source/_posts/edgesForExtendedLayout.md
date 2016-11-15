---
title: edgesForExtendedLayout
date: 2016-11-11 16:49:31
categories:  iOS开发	
tags: iOS
keywords: edgesForExtendedLayout
---

edgesForExtendedLayout是一个类型为UIExtendedEdge的属性，指定边缘要延伸的方向。
因为iOS7鼓励全屏布局，它的默认值很自然地是UIRectEdgeAll，四周边缘均延伸，就是说，如果即使视图中上有navigationBar，下有tabBar，那么视图仍会延伸覆盖到四周的区域。

```
self.edgesForExtendedLayout = UIRectEdgeNone;
```

来解决UINavigationBar透明的问题。设置了UIRectEdgeNone之后，你嵌在UIViewController里面的UITableView和UIScrollView就不会穿过UINavigationBar了，同时UIView的控件也回复到了iOS6时代。

不过这个对于Status Bar在iOS7上面的变化是无效的，正确的说应该是部分无效。在存在Navigation部分或者Tabbar部分的时候，上面的代码可以使得Status Bar也不载透明（有待验证，毕竟UIViewController的可用空间被控制了），但是在没有这两个部分的时候，Status Bar依旧是会记入UIView范围的。