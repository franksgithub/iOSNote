# About NSTimer

NSTimer是和RunLoop相关联的，RunLoop到约定的时间点会触发任务。创建NSTimer时，可以将其预先安排在当前的运行循环中，也可以先创建好，然后开发者自己来调度。无论采用哪种方式，只有把NSTimer放在RunLoop中，它才能正常触发任务。

自己创建的线程中，需要配置好RunLoop，NSTimer才能正常运行。


```
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
+ (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
```

因为NSTimer会保持对target的强引用，所以上面的两种创建NSTimer的方式，如果repeats为YES，而NSTimer又是target对象中的成员变量，就会导致retain cycle。

解决方法

* 在合适的时机调用NSTimer的invalidate方法，令NSTimer失效。
* 使用苹果新提供的两个API来创建NSTimer，但只能在iOS10以上的版本中使用。

    ```
    + (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block;
    
    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block;
    ```

* 自己实现NSTimer的Category，同样以使用Block的方式，来支持老版本。


    ```
    @interface NSTimer (BlockSupport)
    
    + (NSTimer *)bst_scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)())block;
    
    @end
    
    @implementation NSTimer (BlockSupport)
    
    + (NSTimer *)bst_scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)())block {
        return [self scheduledTimerWithTimeInterval:interval target:self selector:@selector(bst_blockInvoke:) userInfo:[block copy] repeats:repeats];
    }
    
    + (void)bst_blockInvoke:(NSTimer *)timer {
        void (^block)() = timer.userInfo;
        if (block) {
            block();
        }
    }
    
    @end
    ```


