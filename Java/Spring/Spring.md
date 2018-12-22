# Spring
**Bean生命周期**
> Spring 只帮我们管理单例模式 Bean 的完整生命周期，对于 prototype 的 bean ，Spring 在创建好交给使用者之后则不会再管理后续的生命周期。

-------

1. 生命周期图示

![bean_life_cycle](../media/bean_life_cycle1.png)

![bean_life_cycle](../media/bean_life_cycle2.png)

```xml
 
 <bean id="beanFactoryPostProcessor" class="springBeanTest.MyBeanFactoryPostProcessor">
 </bean>

 <bean id="beanPostProcessor" 	      class="springBeanTest.MyBeanPostProcessor">
 </bean>

 <bean id="instantiationAwareBeanPostProcessor" class="springBeanTest.MyInstantiationAwareBeanPostProcessor">
 </bean>

<bean id="person" class="springBeanTest.Person" init-method="myInit" destroy-method="myDestory" scope="singleton" p:name="张三" p:address="广州" p:phone="15900000000" />
```

初始化容器 --> BeanFactoryPostProcessor -->  BeanPostProcessor --> InstantiationAwareBeanPostProcessorAdapter --> postProcessBeforeInstantiation --> Bean构造器，注入属性 --> BeanPostProcessor和InstantiationAwareBeanPostProcessor 调用postProcessBeforeInitialization  --> 容器初始化成功 --> 关闭容器 DiposibleBean.destroy() 和 Bean的destroy().



2. 注解: 继承自Annotation的特殊接口，通过反射获取动态代理对象$Proxy1，传递给InvocationHandler实例的invoke方法处理注解内的方法。该方法会从memberValues这个Map中索引出对应的值。而memberValues的来源是Java常量池

   #####@Component, @Service, @Repository, @Controller :

   ​    都可以作为**@Component**，因为**@Service @Repository @Controller**引用了**@Component** 注解，所以可以被Component-scan扫描到，并且拥有各自适应的切面和切点。

   ------

   ##### @Autowired 与@Resource:  一个按类型装配，一个按名称装配

   1. @Autowired与@Resource都可以用来装配bean. 都可以写在字段上,或写在setter方法上。
   2. @Autowired默认**按类型装配**（这个注解是属业spring的），默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，如：
      @Autowired(required=false) ，如果想按照名称来转配注入，则需要结合@Qualifier一起使用
   3. @Resource **是JDK1.6支持的注解**，**默认按照名称进行装配**，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名，按照名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。只有当找不到与名称匹配的bean才会按照类型来装配注入。

3. **Scope:**

   a. Singleton 

   b. Prototype: 每次注入都会新建bean，初始化以后容器不负责后续生命周期。

   c. Request：每次Http请求会产生新的bean，并仅在该request内有效。

   d. Session：每次Http请求会产生新的bean，并仅在该session内有效。

   e. Global Session：仅在基于portlet的web应用，后同session。

   f. 自定义：继承Scope 但是不能覆盖singleton和prototype

4. ##### AOP

   **代理**：

   1. Java动态代理，就是Java本身提供的代理实现，要求对象必须实现某一个接口。

   2. CGLib库，为Java提供了，为普通类提供代理的功能。

   3. aopalliance，aop联盟包，统一类aop编程的一些概念，这个包里没有具体的类实现，而是定义了几个重要的概念接口，具体的aop实现，要遵从这些接口编程，才能达到一定的通用性。

   4. aspectj包，实现了，通过一种固定的编程语言，通过这种简单的编程语言，我们可以定位到被代理的类，自动完成代理。

      [AOP核心概念](https://www.jianshu.com/p/c403609185a5)


5. JPA, @Repository

   Repository接口有默认实现类SimpleJpaRepository。

   一个未实现的方法是如何运作的？在这个```RepositoryFactorySupport```类中可以看到，首先它获取了Repository的各种信息，包括metadata、customImplementation、information等， 然后使用了`ProxyFactory`，并且为这个`ProxyFactory`注册了几个`Advice`，最后通过`getProxy`方法创建了一个Repository的代理。 而真正起到魔法般作用的应该就是这个[`QueryExecutorMethodInterceptor`](https://github.com/spring-projects/spring-data-commons/blob/master/src/main/java/org/springframework/data/repository/core/support/RepositoryFactorySupport.java#L389)类了



6. 处理异常

   异常处理控制器 @ControllerAdvice 标记该控制器为处理全局异常的控制器，@ExceptionHandler 注解声明异常处理方法，如果 @ExceptionHandler 注解中未声明要处理的异常类型，则默认为参数列表中的异常类型。所以上面的写法，还可以写成这样：

   ```java
   @ExceptionHandler(Exception.class)
   @ResponseBody
   String handleException(){
      return "Exception Deal!";
   }
   // ----------
   @ExceptionHandler
   @ResponseBody
   String handleException(Exception e){
       return "Exception Deal! " + e.getMessage();
   }
   ```

