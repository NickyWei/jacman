---
title: iOS LLDB调试
date: 2016-11-09 11:22:29
categories: iOS开发
tags: [调试,LLDB]
keywords: 调试
---

LLDB是XCode内置调试工具
![1.png](http://upload-images.jianshu.io/upload_images/2071097-580248ecfbc1b2e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们加了断点，然后在运行到断点处就停了下来，接下来我们看到lldb这里了吗？我们可以通过lldb所提供的命令来操作。

#### 常用p、po、call命令

从官方的描述来看，p、print、call是一样的，但是po就不太一样了，输入一样但是输出不一样。po的输出的是具体对象的内容。
#### lldb声明变量
我们可以使用e命令定义变量，然后在调试中使用。看如下的例子：

```
(lldb) e NSString *$str = @"http://www.huangyibiao.com"
(lldb) po $str
http://www.huangyibiao.com
 
(lldb) e int $count = 10
(lldb) p $count
(int) $count = 10
(lldb) e NSArray *itemArray = @[@"Test", @"Demo", @"huangyibiao"]
(lldb) po $count
10
 
```

我们使用e声明了$str变量，然后下面就可以使用了。我们看到通过p命令打印出来的都是$开头的变量了吗？我们在声明和使用时也需要加上$符号，与PHP一样！

在调试时，有时候想临时计算一下某个值来比较时，就可以通过这种方式来实现了，再也不用到源代码处添加上声明实现然后添加一句打印了，是否便利了很多？
#####调用变量的API
当我们在断点处，定义了blogName变量了，因此我们可以通过调试命令来调用

```
(lldb) po [blogName uppercaseString]
标哥的技术博客
 
(lldb) po [blogName substringFromIndex:2]
的技术博客
```

#### 强转返回值类型
当我们调用API返回值类型不指定时，有时候所打印出来的东西是我们看不懂的，比如下面的获取结果应该是“标”字，但是不同类型打印结果却不一样：

```
(lldb) po [blogName characterAtIndex:0]
26631
 
(lldb) po (unsigned int)[blogName characterAtIndex:0]
26631
 
(lldb) po (char)[blogName characterAtIndex:0]
'\a'
 
(lldb) po (NSString *)[blogName characterAtIndex:0]
0x0000000000006807
 
(lldb) po (unichar)[blogName characterAtIndex:0]
U+6807 u'标'
```

#### 设置断点触发条件
看下图，笔者是怎么设置触发条件的：

![2.png](http://upload-images.jianshu.io/upload_images/2071097-2b1f90f0b6d78df8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们在NSLog这一行，设置了条件，只有当条件满中时，才会进入断点，不过这里并没有让它进入断点，而条件满足时就发出声音并打印提示语。

这种应用场景主要是在循环遍历数据时，想要断点跟踪就只能通过这种方式了，除非添加NSLog打印，但是这种需要手动添加代码，在调试时才想到要添加一些打印语句，这时候又得重新运行，这太慢了。如果懂得如何设置断点条件，那么就能满足我们的需求了，直接可以设置条件。
##### 常用打印视图层次结构
当我们想要知道某个视图的结构时，可以通过调用recursiveDescription方法来打印出来，那么其结构就一目了然了：

```
(lldb) po [self.view recursiveDescription]
<UIView: 0x7fdd1052af10; frame = (0 0; 320 568); autoresize = W+H; layer = <CALayer: 0x7fdd1052b290>>
   | <UIButton: 0x7fdd10529070; frame = (66 183; 188 40); opaque = NO; autoresize = RM+BM; layer = <CALayer: 0x7fdd1051bff0>>
   |    | <UIButtonLabel: 0x7fdd104162f0; frame = (41.5 11; 105 18); text = '标哥的技术博客'; alpha = 0.2; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x7fdd10412590>>
   |    |    | <_UILabelContentLayer: 0x7fdd12804f30> (layer)
   | <_UILayoutGuide: 0x7fdd1052b300; frame = (0 0; 0 20); hidden = YES; layer = <CALayer: 0x7fdd1052b710>>
   | <_UILayoutGuide: 0x7fdd1052c070; frame = (0 568; 0 0); hidden = YES; layer = <CALayer: 0x7fdd1052c200>>
   
```
 
 
#### 临时刷新界面UI
最开始按钮的背景颜色是blueColor，现在我们要在调试过程中修改其背景色为红色，并刷新界面。执行下面的命令行，App界面的按钮背景颜色是：

```
(lldb) e ((UIButton *)sender).backgroundColor = [UIColor redColor]
(UICachedDeviceRGBColor *) $41 = 0x00007fdd10715b00
(lldb) e (void)[CATransaction flush]
```

#### 修改变量值
本小节通过使用expr在调试过程中修改变量的值，感谢我的大徒弟在看完之后根据自己的经验总结出这一条技巧，现在也分享出来给大家~

使用很简单，就是这样的规则：

```
expr variable = newValue
```

可以输入help查看！
>感谢作者在[标哥的技术博客](http://www.huangyibiao.com/)
