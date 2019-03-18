```java
public class Singleton {
     private static volatile Singleton singleton = null; //1. 可见性 2. 防止指令重排
     private Singleton(){}
     public static Singleton getSingleton(){
        if(singleton == null){
            synchronized (Singleton.class){
                if(singleton == null){
                    singleton = new Singleton(); // 这一步会有多个指令，指令重排会导致先赋值再初始化。
                }
            }
        }
        return singleton;
    }  
}
```

如果强制实现了Cloneable接口，则会创建一个新对象（浅拷贝）。