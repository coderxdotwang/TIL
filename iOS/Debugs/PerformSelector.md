## 场景

A 传入 *target*（方法接收者）及 `SEL` / *selector*  到 B，在 B 中通过 `[target performSelector:selector]` 调用，会触发编译警告：

> PerformSelector may cause a leak because its selector is unknown

## 大致原因

由于 performSelector 方法有一个 id 类型的返回：

`- (id)performSelector:(SEL)aSelector;`

编译器假定这个返回的对象指针可能不会被正确处理。

## 解决方法

使用 **Objc Runtime** 里的 [`methodForSelector:`](https://developer.apple.com/documentation/objectivec/nsobject/1418863-methodforselector/) 方法获取 *target* 中 *selector* 的实现 **`IMP`** 地址，然后通过方法调用让 *target* 执行 *selector*。

`((void (*)(id, SEL))[_target methodForSelector:_selector])(_target, _selector)`

### 静态  Seletors

通过 `@selector` 方式获取的 `SEL` 不会触发警告，因为编译期可以决定是否会 leak。

`[target performSelector:@selector(someMethod)];`

### Swift 警告

Swift 中已经不建议使用这个方法，由于内部缺乏类型安全保证。

> Because of the inherent lack of type safety, this API is not recommended for use in Swift unless your code specifically relies on the dynamic method resolution provided by the Objective-C run-time.

## 参考

1. Stackoverflow - [performSelector may cause a leak because its selector is unknown](https://stackoverflow.com/questions/7017281/performselector-may-cause-a-leak-because-its-selector-is-unknown)
2. Apple doc: [objectivec/1418956-nsobject/1418867-performselector](https://developer.apple.com/documentation/objectivec/1418956-nsobject/1418867-performselector?language=objc)；[/objectivec/nsobjectprotocol/1418867-perform](https://developer.apple.com/documentation/objectivec/nsobjectprotocol/1418867-perform)