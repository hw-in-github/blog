---
title: 'Swift中strong,weak,unowned关键字'
date: 2017-11-05 20:53:30
tags:
---
* `strong`:当你声明一个属性时,它默认就是强引用
* `weak`:弱引用对象的引用计数不会+1, 必须为`可选类型变量`
在声明弱引用对象是必须用`var`关键字, 不能用`let`.
因为弱引用变量在没有被强引用的条件下会变为nil, 而let常量在运行的时候不能被改变.

```swift
class XDTest {
    //会报错
    weak let tentacle = Tentacle() //let is a constant! All weak variables MUST be mutable.
}
```
* `unowned`:相当于`__unsafe_unretained`, 不安全. 必须为`非可选类型`.
`unowned`引用是`non-zeroing`(非零的), 在ARC销毁内存后，不会被赋为nil, 这表示着当一个对象被销毁时, 它指引的对象不会清零. 也就是说使用unowned引用在某些情况下可能导致`dangling pointers`(野指针). 所以在访问无主引用的时候，要确保其引用正确，不然会引起内存崩溃.

* `隐式解析可选类型` ：语法是在变量后面加上感叹号(例如var name:String!). 在初始的时候可以为nil, 但是第一次赋值以后便会一直有值. 使用该类型只需要正常调用, 不需要像可选类型那样解包.

### 避免在闭包中循环引用

在闭包中, 要拿到对象本身的属性, 必须要用到self关键字.
导致block对对象进行了强引用, 而对象本身对block也是强引用, 这样就形成了循环引用. (`Self <-> Block`)

1. 解决办法和OC中一样, 将强引用self变为弱引用self.
OC中解决办法是`__weak SelfClass *weakSelf = self;``
在Swift中类似的解决办法是(解决方式一)`weak var weakSelf = self`
2. 使用捕获列表(在其定义的上下文中捕获常量或变量, 即使定义这些常量和变量的原域已经不存在, 闭包仍然可以在闭包函数体内引用和修改这些值.)
苹果官方语言指南要求如果闭包和其捕获的对象相互引用, 应该使用unowned, 这样可以保证他们会同时被销毁. 大概是为了避免对象被释放后维护weak引用空指针的开销.

*格式*:在闭包前面加上`[unowned 想要捕获的变量]`

官方文档中的例子:
有参数和返回值的block

```swift
lazy var someClosure: (Int, String) -> String = {
    [unowned self, weak delegate = self.delegate!] (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}
```
无参数和返回值的block

```swift
lazy var someClosure: Void -> String = {
    [unowned self, weak delegate = self.delegate!] in
    // closure body goes here
}
```
例子:

```swift
import UIKit

class ViewController: UIViewController {
    var finishedCallBack: ( (dataString: String) -> () )?
    override func viewDidLoad() {
        super.viewDidLoad()

        //解决方式三: [unowned self]  跟 _unsafe_unretained 类似  
        loadData { [unowned self] (dataString) -> () in
            print("\(dataString) \(self.view)")
        }  
    }

    func method2() {
        //解决方式二:  在swift中 有特殊的写法  [weak self]
        loadData { [weak self] (dataString) -> () in

            //以后在闭包中中 使用self 都是若引用的
            print("\(dataString) \(self?.view)")
        }
    }

    func method1() {
        // 解决方式一: weak , OC中类似方法__weak
        weak var weakSelf = self
        loadData { (dataString) -> () in
            print("\(dataString) \(weakSelf?.view)")
        }
    }


    func loadData(finished: (dataString: String) -> ()) {

        // 记录闭包
        self.finishedCallBack = finished
        //加载数据
        dispatch_async(dispatch_get_global_queue(0, 0)) { () -> Void in

            print("执行耗时操作")

            dispatch_async(dispatch_get_main_queue(), { () -> Void in
                //执行回调
                self.working()
            })
        }
    }


    func working() {
        self.finishedCallBack?(dataString: "<html>")
    }

    //swift dealloc
    //析构函数
    deinit {
        print("销毁")
    }
}
```
