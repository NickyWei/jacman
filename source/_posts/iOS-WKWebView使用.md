---
title: iOS WKWebView使用
date: 2016-11-10 10:52:51
categories: iOS开发
tags: [iOS,WKWebView]
keywords: WKWebView
---

WKWebView 是 iOS 8 引入的一个 WebKit API，替换 iOS 的 UIWebView 和 OSX 的 WebView，WKWebView 使用 Nitro JavaScript 引擎，运行 JavaScript 和 Safari 一样快。WKWebView 和 UIWebView 使用上有很多区别，这篇主要介绍代理、标题、进度条展示的区别。

### 代理
 UIWebView 是 delegate: UIWebViewDelegate，WKWebView 使用以下两个代理
 
1. navigationDelegate: WKNavigationDelegate
2. UIDelegate: WKUIDelegate   

#### WKNavigationDelegate

  该代理提供的方法，可以用来追踪加载过程（页面开始加载、加载完成、加载失败）、决定是否执行跳转。
  
UIWebView | WKWebView 
----|------
– (NSString )stringByEvaluatingJavaScriptFromString:(NSString )script; | – (void)evaluateJavaScript:(NSString )JavaScriptString completionHandler:(void (^ nullable)(nullable id, NSError __nullable error))completionHandler;  
– (void)webViewDidStartLoad:(UIWebView *)webView | – (void)webView:(WKWebView )webView didCommitNavigation:(null_unspecified WKNavigation )navigation  
– (void)webViewDidFinishLoad:(UIWebView *)webView | – (void)webView:(WKWebView )webView didFinishNavigation:(null_unspecified WKNavigation )navigation  
– (void)webView:(UIWebView )webView didFailLoadWithError:(NSError )error | – (void)webView:(WKWebView )webView didFailNavigation:(null_unspecified WKNavigation )navigation withError:(NSError *)error
– (BOOL)webView:(UIWebView )webView shouldStartLoadWithRequest:(NSURLRequest )request navigationType:(UIWebViewNavigationType)navigationType | – (void)webView:(WKWebView )webView decidePolicyForNavigationAction:(WKNavigationAction )navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler

比如，如果要拦截网页请求，UIWebView 使用以下代码：

```
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
     NSURL *url = request.URL;
    NSString *host = [url host];
    if ([url.scheme isEqualToString:@"inapp"]) {

        /*
         * 做一些处理
         */

        return NO;
    }

    return YES;
}
```

在 WKWebView 中则用以下代码：

```
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {

    NSURL *url = navigationAction.request.URL;
    NSString *host = [url host];
    if ([url.scheme isEqualToString:@"inapp"]) {

        /*
         * 做一些处理
         */

        decisionHandler(WKNavigationActionPolicyCancel);
    } else {

        decisionHandler(WKNavigationActionPolicyAllow);
    }
}
```

### 获取 title

在 UIWebView 中

```
self.title = [webView stringByEvaluatingJavaScriptFromString:@"document.title"];
```

而 WKWebView 有 title 属性可直接获取网页标题，可通过 KVO 方式获取网页标题的动态变化。

```
self.title = webView.title;
```

### estimatedProgress 进度条

WKWebView 增加了 estimatedProgress，获取网页加载进度，轻松显示进度条。可通过 KVO 的方式获取 estimatedProgress 的变化。首先在 viewDidLoad 中添加观察者

```
[self.webView addObserver:self forKeyPath:NSStringFromSelector(@selector(estimatedProgress)) options:NSKeyValueObservingOptionNew context:NULL];
[self.webView addObserver:self forKeyPath:NSStringFromSelector(@selector(title)) options:NSKeyValueObservingOptionNew context:NULL];
```

KVO 的实现：

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    if ([keyPath isEqualToString:NSStringFromSelector(@selector(estimatedProgress))] && object == self.webView) {
        NSLog(@"%f", self.webView.estimatedProgress);
        // estimatedProgress is a value from 0.0 to 1.0
        // Update your UI here accordingly

        [self.progressView setAlpha:1.0f];
        [self.progressView setProgress:self.webView.estimatedProgress animated:YES];

        if(self.webView.estimatedProgress >= 1.0f) {
            [UIView animateWithDuration:0.3 delay:0.3 options:UIViewAnimationOptionCurveEaseOut animations:^{
                [self.progressView setAlpha:0.0f];
            } completion:^(BOOL finished) {
                [self.progressView setProgress:0.0f animated:NO];
            }];
        }

    } else if ([keyPath isEqualToString:@"title"]) {
        if (object == self.webView) {
            self.title = self.webView.title;
        } else {
            [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
        }
    } else {

        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
}
```

最后，在 dealloc 中移除观察者

```
- (void)dealloc {
    if ([self isViewLoaded]) {
        [self.webView removeObserver:self forKeyPath:NSStringFromSelector(@selector(estimatedProgress))];
        [self.webView removeObserver:self forKeyPath:NSStringFromSelector(@selector(title))];
    }

    [self.webView setNavigationDelegate:nil];
}
```

### Message Handlers 用于和网页的JS交互


[Message Handlers详解](http://weichengzong.cf/2016/11/10/iOS%E4%B8%8BJS%E4%B8%8EOC%E4%BA%92%E7%9B%B8%E8%B0%83%E7%94%A8MessageHandler/)
