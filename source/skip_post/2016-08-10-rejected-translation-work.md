---
title: 使用Swift在HealthKit中进行睡眠分析
date: 2017-09-11 14:50:38
tags: [Translation]
---
Some translation work
<!--more-->
#使用Swift在HealthKit中进行睡眠分析
当前，睡眠革命正成为时尚，用户从没有比现在更加好奇，不仅想知道睡眠时间，而且通过收集长期睡眠数据从而分析出睡眠趋势。硬件发展，特别是手机的升级，为实现这一日益增长的需求提供了新的方向。

Apple提供了一种非常酷的的方法来获取用户的私人健康信息，而数据被安全的存储在内置的Health应用里。这样你不仅可以使用HealthKit开发健康类APP，你还能用获取HealthKit中的睡眠分析数据。

下面的教程里，我将对HealthKit框架做一个快速的简介，并且教会你做一个分析睡眠的简单应用。

##简介
HealthKit框架提供一个叫做HealthKit Store的存储体，它可以将数据存储到一个加密的数据库里。你可以通过HKHealthStore类访问这个数据库。iPhone和Apple Watch都各自有一个HealthKit Store。健康数据在Apple Watch和iPhone间是同步的。但是旧数据会周期性的从Apple Watch中删除以节省空间。而在iPad上，HealthKit和Health应用都是不可用的。

如果你想要开发基于健康数据的iOS和watchOS应用，那HealthKit将是非常强大的工具。它被设计用来管理各个源获取来的数据，基于用户设置可以自动合并不同源的数据，开发应用也可以直接获取各个源的原始数据，自己合并数据。它不仅可以测量身体状况，监测健康或营养数据，数据也可以用作睡眠分析。

在以下的文章里，我将教你怎样使用HealthKit框架来存储和获取iOS中的睡眠分析数据。watchOS的方法也同理。注意下面的教程根据Swift 2.0和Xcode7而写的。所以参照教程的时候请确认使用的是Xcode7(或以上）。

在继续以前，请先下载和解压[初始工程](https://github.com/appcoda/SleepAnalysis/blob/master/SleepAnalysisStarter.zip?raw=true)。我已经把用户界面和一些基本的功能写好。当你跑起来初始工程，你会看到一个计时器，当你按下开始按钮，就会开始计时。

##使用HealthKit框架

我们的应用要做的是按下_开始_ & _停止_按钮时，程序会获取和存储睡眠分析信息。使用HealthKit之前，你必须先把app bunndle的HealthKit capabilities打开。在工程页面里，将Target栏中Capabilities菜单打开。

![](http://www.appcoda.com/wp-content/uploads/2016/05/HealthKit-allow-1240x775.png)

接下来，你需要使用以下代码在ViewController类里创建一个HKHealthStore实例：

``` let healthStore = HKHealthStore()```

之后我们会用HKHealthStore实例访问HealthKit store。

正如上面说的一样，HealthKit让用户可以控制自己的健康数据。所以在对用户的睡眠数据作读写操作之前，你首先要向用户请求权限。实现这个，你要先import内置的HealthKit框架，像这样修改_viewDidLoad_方法：

```
override func viewDidLoad() {
    super.viewDidLoad()

    let typestoRead = Set([
        HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis)!
        ])

    let typestoShare = Set([
        HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis)!
        ])

    self.healthStore.requestAuthorizationToShareTypes(typestoShare, readTypes: typestoRead) { (success, error) -> Void in
        if success == false {
            NSLog(" Display not allowed")
        }
    }
}
```

这段代码会提示用户允许或者拒绝权限请求。在completion block里，你可以处理成功或者失败的情况。用户不一定会允许所有的权限请求。你必须在程序里得体的处理所产生的错误。

但是出于测试的目的，演示的app里你必须选择“允许“给予App获取你设备上的健康数据。

![](http://www.appcoda.com/wp-content/uploads/2016/05/Health-App-Permission.png)

## 写入睡眠分析数据

首先，我们怎么才能获取睡眠分析数据？Apple的文档上讲到，每个睡眠分析样本仅能有一个值。要表示用户已上床且已入睡，HealthKit需要用到时间重叠两个或多个样本。通过比较这些样本的起始和终止时间，程序可以计算出许多次要的统计数据：

* 用户进入睡眠所需的时间
* 用户上床并且真正入睡的百分比
* 用户在床上醒来的次数
* 用户上床总时间和入睡的总时间

![](http://www.appcoda.com/wp-content/uploads/2016/05/record_sleep_data.png)

简单讲，你要用一下方法来存储睡眠分析数据到HealthKit store：

1. 我们要定义两个NSDate对象分别对应起始时间和终止时间。
2. 然后使用HKCategoryTypeIdentifierSleepAnalysis创建一个HKObjectType对象。
3. 我们需要一个新的HKCategorySample类的实例。通常你要使用不同类型样本来记录不同类型睡眠数据。单独的一个样本只代表在床上或者代表已入睡的时间片段。所以我们同时创建上床样本和入睡样本时，会出现重叠的时间。
4. 最后，我们使用HKHealthStore的saveObject方法来存储对象。

编者注：更多的样本类型，你可以[HealthKit Constants Reference](https://developer.apple.com/library/ios/documentation/HealthKit/Reference/HealthKit_Constants/index.html#//apple_ref/doc/uid/TP40014710)里查到。

如果你把以上所讲转化成Swift，下面时用来存储上床和入睡时间睡眠分析数据。请将下面的方法加到ViewController类里：

```
func saveSleepAnalysis() {

    // alarmTime and endTime are NSDate objects
    if let sleepType = HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis) {

        // we create our new object we want to push in Health app
        let object = HKCategorySample(type:sleepType, value: HKCategoryValueSleepAnalysis.InBed.rawValue, startDate: self.alarmTime, endDate: self.endTime)

        // at the end, we save it
        healthStore.saveObject(object, withCompletion: { (success, error) -> Void in

            if error != nil {
                // something happened
                return
            }

            if success {
                print("My new data was saved in HealthKit")

            } else {
                // something happened again
            }

        })


        let object2 = HKCategorySample(type:sleepType, value: HKCategoryValueSleepAnalysis.Asleep.rawValue, startDate: self.alarmTime, endDate: self.endTime)

        healthStore.saveObject(object2, withCompletion: { (success, error) -> Void in
            if error != nil {
                // something happened
                return
            }

            if success {
                print("My new data (2) was saved in HealthKit")
            } else {
                // something happened again
            }

        })

    }

}
```

当我们想要存储睡眠分析数据到HealthKit时可以调用这个方法。

##读取睡眠分析数据

要读取睡眠分析数据，我们需要创建一条查询。你先要定义一个类型为HKCategoryTypeIdentifierSleepAnalysis的HKObjectType对象。你可能还要用predicate来过滤取回的数据，你可以用表示的startDate和endDate的NSDate对象来对应要查询的时间范围。你同时需要创建一个sortDescriptor来给查询到的纪录排序以得到想到的结果。

获取睡眠分析数据的代码应该如下：

```
func retrieveSleepAnalysis() {

    // first, we define the object type we want
    if let sleepType = HKObjectType.categoryTypeForIdentifier(HKCategoryTypeIdentifierSleepAnalysis) {

        // Use a sortDescriptor to get the recent data first
        let sortDescriptor = NSSortDescriptor(key: HKSampleSortIdentifierEndDate, ascending: false)

        // we create our query with a block completion to execute
        let query = HKSampleQuery(sampleType: sleepType, predicate: nil, limit: 30, sortDescriptors: [sortDescriptor]) { (query, tmpResult, error) -> Void in

            if error != nil {

                // something happened
                return

            }

            if let result = tmpResult {

                // do something with my data
                for item in result {
                    if let sample = item as? HKCategorySample {
                        let value = (sample.value == HKCategoryValueSleepAnalysis.InBed.rawValue) ? "InBed" : "Asleep"
                        print("Healthkit sleep: \(sample.startDate) \(sample.endDate) - value: \(value)")
                    }
                }
            }
        }

        // finally, we execute our query
        healthStore.executeQuery(query)
    }
}
```

这段代码从HealthKit查询到睡眠分析数据，然后按倒序排序。然后每条数据都打印出startDate和endDate，还有样本类型，i.e.在床上或者入睡。我上限设置为30，取到是最后30条纪录的样本。你当然也可以使用predicate方法来自定义初始和终止的日期。

##App测试

在演示的程序里，我使用了NSTimer来计算按了开始按钮后经过的时间。开始和停止按钮按下时都创建一个NSDate对象，同时存储下这段时间里的睡眠分析数据。在终止按钮的IBAction方法里，你可以调用saveSleepAnalysis()来存储数据，调用retrieveSleepAnalysis()方法来获取数据。

```
@IBAction func stop(sender: AnyObject) {
    endTime = NSDate()
    saveSleepAnalysis()
    retrieveSleepAnalysis()
    timer.invalidate()
}
```

程序里， 存储在床上和入睡的数值时，你可能需要修改NSDate对象以获得由相关性的这两种类型的初始时间和结束时间（很可能不同）

当你完成以上的修改后，运行起来演示的程序再开启计时器。然后让他运行一段时间再按下停止按钮。然后，打开Health应用，你就能看到睡眠数据出现了。

![](http://www.appcoda.com/wp-content/uploads/2016/06/sleep-analysis-test-1240x878.png)

##开发Health应用的一些建议

设计HealthKit是用来提供给开发者一个分享和获取用户数据的平台，它避免了重复或者冲突的数据的出现。Apple的审核指南里非常明确的指出使用HealthKit并且请求读／写权限的app，如果没有清楚的演示用处的很有可能会导致app被拒。

存储虚假或错误数据到Health应用的App也同样会被拒。这说明，你不能天真的以为可以像此教程里分析睡眠一样使用自己的算法来计算出不同的健康数据。你应该尝试读取内置的传感器数据来和调整任意参数以避免计算出错误的数据。

完整的Xcode工程，你可以在[这里](https://github.com/appcoda/SleepAnalysis)取得。
