---
title: 记录自己上传AppStore被拒的原因
date: 2016-11-09 11:56:17
categories: iOS开发
tags: [iOS,审核被拒]
keywords: 审核
---

记录一下自己上架应用被拒的一些问题。
### 问题一

>Performance - 2.1
We were unable to review your app as it crashed on launch. We have attached detailed crash logs to help troubleshoot this issue.**Next Steps**Please revise your app and test it on a device while connected to an IPv6 network (all apps must support IPv6) to ensure it will launch without crashing.**Resources**For additional information about supporting IPv6 Networks, please refer to [Supporting IPv6 DNS64/NAT64 Networks](https://developer.apple.com/library/mac/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/UnderstandingandPreparingfortheIPv6Transition/UnderstandingandPreparingfortheIPv6Transition.html#//apple_ref/doc/uid/TP40010220-CH213-SW1) and [Supporting IPv6-only Networks](https://developer.apple.com/support/ipv6/).For a networking overview, please see [About Networking](https://developer.apple.com/library/mac/documentation/NetworkingInternetWeb/Conceptual/NetworkingOverview/Introduction/Introduction.html).For information on how to symbolicate and read a crash log, please see [Tech Note TN2151 Understanding and Analyzing Application Crash Reports](https://developer.apple.com/library/content/technotes/tn2151/_index.html).If you have difficulty reproducing this issue, please try testing the workflow described in [Testing Workflow with Xcode's Archive feature](https://developer.apple.com/library/ios/qa/qa1764/).If you have code-level questions after utilizing the above resources, you may wish to consult with [Apple Developer Technical Support](https://developer.apple.com/support/technical/submit/). When the DTS engineer follows up with you, please be ready to provide:- complete details of your rejection issue(s)- screenshots- steps to reproduce the issue(s)- [symbolicated crash logs](https://developer.apple.com/library/content/technotes/tn2151/_index.html) - if your issue results in a crash log

bb了一大堆说就是用你的应用挂了还扯到supporting IPv6，用的最新的网络请求库，网络地址用的是域名，测试没什么问题，即使没数据也不会挂了啊，查了半天竟然是数据库挂了，数据库挂了，数据库挂了，我的天
[专门解决苹果审核问题的仓库.有ipv6 和后台定位被拒绝的终极解决方案，还有其他的](https://github.com/wg689/Solve-App-Store-Review-Problem)
### 问题二
> -5  . 1.1 LEGAL: PRIVACY - DATA COLLECTION AND STORAGE
Legal -5.1.1
Your app uses the HealthKit or CareKit frameworks but does not include the required privacy policy.**Next Steps**Please update your app metadata to include a privacy policy URL and ensure that the URL you provide directs the user to your privacy policy.

应用中用到了HealthKit框架，有些必要的描述和隐私协议都没具体的写清楚。
### 问题三
>Design - 4.2.1
Your app uses the HealthKit or CareKit APIs but does not indicate integration with the Health app in your Application Description and clearly identify the HealthKit and CareKit functionality in your app's user interface.**Next Steps**Please revise your app description to specify that your app integrates with the Health app.

描述都已经写的很明确了，审核的人都不看的，直接把之前的问题抛出来，回复审核的人，内容都已经写好了，就可以发布了
### 问题四
>Design Preamble
Your app includes an update button or alerts the user to update the app. To avoid user confusion, app version updates must utilize the iOS built-in update mechanism. We've attached screenshot(s) for your reference.**Next Steps**Please remove the update feature from your app. To distribute a new version of your app, upload the new app binary version into the same iTunes Connect record you created for the app's previous version. Updated versions keep the same Apple ID, iTunes Connect ID (SKU), and bundle ID as the original version, and are available free to customers who purchased a previous version. 

说是内部包括一个更新按钮或提醒用户更新应用程序，呵呵哒，里面只是一个记录版本号，也没