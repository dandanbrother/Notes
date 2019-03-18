# Servlet & Jsp

主要不同：

- Servlet在Java代码中通过HttpServletResponse对象动态输出HTML内容
- JSP在静态HTML内容中嵌入Java代码，Java代码被动态执行后生成HTML内容



区别：

1、JSP第一次运行的时候会编译成Servlet，驻留在内存中以供调用。
2、JSP是web开发技术，Servlet是服务器端运用的小程序，我们访问一个JSP页面时，服务器会将这个JSP页面转变成Servlet小程序运行得到结果后，反馈给用户端的浏览器。
3、Servlet相当于一个控制层再去调用相应的JavaBean处理数据,最后把结果返回给JSP。
4、Servlet主要用于转向，将请求转向到相应的JSP页面。
5、JSP更多的是进行页面显示，Servlet更多的是处理业务，即JSP是页面，Servlet是实现JSP的方法。
6、Servlet可以实现JSP的所有功能，但由于美工使用Servlet做界面非常困难，后来开发了JSP。
7、JSP技术开发网站的两种模式：JSP + JavaBean；JSP + Servlet + JavaBean（一般在多层应用中, JSP主要用作表现层,而Servlet则用作控制层,因为在JSP中放太多的代码不利于维护，而把这留给Servlet来实现,而大量的重复代码写在JavaBean中