# load-initialize

1. load
    * The load message is sent to classes and categories that are both dynamically loaded and statically linked, but only if the newly loaded class or category implements a method that can respond.(不管是动态加载，还是动态链接，只有类和类别自己实现了load方法，才会在runtime加载类时调用。)
    * The order of initialization is as follows:
        1. All initializers in any framework you link to.
        2. All +load methods in your image.
        3. All C++ static initializers and C/C++ __attribute__(constructor) functions in your image.
        4. All initializers in frameworks that link to you.

    * In addition:
        1. A class’s +load method is called after all of its superclasses’ +load methods.(父类的load的方法要先于子类的被调用)
        2. A category +load method is called after the class’s own +load method.(类别的load方法要比类的调用的晚)
    * In a custom implementation of load you can therefore safely message other unrelated classes from the same image, but any load methods implemented by those classes may not have run yet.(在自己实现的load方法中，可以安全的给其它类发消息，但其它类的load方法可能还没执行)
    
2. initialize
    * Initializes the class before it receives its first message.(在类收到第一个方法调用之前，初始化类)
    * The runtime sends initialize to each class in a program just before the class, or any class that inherits from it, is sent its first message from within the program. Superclasses receive this message before their subclasses.(在第一次调用类的方法之前，runtime会调用类的initialize方法，父类的initialize方法会先于子类被调用)
    * The runtime sends the initialize message to classes in a thread-safe manner. That is, initialize is run by the first thread to send a message to a class, and any other thread that tries to send a message to that class will block until initialize completes.
    * Because initialize is called in a blocking manner, it’s important to limit method implementations to the minimum amount of work necessary possible. (runtime以阻塞线程的方法来调用initialize方法，所以尽可能的减少在initialize中执行的任务数是非常重要的)
    * The superclass implementation may be called multiple times if subclasses do not implement initialize—the runtime will call the inherited implementation—or if subclasses explicitly call [super initialize].（如果子类未实现initialize而父类实现的，父类的initialize方法会被多次调用）

