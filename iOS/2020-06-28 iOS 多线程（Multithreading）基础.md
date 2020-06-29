# 2020-06-28 iOS 多线程（Multithreading）

## Queues（队列）

在 iOS 开发里 多线程 主要是关于 ‘queues（队列）’。队列里为一个一个方法（Fuctions，通常是 closures ），系统从队列中依次取出方法在关联线程（associated thread）上执行。

队列分为两种：***serial***（***线性队列***，一次执行一个 closure），***concurrent***（***并行队列***，可使用多个线程）。

## Main Queues（主队列）

UI 运行在一个的线性队列上 - 即主队列。所有的 UI 活动必须发生在主队列，并且只能发生于主队列。

## Global Queues（全局队列）

除 UI 活动外其他事物，通常使用一个共享的（shared）、全局的（global）、并行的（concurrent）队列。

## 将一块代码放到队列中

有两种方法将 closure 放到队列中：

``` swift
/**
方式1：异步 asynchronous
@discuss 多数时候使用 async
@discuss 
*/
queue.async { ... }
 
/**
方式2：同步
@warning 永远不要在 主线程 上使用 sync 。-> 因为我们不想阻塞主队列。
@discuss 你可能会在 非主队列 上使用 sync ，以等待主队列执行完某些任务。
*/
queue.sync { ... }
```

## 获取一个非全局的队列

极少数情况可能需要一个主队列或全局队列之外的队列，有以下两种方式：

```swift
/// 方式1：创建线性队列（仅当你需要处理多个连续的独立活动时）
let serialQueue = DispatchQueue(label: "MySerialQ")
/// 方式2：创建并行队列（相比于全局队列，极少使用）
let concurrentQueue = DispatchQueue(label: "MyConcurreentQ", attributes: .concurrent)
```

## 其他  API 使用多线程

**OperationQueue** 与 **Operation** 通常用于更复杂的多线程任务，相比之下 **DispatchQueue** API 更常用，嵌套式的代码也更易读。

## 多线程使用示例及执行顺序

```swift
   let session = URLSession(configuration: .default)
a: if let url = URL(string: "http://stanford.edu/...") {
b:     let task = session.dataTask(with: url) { (data: Data?, response, error) in
c:         // 这里执行数据相关操作
d:         DispatchQueue.main.async {
e:             // 这里执行 UI 相关
           }
f:         print("数据操作已执行完，UI 部分尚未执行")
       }
g:     task.resume()
   }
h: print("已触发获取 url 内容的请求")
```

首先，显然是执行 a，接着执行 b，创建一个 dataTask 赋值给 task，并且 **立即** 返回。

紧接着 b 的是 g，即时在 *后台* 另外一个未知队列（unknown queue, background）上触发 url 请求以获取数据，c d e f 的代码最终会在获取到数据后在另外一个未知队列上执行。所以接着执行 h。

到目前为止，a b g h 依次执行，没有延迟。获取到数据后先执行 c、d，e 处的代码可能由于 主队列 上有其他事务在处理而不能立即执行，所以会先执行 f，最后执行 e。

综合来看，最有可能的执行顺序是：a b g h c d f e。也有可能是 a b g h c d e f（即 e 在 f 之前执行，因为主队列优先级很高，特别是在单核机器上则更有可能）。

### 参考

1. [Stanford CS193P 2017 Lecture 10](https://www.bilibili.com/video/BV1Mx411L7dS?p=12)









