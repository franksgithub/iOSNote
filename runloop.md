# RunLoop

### RunLoop与线程的关系

* 苹果不允许直接创建RunLoop，它提供了两个自动获取的函数:`CFRunLoopGetMain()`和`CFRunLoopGetCurrent()`。
* 线程和RunLoop之间是一一对应的，其关系是保存在一个全局Dictionary中的。线程刚创建时并没有RunLoop，如果你不主动获取，那它一直都不会有。
* RunLoop的创建是发生在第一次获取时，RunLoop的销毁是发生在线程线束时。你只能在一个线程的内部获取其RunLoop（主线程除外）。

### RunLoop的内部结构


```
CFRunLoopRef
CFRunLoopModeRef
CFRunLoopSourceRef
CFRunLoopTimerRef
CFRunLoopObserverRef
```
关系如下：
![RunLoop_0](media/RunLoop_0-1.png)

一个RunLoop包含若干个Mode，每个Mode又包含若干个Source/Timer/Observer。每次调用RunLoop的主函数时，只能指定其中一个Mode，这个Mode被称作CurrentMode，如果需要切换Mode，只能退出Loop，再重新指定一个Mode进入。这样做主要是为了分隔开不同组的Source/Timer/Observer，让其互不影响。

CFRunLoopSourceRef是事件产生的地方。Source有两个版本:Source0和Source1。
    * Source0：只包含了一个回调，它并不能主动触发事件。使用时，你需要先调用CFRunLoopSourceSignal(source)，将这个Source标记为待处理，然后手动调用CFRunLoopWakeUp(source)来唤醒RunLoop，让其处理事件。
    * Source1：包含了一个mach_port和一个回调，被用于通过内核和其他线程相互发送消息，可以主动唤醒RunLoop的线程。

CFRunLoopTimerRef是基本时间的触发器，它和NSTimer是toll-freee bridged的，可以混用。其包含一个时间长度和一个回调。当其加入到RunLoop时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

CFRunLoopObserverRef是观察者，每个Observer都包含了一个回调，当RunLoop的状态发生变化时，观察者就能通过回调接受这个变化。可以观测的时间点有以下几个:

```
/* Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0),           // 即将进入Loop
    kCFRunLoopBeforeTimers = (1UL << 1),    // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2),   // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5),   // 即将进入休眠
    kCFRunLoopAfterWaiting = (1UL << 6),    // 刚从休眠中唤醒
    kCFRunLoopExit = (1UL << 7),            // 即将退出Loop
    kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```
上面的Source/Timer/Observer被统称为mode item，一个item可以被同时加入多个mode，但一个item被重复加入同一个mode时是不会有效果的。如果一个mode中一个item也没有，则RunLoop会直接退出，不进入循环。

### RunLoop 的 Mode

CFRunLoopMode 和 CFRunLoop 的结构大致如下：

```
struct __CFRunLoopMode {
    CFStringRef _name;            // Mode Name, 例如 @"kCFRunLoopDefaultMode"
    CFMutableSetRef _sources0;    // Set
    CFMutableSetRef _sources1;    // Set
    CFMutableArrayRef _observers; // Array
    CFMutableArrayRef _timers;    // Array
    ...
};
 
struct __CFRunLoop {
    CFMutableSetRef _commonModes;     // Set
    CFMutableSetRef _commonModeItems; // Set<Source/Observer/Timer>
    CFRunLoopModeRef _currentMode;    // Current Runloop Mode
    CFMutableSetRef _modes;           // Set
    ...
};
```    
CommonModes:一个mode可以将自己标记为"Common"属性。每当RunLoop的内容发生变化时，RunLoop都会自动将_commonModeItems里的Source/Observer/Timer同步到具有"Common"标记的所有mode里。

应用场景举例：主线程的 RunLoop 里有两个预置的 Mode：kCFRunLoopDefaultMode 和 UITrackingRunLoopMode。这两个 Mode 都已经被标记为”Common”属性。DefaultMode 是 App 平时所处的状态，TrackingRunLoopMode 是追踪 ScrollView 滑动时的状态。当你创建一个 Timer 并加到 DefaultMode 时，Timer 会得到重复回调，但此时滑动一个TableView时，RunLoop 会将 mode 切换为 TrackingRunLoopMode，这时 Timer 就不会被回调，并且也不会影响到滑动操作。

有时你需要一个Timer，在两个mode中都能得到回调，一种办法是将这个 Timer 分别加入这两个 mode。还有一种方式，就是将 Timer 加入到顶层的RunLoop的"_commonModeItems"中。"_commonModeItems"会被RunLoop自动更新到所有具有"Common"属性的mode里去。

CFRunLoop对外暴露的管理 Mode 接口只有下面2个:

```
CFRunLoopAddCommonMode(CFRunLoopRef runloop, CFStringRef modeName);
CFRunLoopRunInMode(CFStringRef modeName, ...);
```

Mode 暴露的管理 mode item 的接口有下面几个：

```
CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
```

你只能通过mode name来操作内部的mode，当你传入一个新的mode name但RunLoop内部没有对应的mode时，RunLoop会自动帮你创建对应的CFRunLoopModeRef。对于一个RunLoop来说，其内部的mode只能增加不能删除。

### RunLoop 的内部逻辑

根据苹果在文档里的说明，RunLoop 内部的逻辑大致如下:
![RunLoop_1](media/RunLoop_1-1.png)

### RunLoop 的底层实现

RunLoop 的核心就是一个 mach_msg()，RunLoop 调用这个函数去接收消息，如果没有别人发送 port 消息过来，内核会将线程置于等待状态。例如你在模拟器里跑起一个 iOS 的 App，然后在 App 静止时点击暂停，你会看到主线程调用栈是停留在 mach_msg_trap() 这个地方。   

### AutoreleasePool

在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的AutoreleasePool 环绕着，所以不会出现内存泄漏，开发者也不必显示创建 Pool 了。

### RunLoop什么时候会被销毁？
线程销毁时，在创建RunLoop的时候，会注册一个回调函数，在对应的线程销毁时调用。

### Other
*  Port-based sources are signaled automatically by the kernel, and custom sources must be signaled manually from another thread.

-
参考链接:[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/#base)



