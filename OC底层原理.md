# 底层原理

#### Category的实现原理


运行时会动态的将Category中新加的方法，添加到类的方法列表中，且如果是已存在的方法，会位于原方法之前，这就是Category中的方法会覆盖原类方法的原因。但原方法也还是存在的，只是排序靠后，而方法调用查询方法列表时，排在前面的方法会先命中，从而不再继续查找。


--

[iOS底层原理总结](https://www.jianshu.com/notebooks/24110540)
[深入理解Objective-C：Category](https://tech.meituan.com/DiveIntoCategory.html)

  


