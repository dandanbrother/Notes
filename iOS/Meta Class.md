#类 元类(meta-class)
对象的类，取决于它的isa指针

```c
typedef struct objc_object {
    Class isa;
} *id;
```
####OC最重要的特性是消息机制
```objective-c
[@"stringValue"
    writeToFile:@"/file.txt" atomically:YES encoding:NSUTF8StringEncoding error:NULL];
```


>This works because when you send a message to an Objective-C object (like the NSCFString here), the runtime follows object's isa pointer to get to the object's Class (the NSCFString class in this case). The Class then contains a list of the Methods which apply to all objects of that Class and a pointer to the superclass to look up inherited methods. The runtime looks through the list of Methods on the Class and superclasses to find one that matches the message selector (in the above case, writeToFile:atomically:encoding:error on NSString). The runtime then invokes the function (IMP) for that method.

这段代码，先向`NSString`对象(`@"stringValue"`) 发送 `writeToFile` 消息，runtime根据该对象中的isa指针找到对应的类(`NSString`)。类结构体中包含一个方法列表的指针，以及一个指向父类的指针，便于runtime查找与消息选择器(message selector)匹配的方法。最后runtime引用方法所对应的实现(`IMP`). 

####什么是元类
>A Class in Objective-C is also an object

首先，类也是个对象。那就意味着，你也可以向类发送消息

```
NSStringEncoding defaultStringEncoding = [NSString defaultStringEncoding];
```
在这里，`defaultStringEncoding`消息被发送给`NSString`类。因为类就是它本身的一个对象，那也就是说，`Class`结构体一定以一个`isa`指针开始。


```
typedef struct objc_class *Class;
struct objc_class {
    Class isa;
    Class super_class;
    /* followed by runtime specific details... */
};
```
但是，为了让我们能够让一个`Class`引用一个方法，这个类结构体中的isa指针必须指向一个类结构体，其中包含一个方法列表指针并拥有我们所引用的方法。

因此，这就导出了元类的定义：**元类是一个类的类**
>This leads to the definition of a meta-class: the meta-class is the class for a `Class` object.

简单来说：
* 当你向一个对象发送一条消息时，这条消息在该对象的类中寻找。
* 当你想一个类发送一条消息时，这条消息在该类的元类中寻找。


```
When you send a message to an object, that message is looked up in the method list on the object's class.
When you send a message to a class, that message is looked up in the method list on the class' meta-class.
```
元类非常必要，因为它保存了这个类的方法。对于每一个`Class`都必须有一个单独的元类与之对应，因为每个`Class`都可能包含完全不一样的方法。

####元类是什么类? (What is the class of the meta-class?)
`meta-class`，就像之前的`Class`一样，它也是对象。这就说，你也可以对它引用方法。所以，这也意味着，`meta-class`也有一个`Class`。
所有的元类都使用根类的元类(在继承体系中最顶上类的元类)作为他们的类。也就是说，对于几乎所有类都使用`NSObject`的元类作为自己的类。
根类是它自己本身的类。(它们的isa指针指向它们自己)
>The result of this inheritance hierarchy is that all instances, classes and meta-classes in the hierarchy inherit from the hierarchy's base class.

####下面做一个实验验证
用runtime创建一个类：

```
Class newClass =
    objc_allocateClassPair([NSError class], "RuntimeErrorSubclass", 0);
class_addMethod(newClass, @selector(report), (IMP)ReportFunction, "v@:");
objc_registerClassPair(newClass);
```
我们添加的`ReportFunction`方法实现为：

```
void ReportFunction(id self, SEL _cmd)
{
    NSLog(@"This object is %p.", self);
    NSLog(@"Class is %@, and super is %@.", [self class], [self superclass]);
    
    Class currentClass = [self class];
    for (int i = 1; i < 5; i++)
    {
        NSLog(@"Following the isa pointer %d times gives %p", i, currentClass);
        currentClass = object_getClass(currentClass);
    }

    NSLog(@"NSObject's class is %p", [NSObject class]);
    NSLog(@"NSObject's meta class is %p", object_getClass([NSObject class]));
}
```
为了执行`ReportFunction`，我们需要创建一个实例。

```
id instanceOfNewClass =
    [[newClass alloc] initWithDomain:@"someDomain" code:0 userInfo:nil];
[instanceOfNewClass performSelector:@selector(report)];
[instanceOfNewClass release];
```
因为没有`report`方法，所以用`performSelector:`引用它，这样编译器不会报错。
`ReportFunction`方法将会沿着isa指针检索，并告诉我们什么是`class`，`meta-class`，`class of meta-class`。不断获取类的地址。
>Getting the class of an object: the ReportFunction uses object_getClass to follow the isa pointers because the isa pointer is a protected member of the class (you can't directly access other object's isa pointers). The ReportFunction does not use the class method to do this because invoking the class method on a Class object does not return the meta-class, it instead returns the Class again (so [NSString class] will return the NSString class instead of the NSString meta-class).

让我们来看看`ReportFunction`的输出， 

```
2016-10-09 12:58:46.560 iOS进阶[2753:112428] This object is 0x608000240690.
2016-10-09 12:58:46.561 iOS进阶[2753:112428] Class is RuntimeErrorSubclass, and super is NSError.
2016-10-09 12:58:46.561 iOS进阶[2753:112428] Following the isa pointer 1 times gives 0x6080002406f0
2016-10-09 12:58:46.562 iOS进阶[2753:112428] Following the isa pointer 2 times gives 0x6080002406c0
2016-10-09 12:58:46.562 iOS进阶[2753:112428] Following the isa pointer 3 times gives 0x1093cce08
2016-10-09 12:58:46.562 iOS进阶[2753:112428] Following the isa pointer 4 times gives 0x1093cce08
2016-10-09 12:58:46.562 iOS进阶[2753:112428] NSObject's class is 0x1093cce58
2016-10-09 12:58:46.562 iOS进阶[2753:112428] NSObject's meta class is 0x1093cce08
```
* 对象的地址是`0x608000240690`.
* 类(`RuntimeErrorSubclass`)的地址是`0x6080002406f0`.
* 元类的地址是`0x10010c630`.
* 元类的类的地址是`0x1093cce08`.
* `NSObject`的元类的类就是他自己.

####总结
元类就是一个`Class`的类。元类总会确保`Class`对象拥有所有的基类的实例和类方法。对于从NSObject继承下来的类，这意味着所有的`NSObject`实例和`protocol`方法在所有的类（和`meta-class`）中都可以使用。
所有的`meta-class`使用基类的`meta-class`作为自己的基类，对于顶层基类的`meta-class`也是一样，只是它指向自己而已。


