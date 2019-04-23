## iOS知识盲点

* 创建弱引用的对象:`[NSValue valueWithNonretainedObject:self]`
* To respond to methods that your object does not itself recognize, you must override methodSignatureForSelector: in addition to forwardInvocation:. The mechanism for forwarding messages uses information obtained from methodSignatureForSelector: to create the NSInvocation object to be forwarded. Your overriding method must provide an appropriate method signature for the given selector, either by pre formulating one or by asking another object for one.
* 强制显示状态栏并回到竖屏状态
    
    ```
    [UIApplication sharedApplication].statusBarOrientation = UIInterfaceOrientationPortrait;
    [UIApplication sharedApplication].statusBarHidden = NO;
    ```

* 注意此方法中的block会强引用被捕获的对象

```
- (id <NSObject>)addObserverForName:(nullable NSNotificationName)name object:(nullable id)obj queue:(nullable NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block
```

* 系统内核级原子性递增方法`OSAtomicIncrement32`

* 判断是否是主线程`pthread_main_np`

* about addMethod
> class_addMethod will add an override of a superclass's implementation, 
> but will not replace an existing implementation in this class. 
> To change an existing implementation, use method_setImplementation

* objc_allocateClassPair:调用此方法后，并不会自动继承父类中的方法，即此时新类的方法列表为空，需要手动为新类添加方法
> To create a new class, start by calling objc_allocateClassPair. 
> Then set the class's attributes with functions like class_addMethod and  class_addIvar.
> When you are done building the class, call objc_registerClassPair. The new class is now ready for use.
> Instance methods and instance variables should be added to the class itself. 
> Class methods should be added to the metaclass.

* LRU(Least recently used)(最近最少使用算法)
* MRU(Most recently used)(最近最常使用算法)
* UIScrollView的属性contentInsetAdjustmentBehavior，不修改默认值的情况下会自动适配iPhoneX系列


