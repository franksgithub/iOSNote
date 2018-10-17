## iOS知识盲点

* 创建弱引用的对象:`[NSValue valueWithNonretainedObject:self]`
* To respond to methods that your object does not itself recognize, you must override methodSignatureForSelector: in addition to forwardInvocation:. The mechanism for forwarding messages uses information obtained from methodSignatureForSelector: to create the NSInvocation object to be forwarded. Your overriding method must provide an appropriate method signature for the given selector, either by pre formulating one or by asking another object for one.


