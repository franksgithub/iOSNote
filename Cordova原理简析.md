# Cordova原理简析

1. H5页面调用插件的js方法，前台js框架把调用的方法存入队列(`commandQueue`)中，相关参数包括类名、方法名、参数、回调id等。
2. 通过`execIframe.contentWindow.location = 'gap://ready'`，修改webview的url，从而执行UIWebView的代理方法`- (BOOL)webView:(UIWebView*)webView shouldStartLoadWithRequest:(NSURLRequest*)request navigationType:(UIWebViewNavigationType)navigationType`。
3. 判断`if ([[url scheme] isEqualToString:@"gap"])`，然后执行js方法`nativeFetchMessages`，从js的`commandQueue`中取出要执行的方法。
4. 根据方法的参数，查找对应的插件类及方法，并最终以`objc_msgSend`的方式执行插件的方法。
5. 将方法执行的结果，以调用UIWebView的`stringByEvaluatingJavaScriptFromString`方法，对应的js方法为`cordova.require('cordova/exec').nativeCallback('IMSDKPlugin92409026',1,"1",0, 1)`，从而将结果传回给H5页面。

## 注
* 前台js框架中会用数据来维护一个callbacks数组，来解决回调的问题

