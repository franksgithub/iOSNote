## LLDB(Low Lever Debug)

### 断点

* 对方法名下断点
`breakpoint set -n test1`
`breakpoint set --selector touchesBegan:withEvent`

* 指定类中的多个方法下断点
`breakpoint set -n "[ViewController save:]" -n "[ViewController open:]"`

* 指定文件的方法下断点
`breakpoint set --file ViewController.m --selector touchesBegan:withEvent:`

* 遍历整个项目中所有包含Game的方法，并设置断点
`breakpoint set -r Game`

* 查看全部断点信息
`breakpoint list`

* 禁用断点
`breakpoint disable 1`

* 启用断点
`breakpoint enable 1`

* 删除断点，以组为单位
`breakpoint delete 1`

* 查看breakpoint命令详情，help也可用于查看其它命令详情
`help breakpoint`

* 注：
breakpoint可简写为b

-
### 内存断点

* 变量下断点
`watchpoint set variable p1->name`

* 内存地址下断点
`watchpoint set expression 0x000900`

-

### LLDB下执行代码

* 执行代码，因涉及 UI 更新，下边的示例代码需过掉断点才能生效，如`c`
`p sele.view.backgroundColor = [UIColor redColor]`

* 修改对象属性值，不强转的话，$3默认是id类型

    ```
    p (Person *)self.users.lastObject
    p $3.name = "John"
    ```

* 可一次写多行代码,换行用ctrl+enter

    ```
    p NSObject *obj = [NSObject new];
    [sele.users addObject:obj];
    ```

* break command
    * 添加触发断点时执行的指令

        `break command add 2`
    * 接着输入要执行的指令,如

        ```
        po self 
        po self.name
        ```

-

### 流程控制
* 继续执行
`c`

* 单步运行,将子函数当作整体一步执行
`n`

* 单步运行,会进行函数内部
`s`

* 汇编级别单步运行
`ni`

* 汇编级别会跳入函数内部
`si`

* 查看调用栈
`bt`

* 回退
`up`

* 下一个
`down`

* 代码回滚,不再继续执行
`thread return`

-

### stop-hook
触发任何断点都会执行相应的指令，只对breakpoint,watchpoint断点起作用
`target stop-hook add -o "frame variable"`

-

### image
* 对查看没有明显堆栈信息的崩溃日志，很有帮助

    `image lookup -a 0x10445a38`
    
* 快速查看Person类的信息
`image lookup -t Person`
* 查看app相关的系统或三方库信息
`image list`

### 内存查看

* 查看内存中的数据

    `memory read 0x00009`
    `x 0x0009`
    
    -
    
### lldbinit
* 在当前用户目录下可以自己手动创建.lldbinit文件，在此文件中可输入每次进行断点时执行的指令。LLDB启动时会加载.lldbinit文件，程序进入断点时就会执行.lldbinit中的相关指令。

-

### other
* 查看对象图深度为2的对象信息

    `expr -P 2 -- self`


