## GCD

1. 串行：当前任务不执行完，后续的任务无法开始执行，不管有多少线程。
2. 并行：当前任务不执行完，后续的任务也可以执行。


##### 此例子可用来帮助理解串行队列和并行队列的深层次的区别
此例子用来解除之前自己觉得dispatch_sync同步派发任务到并行和串行队列无区别的错误想法。如果在同一个线程上派发，确实无区别，但如果考虑有多个线程并行访问队列，则区别一目了然。


```
- (void)gcdConcurreceQueueTest {
    dispatch_queue_t conQueue = dispatch_queue_create("com.concurrent.zz", DISPATCH_QUEUE_CONCURRENT);
    dispatch_queue_t globalQueue = dispatch_get_global_queue(0, 0);
    dispatch_async(globalQueue, ^{
        dispatch_sync(conQueue, ^{
            NSLog(@"start sync work");
            sleep(3);
            NSLog(@"finish sync work");
        });
    });
    dispatch_async(globalQueue, ^{
        dispatch_async(conQueue, ^{
            NSLog(@"start async work");
            NSLog(@"finish async work");
        });
    });
}
```


