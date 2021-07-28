

## 声明特性

***

### **@discardableResult**

>该特性用于的函数或方法声明，以抑制编译器中函数或方法被调用而其返回值没有被使用时的警告。
>
>discard: vt. 抛弃、丢弃、放弃
>
>discardable: adj. 可废弃的

示例（[Alamofire-ResponseSerialization](https://github.com/Alamofire/Alamofire/blob/master/Source/ResponseSerialization.swift)）：

```swift
extension DataRequest {
    /// 添加一个请求完成时被调用的 handler
    ///
    /// - Parameters:
    ///   - queue:             完成回调 handler 被分发的队列，默认为 `.main`。
    ///   - completionHandler: 请求完成时被执行的代码。
    ///
    /// - Returns:             请求实例自身。
    @discardableResult
    public func response(queue: DispatchQueue = .main, completionHandler: @escaping (AFDataResponse<Data?>) -> Void) -> Self {
        appendResponseSerializer {
            let result = AFResult<Data?>(value: self.data, error: self.error)
            self.underlyingQueue.async {
                let response = DataResponse(request: self.request,
                                            response: self.response,
                                            data: self.data,
                                            metrics: self.metrics,
                                            serializationDuration: 0,
                                            result: result)

                self.eventMonitor?.request(self, didParseResponse: response)

                self.responseSerializerDidComplete { queue.async { completionHandler(response) } }
            }
        }

        return self
    }
}
```

有时不会使用返回的请求实例，所以使用 **@discardableResult** 屏蔽编译器警告。

