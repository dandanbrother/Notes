```java
public class Singleton {
     private static volatile Singleton singleton = null; //1. 可见性 2. 防止指令重排
     private Singleton(){}
     public static Singleton getSingleton(){
        if(singleton == null){
            synchronized (Singleton.class){
                if(singleton == null){
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }  
}
```

如果强制实现了Cloneable接口，则会创建一个新对象（浅拷贝）。