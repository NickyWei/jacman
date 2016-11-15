---
title: iOS Healthkit和CMPedometer区别
date: 2016-11-09 13:43:12
categories: iOS开发	
tags: [iOS,Health,计步器]
keywords:
---

这几天在弄计步功能, 找了好久看了几篇零散的文章, 我自己做了总结, 我忘了我看文章的地址了, 没法附上转载连接了

CMPedometer 可以访问用户活动(可以访问计步数据) 适用ios8以上的系统 ios可看最下面说明

Healthkit 是ios系统中的健康应用 需要应用授权来访问健康数据(比如计步)

这两个有什么区别呢?
其实 Healthkit 也是 使用 CMPedometer 读取用户的步数和走的里程,当然计算的方法是在手机内部计算的, 它是利用 iphone5s 以上的m处理器来获取数据计算了,所以5s以下的设备是不支持 CMPedometer 手机计步的,当然有大牛可以利用重力传感器和加速度传感器一样可以写出自己的计步器软件,我是个菜鸟只能利用系统的拉

Healthkit计步有个缺点, 计步不准,有人会问了:这个 Healthkit 和我下面的 demo 不都是用 CMPedometer 来获取计步数的,为什么 Healthkit 不准呢?
答: 那是因为Healthkit记录的时区问题, ios获取时间的时区按中国的时区少了8个小时, 举个例子, 如果我要记录 5月5日的数据应该是 xxxx-05-05 00:00:00 至xxxx-05-05 23:59:59 这个区间的步数情况, 而 Healthkit 记录的时区却是 xxxx-05-04 16:00:00 至 xxxx-05-05 15:59:59 ,所以Healthkit记录的信息是不准的, 如果要是自己开发计步软件呢,我们就要自己写把, 不要获取 Healthkit 的信息啦

以下demo是用 swift语言写的

```
定义个全局变量 如果不是全局变量会报错
let pedonmeter:CMPedometer = CMPedometer()

在方法里面写, 两种方法用哪个都可以,不明白的地方可以看文档
//判断该设备是否支持计步功能
if CMPedometer.isStepCountingAvailable(){
//开始时间
let startTime = getStartTime()
//结束时间
let endTime = getEndTime()
//第一种
//获取一个时间范围内的数据最大7天 参数 开始时间,结束时间, 一个闭包
pedonmeter.queryPedometerDataFromDate(startTime, toDate:endTime, withHandler: { (pedometerData:CMPedometerData?, error:NSError?) -> Void in
if error != nil{
print("error:\(error)")
}
else{
print("开始时间:\(startTime)")
print("结束时间:\(endTime)")
print("步数===\(pedometerData!.numberOfSteps)")
print("距离===\(pedometerData!.distance)")
}
})
}

//第二种
//获取指定开始时间到当前时间的数据 参数 开始时间, 一个闭包
pedonmeter.startPedometerUpdatesFromDate(startTime, withHandler: { (pedometerData:CMPedometerData?, error:NSError?) -> Void in
if error != nil{
print("error:\(error)")
}
else{
print("开始时间:\(startTime)")
print("结束时间:\(endTime)")
print("步数===\(pedometerData!.numberOfSteps)")
print("距离===\(pedometerData!.distance)")
}
})
```

下面是获取时间的方法,我是获取当天的,你们也可以自己改


```
获取当前时区的时间
*/
func getEndTime() -> NSDate
{
//转换成本地时区
let date = NSDate()
let zone = NSTimeZone.systemTimeZone()
let interval = zone.secondsFromGMTForDate(date)
let nowDate = date.dateByAddingTimeInterval(Double(interval))
return nowDate
}
/**
获取开始时间 当天0时0分0秒
*/
func getStartTime() -> NSDate
{
let datef = NSDateFormatter()
datef.dateFormat = "yyyy-MM-dd"
let stringdate = datef.stringFromDate(getEndTime())
print("当天日期:\(stringdate)")
let tdate = datef.dateFromString(stringdate)
//获取本地时区的当天0时0分0秒
let zone = NSTimeZone.systemTimeZone()
let interval = zone.secondsFromGMTForDate(tdate!)
let nowday = tdate!.dateByAddingTimeInterval(Double(interval))
return nowday
}
```

如果你像支持ios7的话那不就能使用CMPedometer
CMPedometer 的前身是 CMStepCounter
大家可以参考[http://www.aichengxu.com/view/27076](http://www.aichengxu.com/view/27076)