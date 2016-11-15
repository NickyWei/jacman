---
title: UIButton设置图片和文字位置
date: 2016-11-08 19:36:09
categories: iOS开发	
tags: [iOS,UIButton]
keywords: Button
---
在开发的过程中经常会遇到需要在button中放置图片和文字，比如将图片放置在button左边，文字放置在右边。实现的方法也是很多的。

因为UIButton也是继承自UIView，因此可以像其它的view一样添加subView，

``` ruby
//创建button
UIButton *button = [UIButton buttonWithType:UIButtonTypeRoundedRect];
// 创建imageview
UIImage *image = [UIImage imageNamed:@"yourImage.png"];
UIImageView *imageView = [[UIImageView alloc] initWithFrame:CGRectMake(/*frame*/)];
[imageView setImage:image];
// 创建label
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(/*frame*/)];
[label setText:@"Your title"];
// 添加到button中
[button addSubview:label];
[button addSubview:imageView];
```
还可以通过继承UIView进行自定制，这种方法的好处是简单明了，但是其实在UIButton中已经包含了UIImageView，我们不需要在自己添加该imageView的。

UIButton中有imageEdgeInsets和titleEdgeInsets两个属性可以控制image和label的位置
我们可以给UIButton添加扩展，在扩展中对imageEdgeInsets和titleEdgeInsets进行重新的设置，这样就可以实现图片和文字位置切换。

```
typedef struct UIEdgeInsets {
    CGFloat top, left, bottom, right;  // specify amount to inset (positive) for each of the edges. values can be negative to 'outset'
} UIEdgeInsets;
```
从上边我们可以看出来偏移量是上，左，下，右
这样就可以通过计算进行偏移量的计算
>比如图片在右文字在左						
>也就是最终我们希望图片上下位置不变top = 0, bottom = 0;向右位移一个label的宽度
同时标题上下位置不变top = 0, bottom = 0;向左位移一个图片的宽度
即 imageEdgeInsets = UIEdgeInsetsMake(0,0 + labelWidth,0,0 - labelWidth); 图片相对初始状态左边距加了labelWidth，右边减labelWidth，以及titleEdgeInsets ＝ UIEdgeInsetsMake(0,0 - imageViewWidth,0, 0 + imageViewWidth);

[Demo UIButton类别](https://github.com/NickyWei/UIButton-CZEdgeInsets/tree/master)