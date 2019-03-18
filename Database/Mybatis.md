###{}和${}的区别是什么？

在Mybatis中，有两种占位符

- `#{}`解析传递进来的参数数据
- ${}对传递进来的参数**原样**拼接在SQL中
- **#{}是预编译处理，${}是字符串替换**。
- 使用#{}可以有效的防止SQL注入，提高系统安全性



## 当实体类中的属性名和表中的字段名不一样 ，怎么办 ？

第1种： 通过在查询的sql语句中**定义字段名的别名，让字段名的别名和实体类的属性名一致**

```xml
     <select id=”selectorder” parametertype=”int” resultetype=”me.gacl.domain.order”> 
       select order_id id, order_no orderno ,order_price price form orders where order_id=#{id}; 
    </select> 
```

第2种： **通过\<resultMap>来映射字段名和实体类属性名的一一对应的关系**

```xml
 <select id="getOrder" parameterType="int" resultMap="orderresultmap">
        select * from orders where order_id=#{id}
    </select>
   <resultMap type=”me.gacl.domain.order” id=”orderresultmap”> 
        <!–用id属性来映射主键字段–> 
        <id property=”id” column=”order_id”> 
        <!–用result属性来映射非主键字段，property为实体类属性名，column为数据表中的属性–> 
        <result property = “orderno” column =”order_no”/> 
        <result property=”price” column=”order_price” /> 
    </reslutMap>
```



## Mybatis动态sql是做什么的？都有哪些动态sql？能简述一下动态sql的执行原理不？

- Mybatis动态sql可以让我们在Xml映射文件内，**以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能**。
- Mybatis提供了9种动态sql标签：trim|where|set|foreach|if|choose|when|otherwise|bind。
- 其执行原理为，使用OGNL从sql参数对象中计算表达式的值，**根据表达式的值动态拼接sql，以此来完成动态sql的功能**。



## 通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？

- Dao接口，就是人们常说的Mapper接口，接口的全限名，就是映射文件中的namespace的值，接口的方法名，就是映射文件中MappedStatement的id值，接口方法内的参数，就是传递给sql的参数。
- Mapper接口是没有实现类的，当调用接口方法时，**接口全限名+方法名**拼接字符串作为key值，可唯一定位一个MappedStatement。所以不能重载。



# 原理

#### 1. 构建SqlSessionFactory过程

构建主要分为2步：

- 通过XMLConfigBuilder解析配置的XML文件，读出配置参数，包括基础配置XML文件和映射器XML文件；
- 使用Configuration对象创建SqlSessionFactory，SqlSessionFactory是一个接口，提供了一个默认的实现类DefaultSqlSessionFactory。

说白了，就是将我们的所有配置解析为Configuration对象，在整个生命周期内，可以通过该对象获取需要的配置。

由于插件需要频繁访问映射器的内部组成，会重点这部分，了解这块配置抽象出来的对象：

##### MappedStatement

它保存映射器的一个节点（select|insert|delete|update），包括配置的SQL，SQL的id、缓存信息、resultMap、parameterType、resultType等重要配置内容。

它涉及的对象比较多，一般不去修改它。

##### SqlSource

它是MappedStatement的一个属性，主要作用是根据参数和其他规则组装SQL，也是很复杂的，一般也不用修改它。

##### BoundSql

对于参数和SQL，主要反映在BoundSql类对象上，在插件中，通过它获取到当前运行的SQL和参数以及参数规则，作出适当的修改，满足特殊的要求。

BoundSql提供3个主要的属性：parameterObject、parameterMappings和sql，下面分别来介绍。

parameterObject为参数本身，可以传递简单对象、POJO、Map或@Param注解的参数：

- 传递简单对象(int、float、String等)，会把参数转换为对应的类，比如int会转换为Integer；
- 如果传递的是POJO或Map，paramterObject就是传入的POJO或Map不变；
- 如果传递多个参数，没有@Param注解，parameterObject就是一个Map<String,Object>对象，类似这样的形式{"1":p1 , "2":p2 , "3":p3 ... "param1":p1 , "param2":p2 , "param3",p3 ...}，所以在编写的时候可以使用#{param1}或#{1}去引用第一个参数；
- 如果传递多个参数，有@Param注解，与没有注解的类似，只是将序号的key替换为@Param指定的name；

parameterMappings，它是一个List，元素是ParameterMapping对象，这个对象会描绘sql中的参数引用，包括名称、表达式、javaType、jdbcType、typeHandler等信息。

sql，是写在映射器里面的一条sql。

有了Configuration对象，构建SqlSessionFactory就简单了：

```
sqlSessionFactory = new SqlSessionFactoryBuilder().bulid(inputStream);
```

#### 2. SqlSession运行过程

总结下映射器的调用过程，返回的Mapper对象是代理对象，当调用它的某个方法时，其实是调用MapperProxy#invoke方法，而映射器的XML文件的命名空间对应的就是这个接口的全路径，会根据全路径和方法名，便能够绑定起来，定位到sql，最后会使用SqlSession接口的方法使它能够执行查询。



# JSR303

Bean Validation : 使用hibernate-validator 实现