---
title: iOS下JS与OC互相调用MessageHandler
date: 2016-11-10 11:21:59
categories: iOS开发
tags: [MessageHandler,JS与OC交互]
keywords: 交互
---
使用WKWebView的时候，如果想要实现JS调用OC方法，除了拦截URL之外，还有一种简单的方式。那就是利用WKWebView的新特性MessageHandler来实现JS调用原生方法。

## MessageHandler 是什么？

WKWebView 初始化时，有一个参数叫configuration，它是WKWebViewConfiguration类型的参数，而WKWebViewConfiguration有一个属性叫userContentController，它又是WKUserContentController类型的参数。WKUserContentController对象有一个方法- addScriptMessageHandler:name:，我把这个功能简称为MessageHandler。

 addScriptMessageHandler:name:有两个参数，第一个参数是userContentController的代理对象，第二个参数是JS里发送postMessage的对象。 
所以要使用MessageHandler功能，就必须要实现WKScriptMessageHandler协议。 
我们在该API的描述里可以看到在JS中的使用方法：
 
 ```
 window.webkit.messageHandlers.<name>.postMessage(<messageBody>)
//其中<name>，就是上面方法里的第二个参数`name`。
//例如我们调用API的时候第二个参数填@"Share"，那么在JS里就是:
//window.webkit.messageHandlers.Share.postMessage(<messageBody>)
//<messageBody>是一个键值对，键是body，值可以有多种类型的参数。
// 在`WKScriptMessageHandler`协议中，我们可以看到mssage是`WKScriptMessage`类型，有一个属性叫body。
// 而注释里写明了body 的类型：
Allowed types are NSNumber, NSString, NSDate, NSArray, NSDictionary, and NSNull.
 ```

## 怎么使用MessageHandler？

### 创建WKWebViewConfiguration对象，配置各个API对应的MessageHandler。

```
WKUserContentController对象可以添加多个scriptMessageHandler。
```

看了示例代码，会很容易理解。示例代码如下：

 ```
    WKWebViewConfiguration *configuration = [[WKWebViewConfiguration alloc] init];
    configuration.userContentController = [WKUserContentController new];

    [configuration.userContentController addScriptMessageHandler:self name:@"ScanAction"];
    [configuration.userContentController addScriptMessageHandler:self name:@"Location"];
    [configuration.userContentController addScriptMessageHandler:self name:@"Share"];
    [configuration.userContentController addScriptMessageHandler:self name:@"Color"];
    [configuration.userContentController addScriptMessageHandler:self name:@"Pay"];
    [configuration.userContentController addScriptMessageHandler:self name:@"Shake"];
    [configuration.userContentController addScriptMessageHandler:self name:@"GoBack"];
    [configuration.userContentController addScriptMessageHandler:self name:@"PlaySound"];

    WKPreferences *preferences = [WKPreferences new];
    preferences.javaScriptCanOpenWindowsAutomatically = YES;
    preferences.minimumFontSize = 40.0;
    configuration.preferences = preferences;
 ```

    
### 创建WKWebView

这里没什么好说的，直接看示例代码吧：

  ```
  self.webView = [[WKWebView alloc] initWithFrame:self.view.frame configuration:configuration];

    NSString *urlStr = [[NSBundle mainBundle] pathForResource:@"index.html" ofType:nil];
    NSURL *fileURL = [NSURL fileURLWithPath:urlStr];
    [self.webView loadFileURL:fileURL allowingReadAccessToURL:fileURL];

    self.webView.navigationDelegate = self;
    self.webView.UIDelegate = self;
    [self.view addSubview:self.webView];
  ```

### 实现协议方法

我这里实现了两个协议<WKUIDelegate,WKScriptMessageHandler>，WKUIDelegate是因为我在JS中弹出了alert 。WKScriptMessageHandler是因为我们要处理JS调用OC方法的请求。 
先看实现协议方法的示例代码：

 ```
#pragma mark - WKScriptMessageHandler
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message
{
//    message.body  --  Allowed types are NSNumber, NSString, NSDate, NSArray,NSDictionary, and NSNull.
    if ([message.name isEqualToString:@"ScanAction"]) {
        NSLog(@"扫一扫");
    } else if ([message.name isEqualToString:@"Location"]) {
        [self getLocation];
    } else if ([message.name isEqualToString:@"Share"]) {
        [self shareWithParams:message.body];
    } else if ([message.name isEqualToString:@"Color"]) {
        [self changeBGColor:message.body];
    } else if ([message.name isEqualToString:@"Pay"]) {
        [self payWithParams:message.body];
    } else if ([message.name isEqualToString:@"Shake"]) {
        [self shakeAction];
    } else if ([message.name isEqualToString:@"GoBack"]) {
        [self goBack];
    } else if ([message.name isEqualToString:@"PlaySound"]) {
        [self playSound:message.body];
    }
}
 ```

WKScriptMessage有两个关键属性name 和 body。 
因为我们给每一个OC 方法取了一个name，那么我们就可以根据name 来区分执行不同的方法。body 中存着JS 要给OC 传的参数。 
关于参数body 的解析，我就举一个body中放字典的例子，其他的稍后可以看demo。 
解析JS 调用OC 实现分享的参数：

```
- (void)shareWithParams:(NSDictionary *)tempDic
{
    if (![tempDic isKindOfClass:[NSDictionary class]]) {
        return;
    }

    NSString *title = [tempDic objectForKey:@"title"];
    NSString *content = [tempDic objectForKey:@"content"];
    NSString *url = [tempDic objectForKey:@"url"];
    // 在这里执行分享的操作

    // 将分享结果返回给js
    NSString *jsStr = [NSString stringWithFormat:@"shareResult('%@','%@','%@')",title,content,url];
    [self.webView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        NSLog(@"%@----%@",result, error);
    }];
}
```

message.boby 就是JS 里传过来的参数。我们不同的方法先做一下容错性判断。然后正常取值就可以了。

### 处理HTML中JS调用

HMTL的源码跟之前的HTML内容差不多，只有JS的调用部分改变了。

```
// 传null
function scanClick() {
    window.webkit.messageHandlers.ScanAction.postMessage(null);
}
// 传字典              
function shareClick() {
    window.webkit.messageHandlers.Share.postMessage({title:'测试分享的标题',content:'测试分享的内容',url:'http://www.baidu.com'});
}
// 传字符串
function playSound() { 
    window.webkit.messageHandlers.PlaySound.postMessage('shake_sound_male.wav');
}
// 传数组
function colorClick() {
    window.webkit.messageHandlers.Color.postMessage([67,205,128,0.5]);
}
```

### OC调用JS

这里使用WKWebView 实现OC 调用JS方法跟上一篇是一样的，还是利用 
- evaluateJavaScript:completionHandler:。像下面这样使用：

```
// 将分享结果返回给js
    NSString *jsStr = [NSString stringWithFormat:@"shareResult('%@','%@','%@')",title,content,url];
    [self.webView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
        NSLog(@"%@----%@",result, error);
    }];
```

## 使用MessageHandler 的好处

1. 在JS中写起来简单，不用再用创建URL的方式那么麻烦了。
2. JS传递参数更方便。使用拦截URL的方式传递参数，只能把参数拼接在后面，如果遇到要传递的参数中有特殊字符，如&、=、？等，必须得转换，否则参数解析肯定会出错。                   
例如传递的url是这样的:        
http://www.baidu.com/share/openShare.htm?share_uuid=shdfxdfdsfsdf&name=1234556             
使用拦截URL 的JS调用方式

```
loadURL("haleyAction://shareClick?title=分享的标题&content=分享的内容&url=链接地址&imagePath=图片地址"); }
1
```

将上面的url 放入链接地址这里后，根本无法区分share_uuid是其他参数，还是url里附带的参数。 
但是使用MessageHandler 就可以避免特殊字符引起的问题。

更详细的使用步骤还是去工程中查看吧。地址：[JS_OC_MessageHandler](https://github.com/Haley-Wong/JS_OC/tree/master/JS_OC_MessageHandler)

