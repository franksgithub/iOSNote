# NSPort

可用于线程间通信

```
@interface ViewController ()<NSPortDelegate> {
    NSPort *port;
}
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    port = [[NSPort alloc] init];
    port.delegate = self;
    [[NSRunLoop currentRunLoop] addPort:port forMode:NSRunLoopCommonModes];
}

- (void)sendPort {
    NSLog(@"1 : %@", [NSThread currentThread]);
    [port sendBeforeDate:[NSDate date]  msgid:12345 components:nil from:nil reserved:0];
    NSLog(@"2 : %@", [NSThread currentThread]);
}

- (void)handlePortMessage:(NSPortMessage *)message {
    NSLog(@"3 : %@", [NSThread currentThread]);
    NSObject *msg = (NSObject *)message;
    NSLog(@"%@", [msg valueForKey:@"msgid"]);
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [NSThread detachNewThreadSelector:@selector(sendPort) toTarget:self withObject:nil];
}

@end
```


