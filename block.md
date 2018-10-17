# Block

1. Block的实质:带有自动变量值的匿名函数。
2. `clang -rewrite-objc 源代码文件名`
3. 截获自动变量：在执行block语法时，Block语法表达式所使用的自动变量值被保存到Block的结构体实例中(即Block自身)中。
4. __block存储域说明符：用于指定将变量值设置到哪个存储域中。

    * `__block int val = 10`会变成结构体类型的自动变量，即栈上生成的结构体实例。
    
    ```
    struct __Block_byref_val_0 {
        void *__isa;
        struct __Block_byref_val_0 *__forwarding;
        int __flags;
        int __size;
        int val;
    };
    ```
5. 使用__block变量的 Block 持有__block变量，如果 Block 被废弃，它所持有的__block变量也就被释放。
6.  __block变量用结构体成员变量__forwarding的原因：通过Block的复制，__block变量也会从栈复制到堆。栈上的__block变量用结构体实例在__block变量从栈复制到堆上时，会将成员变量__forwarding的值替换为复制到堆上的__block变量用结构体实例的地址。

    ```
    __block int val = 0;
    void (^blk)(void) = [^{
        ++val;
        NSLog(@"in heap : %p, %d", &val, val);
    } copy];
    blk();
    ++val;
    NSLog(@"in stack : %p, %d", &val, val);
    ```


