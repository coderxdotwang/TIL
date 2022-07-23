#WKUIDelegate

实现 [WKUIDelegate](https://developer.apple.com/documentation/webkit/wkuidelegate) 代理中的 `createWebViewWithConfiguration` 方法，响应网页内调用 `window.open(URL)` 方法打开新 Web。

```SWIFT
func webView(
    _ webView: WKWebView,
    createWebViewWith configuration: WKWebViewConfiguration,
    for navigationAction: WKNavigationAction,
    windowFeatures: WKWindowFeatures
) -> WKWebView? {
    if navigationAction.targetFrame == nil || navigationAction.targetFrame?.isMainFrame == false {
        // 加载请求或者创建新 Web 页面
        webView.load(navigationAction.request)
	}
    return nil
}
```

- 如果不实现这个可选代理方法，则调用 `window.open(URL)` 方法会没有响应。
- 注意：网页内调用 `window.open(URL)` 方法传参必须带 `scheme` 如 *https://xxx.com*，否则传递到 navigationAction.request 里的 URL 格式将是 **源页面地址/目标页**，例如在 *https://www.abc.com* 调用 `window.open(www.def.com)`，收到的参数是：*https://www.abc.com/www.def.com*。
- 可以根据 `configuration`、`windowFeatures` 等参数来配置新 Web。