# <? extends A> & <? super A>
**<? extends A>**  上界通配符，放入A及A的子类，只能get()，并存于A或A的基类。

```java
list.add(new A());
list.add(new ASubClass());
A a = list.get();
ASuperClass asc = list.get();
```

编译器只知道是A或A的子类，具体未知。

**<? super A>** 下界通配符，放入A及A的基类，只能add()，只能取出放入Object类。

```Java
list.add(new A());
list.add(new ASuperClass());
Object a = list.get();
```

编译器只知道是A或A的基类，具体未知。所以Object作为任何类的父类符合要求。

**PECS（Producer Extends Consumer Super**
1. 频繁往外读取内容的，适合用上界Extends。
2. 经常往里插入的，适合用下界Super。



**? 通配符: **无需关注具体类型，是未知的，可以调用任何与类型无关的方法。

