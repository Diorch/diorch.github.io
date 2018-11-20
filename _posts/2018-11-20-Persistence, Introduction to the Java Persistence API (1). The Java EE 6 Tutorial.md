## Entities

*entity* 是一种轻量级的持久化领域对象。典型的，一种*entity*即代表了关系型数据库中的一张表，每个*entity*实例对应着表中的每一行记录。*entity*的持久化状态通过持久化域或持久化属性表示。这些域或者属性通过*ORM*注释将实体和实体关系映射到底层数据存储中的。



这些字段或属性使用对象/关系映射注释将实体和实体关系映射为数据库里的关系型数据。

#### Requirements for Entity Classes

一种*entity*类必须实现以下要求：

* 必须使用`javax.persistence.Entity`注解

* 必须申明一个`public`或`protected`的无参构造函数。此外，还可以申明其他构造函数。
* 不能申明为`final`。持久化的实例变量必须申明为`final`。
* 游离状态下的持久化实体实例通过值传递，例如通过一个会话传递至远程接口，那么该类就必须实现`Serializable`接口。
* 实体可能同时继承实体类与非实体类，非实体类允许继承实体类。
* 持久化实例变量必须申明为`private`, `protected`, `package-private`想要直接访问，只能通过实体类的方法。客户端若想获得实体状态必须通过访问方法或业务方法。

#### Persistent Fields and Properties in Entity Classes

实体的持久化状态可以使用实例变量或属性获取。它的域或者属性必须为以下Java语言类型：

* 简单值类型，如`int`,  `double`, `long`, `boolean`...
* `java.lang.String`
* 其他可序列化类型，包括：
  * 简单值类型的封装类型
  * `java.math.BigInteger`
  * `java.math.BigDecimal` 
  * `java.util.Date`
  * `java.util.Calendar`
  * `java.sql.Date`
  * `java.sql.Time`
  * `java.sql.TimeStamp`
  * 自定义可序列化类型
  * `byte[]`
  * `Byte[]`
  * `char[]`
  * `Character[]`
* 枚举类型
* 其他实体的集合
* 可嵌入类(Embeddable classes ???)

实体使用持久域，持久属性，或两者兼用都是允许的。如果映射注释使用在实体的实例变量，等价于使用域。如果映射注释使用在属性上（getter），那实体使用持久属性。

###### Persistent Fields

如果实体类使用持久域，则持久化直接访问实体类实例变量在运行时。所有未声明`javax.persistentce.Transient`或未标记为Java瞬态的域将持久化到数据库中。必须将*ORM*注释应用于实例变量。

###### Persistent Properties



###### Using Collections in Entity Fields and Properties



###### Validating Persistent Fields and Properties

#### Primary Keys in Entities

#### Multiplicity in Entity Relationships

#### Direction in Entity Relationships

###### Bidirectional Relationships

###### Unidirectional Relationships

###### Queries and Relationship Direction

###### Cascade Operations and Relationships

###### Orphan Removal in Relationships

#### Embeddable Classes in Entities



## Entity Inheritance

#### Abstract Entities

#### Mapped Superclasses

#### Non-ENtity SUperclasses

#### Entity Inheritance Mapping Strategies

###### The Single Table per Class Hierarchy Strategy

###### The Table per Concrete Class Strategy

###### The Joined Subclass Strategy

