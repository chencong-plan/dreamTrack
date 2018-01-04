---
title: hibernate篇——基本配置及增删改查
date: 2017-12-27 10:20:50
tags:
    - hibernate
    - 框架
categories:
    - hibernate
---


## 前言

> Hibernate属于ORM框架，至于什么是ORM框架，[搜索一下](https://www.baidu.com/s?wd=orm%E6%A1%86%E6%9E%B6)都有很多的。简而言之：
- 实现java对象和数据库表之间的映射。
- 提供对数据操作的方法。
- 减少开发人员使用SQL和JDBC处理数据。
- 能够使开发人员面向业务模型/商业逻辑编程。
- 减少特定数据库厂商的SQL的差异。

## 介绍

### Hibernate中对象的介绍

- `SessionFactory(org.hibernate.SessionFactory)`

`SessionFactory`本身在Hibernate当中起到了缓冲区的作用,缓冲了Hibernate自动生成的SQL语句和他的映射语句，还缓冲了一些可能重复利用的数据。同时为了能够创建`SessionFactory`对象，在`Hibernate`初始化的时候创建一个`Configuration`类的实例并将已经写好的配置文件`hibernate.cfg.xml`交给其处理，这样`Configuration`就可以创建`SessionFactory`对象了，当`SessionFactory`创建完成后，`Configuration`对象就没有用了，可以将其抛弃。此外，`SessionFacory`还用到了工厂模式，使得程序能够从`SessionFactory`当中获得`Session`实例，能够在整个程序当中共享使用，通常一个项目当中只需要一个`SessionFactory`就够了。

```java
Session session=sessionfactory.openSession(); 
```

- `Session(org.hibernate.Session)`

上面说到了`SessionFactory`能够创建`Session`,处理之外还有什么方式可以或得到呢？
`sessionFactory.opensession()`一般的CRUD我们都是使用的这个操作，只有增删改需要使用事务，查询不需要，在查询时候食用事务，会将修改的数据提交更新数据库。
`sessionFactory.getCurrentSession()`如果存在多个拥有`session`的方法之间相互调用,这个时候就存在多个session对象和事务了，保证不了CRUD操作只在一个session对象和事务中操作，那么这个时候就需要使用这个方法获得session对象了。

```java
Session session=sessionfactory.getCurrentSession();  
```

注意：如果使用`sessionFactory.getCurrentSession()`需要注意以下内容：在hibernate的配置文件当中必须加上`<property name="current_session_context_class">thread</property>`这个配置。不需要写`session.close()`在事务提交的内部已经关闭。CRUD操作都需要事务，保证了业务操作的安全性，在整个方法当中一个session和一个事务。


## 配置Hibernate

### 加入jar包，编写 `hibernate.cfg.xml`文件

- 引入jar包

我在这里使用的是`hibernate4.3.11`的版本，下面是我的使用jar包的截图。

![images](http://osal57kgi.bkt.clouddn.com/hibernate_base_jar.png)

注意需要加入MySQL的驱动jar包。

- 编写`hibernate.cfg.xml`配置文件

多的不说，先将配置文件放上来 ，再来解释解释吧！

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>

    <session-factory>
        <!--数据库连接地址-->
        <property name="connection.url">jdbc:mysql://localhost:3306/java_base?useUnicode=true&amp;characterEncoding=UTF-8</property>
        <!--数据库用户名-->
        <property name="connection.username">chencong</property>
        <!--数据库密码-->
        <property name="connection.password">123456</property>
        <!--数据库连接驱动-->
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
        <!--数据库方言，指定使用的是哪一种数据库-->
        <property name="dialect">org.hibernate.dialect.MySQL5InnoDBDialect</property>
        <!--<property name="hbm2ddl.auto"></property>-->
        <!--执行CRUD操作控制台打印SQL语句-->
        <property name="show_sql">true</property>
        <!--将打印的SQL语句格式化输出-->
        <property name="format_sql">true</property>


        <!--实体类映射文件的类-->
        <mapping class="cc.ccoder.hibernatebase.pojo.Student"/>
    </session-factory>
</hibernate-configuration>
```

这里是最基本的配置，数据库相关配置，控制台格式化打印SQL语句，实体类映射，在这里使用的是注解，就不需要为每一个实体类配置一个xml文件了。

### 编写实体类

 在上面`hibernate.cfg.xml`的配置文件当中有一个`mapping`的映射配置，现在我们就编写上面说到的那个实体类`Student`。
 
 ```java
 /**
 * @author : ChenCong
 * @date : Created in 9:03 2017/12/27
 */
@Entity
@Table(name = "student")
public class Student implements Serializable {

    @Id  // 该字段为主键
    @GenericGenerator(name="uuidGen" , strategy = "uuid")  // 主键生成策略，此处为UUID
    @GeneratedValue(generator = "uuidGen")
    @Column(name = "id")  // 该属性对应表字段为id
    private String id;

    @Column(name="name")  // 该属性对应字段为name
    private String name;

    @Column(name = "age")
    private Integer age;

    @Column(name = "loginName")
    private String loginName;

    @Column(name = "pwd")
    private String pwd;
    // 省略掉getter setter方法
}
 ```

 如上面配置所示，`@Entity`表示当中为一个实体类，对应映射数据库表为`@Table(name = "student")`中的`student`表，主键自增，配置相应生成策略，该实体类当中属性和表中对应字段关联。

好了，到目前为止，所有的配置都已经完成。下面我们就来编写一个`Main`主方法来使用`Hibernate`。

### 使用Hibernate

新建一个`Main`主方法来获得`Configuration`对象，然后获得`SessionFactory`,通过`SessionFactory`获得我们所需的`session`对象，然后通过这个`session`对象来对数据进行CRUD的操作。

```java
/**
 * @author : ChenCong
 * @date : Created in 9:05 2017/12/27
 */
public class Main {
    public static void main(String[] args) {
        // 加载配置
        Configuration config = new Configuration().configure();
        // 创建会话工厂
        ServiceRegistry registry = new StandardServiceRegistryBuilder().applySettings(config.getProperties()).build();
        //获得sessionFactory
        SessionFactory factory = config.buildSessionFactory(registry);
        //开启session
        Session session = factory.openSession();
        System.out.println(session);
    }
}
```

注意此处的`config.getProperties()`是用来加载上面配置的`hibernate.cfg.xml`文件，配置文件放在src根目录下，或者在`IDEA`中新建`resources`文件夹使其`Make Directory as`成为`Resources Root`根目录。

运行上面的程序,我们将获得一个`session`对象。

```java
SessionImpl(PersistenceContext[entityKeys=[],collectionKeys=[]];
ActionQueue[insertions=org.hibernate.engine.spi.ExecutableList@69504ae9 
updates=org.hibernate.engine.spi.ExecutableList@387a8303 
deletions=org.hibernate.engine.spi.ExecutableList@28cda624 
orphanRemovals=org.hibernate.engine.spi.ExecutableList@1500b2f3 
collectionCreations=org.hibernate.engine.spi.ExecutableList@7eecb5b8 
collectionRemovals=org.hibernate.engine.spi.ExecutableList@126253fd 
collectionUpdates=org.hibernate.engine.spi.ExecutableList@57db2b13 
collectionQueuedOps=org.hibernate.engine.spi.ExecutableList@475c9c31 
unresolvedInsertDependencies=UnresolvedEntityInsertActions[]])
```

此时都已经配置好了`hibernate`了，下面我们就来根据这个`session`来对数据库进行CRUD操作。


## CRUD操作

### 查询

#### HQL查询所有记录

在`hibernate`当中我们可以使用`HQL`语句来查询出我们需要的结果。

```java
/**
     * 根据HQL语句查询数据
     *
     * @param session session对象
     */
    private static void queryStudentByHQL(Session session) {
        Query query = session.createQuery("from Student");
        List<Student> result = query.list();
        // 打印结果
        for (Student student : result) {
            System.out.println(student);
        }
    }
```

对上面的`Query query = session.createQuery("from Student");`语句进行说明：`from Student`的这个`Student`为实体类的类名，而不是表明，使用HQL语句是对实体类进行操作，而不是对数据库表进行操作，这就是对象关系映射(ORM).

查询结果如下:

```java
Hibernate: 
    select
        student0_.id as id1_0_,
        student0_.age as age2_0_,
        student0_.loginName as loginNam3_0_,
        student0_.name as name4_0_,
        student0_.pwd as pwd5_0_ 
    from
        student student0_
Student{id='04fe838e6095a548016095a54b3c0000', name='小红', age=22, loginName='xiaohong', pwd='123456'}
Student{id='67f4e72d46c146978dcf948e0dbb68c9', name='小王', age=12, loginName='xiaowang', pwd='123456'}
Student{id='f7c327ab284d4f579872ed2dc71e28cf', name='小明', age=12, loginName='xiaoming', pwd='123456'}
```

第一部分为`hibernate.cfg.xml`当中配置的`show_sql`和`format_sql`，然后第二部分就是我们使用HQL语句查询出来的student的信息(我在实体类当中重写了toString()方法，所以为这种输出格式)。

#### 根据表主键ID查询单条记录

很多情况下我们需要通过ID主键查询该表当中唯一的一条记录，那么就需要使用到session当中的`get()`方法。具体使用方法如下:

```java
/**
     * 根据ID主键查询，返回该实体类记录
     *
     * @param session session对象
     */
    private static void queryStudentById(Session session) {
        // 这里的get方法第一个参数为返回实体类的类类型，第二个参数为表中该条记录的主键
        Student student = (Student) session.get(Student.class, "04fe838e6095a548016095a54b3c0000");
        System.out.println(student);
    }
```

控制台当中显示的结果如下：

```java
Hibernate: 
    select
        student0_.id as id1_0_0_,
        student0_.age as age2_0_0_,
        student0_.loginName as loginNam3_0_0_,
        student0_.name as name4_0_0_,
        student0_.pwd as pwd5_0_0_ 
    from
        student student0_ 
    where
        student0_.id=?
Student{id='04fe838e6095a548016095a54b3c0000', name='小红', age=22, loginName='xiaohong', pwd='123456'}
```

#### SQL语句查询——查询字段和实体类完全匹配

上面第一个说到了使用HQL语句查询记录，但是很多情况下我们还是会使用SQL语句查询的，这样的场景`hibernate`还是支持的。但是有一点需要注意，使用SQL语句查询时候如果查询字段和实体类完全匹配，仍然可以返回实体类进行使用，如果不完全匹配，看下一个例子。

```java
/**
     * SQL语句查询结果字段和实体类完全匹配
     *
     * @param session session对象
     */
    private static void queryStudentBySQL(Session session) {
        SQLQuery sqlQuery = session.createSQLQuery("select * from student");
        sqlQuery.addEntity(Student.class);
        List<Student> list = sqlQuery.list();
        for (Student student1 : list) {
            System.out.println(student1);
        }
    }
```

上面`select * from student`查询的是表明而不是实体类名，同时返回的结果集当中字段和实体类完全匹配，就可以直接使用`addEntity()`方法将结果集放入实体类对象当中。控制台打印如下：（sql 语句为自己书写的SQL语句）

```java
Hibernate: 
    select
        * 
    from
        student
Student{id='04fe838e6095a548016095a54b3c0000', name='小红', age=22, loginName='xiaohong', pwd='123456'}
Student{id='67f4e72d46c146978dcf948e0dbb68c9', name='小王', age=12, loginName='xiaowang', pwd='123456'}
Student{id='f7c327ab284d4f579872ed2dc71e28cf', name='小明', age=12, loginName='xiaoming', pwd='123456'}
```

#### SQL语句查询——查询字段和实体类不完全匹配

上面的例子说了，SQL语句查询的结果集和实体类完全匹配的情况，但是很多时候，我们查询只需要我们需要的字段，而不需要所有的字段，就不能使用上面那种方式了，不能讲查询结果集放入实体类了，返回的结果是对象集合`Object[]`了。

```java
 /**
     * SQL语句查询结果字段和实体类不匹配
     *
     * @param session session对象
     */
    private static void queryStudentByArr(Session session) {
        SQLQuery sqlQuery = session.createSQLQuery("select id,name from student");
        List<Object[]> list = sqlQuery.list();
        for (Object[] objects : list) {
            System.out.println(objects[0] + "  " + objects[1]);
        }
    }
```

如上面所示返回的结果是一个`Object[]`数组。

```java
Hibernate: 
    select
        id,
        name 
    from
        student
04fe838e6095a548016095a54b3c0000  小红
67f4e72d46c146978dcf948e0dbb68c9  小王
f7c327ab284d4f579872ed2dc71e28cf  小明
```

#### HQL查询时候使用事务

通常情况下使用查询时候不需要事务的，但是在查询的时候使用事务会将查询后修改的数据同步到数据库。

```java
    /**
     * 有事务的情况下查询数据,修改之后，如果存在事务提交会将数据全部提交更新数据
     *
     * @param session session 对象
     */
    private static void queryStudentOnTransaction(Session session) {
        session.getTransaction().begin();
        Query query = session.createQuery("from Student");
        List<Student> students = query.list();
        for (Student student : students) {
            student.setPwd("123456");
        }
        session.getTransaction().commit();
    }
```
执行上面的程序，在数据库当猴子那个所有记录的`pwd`字段的值都被修改成`123456`，同时同步到数据库当中。通过控制台输出的SQL语句可以看出，这段程序执行了查询后，又执行了更新操作。

```java
Hibernate: 
    select
        student0_.id as id1_0_,
        student0_.age as age2_0_,
        student0_.loginName as loginNam3_0_,
        student0_.name as name4_0_,
        student0_.pwd as pwd5_0_ 
    from
        student student0_
Hibernate: 
    update
        student 
    set
        age=?,
        loginName=?,
        name=?,
        pwd=? 
    where
        id=?
Hibernate: 
    update
        student 
    set
        age=?,
        loginName=?,
        name=?,
        pwd=? 
    where
        id=?
Hibernate: 
    update
        student 
    set
        age=?,
        loginName=?,
        name=?,
        pwd=? 
    where
        id=?

```

#### HQL查询时使用参数

通常情况下使用查询语句时候，都是需要使用参数的。有两种方式进行操作：

- 使用 `?` 作为占位符

```java
    /**
     * 带参数的查询语句
     *
     * @param session session对象
     */
    private static void queryStudentByParam(Session session) {
        Query query = session.createQuery("from Student where loginName = ?");
        query.setString(0, "xiaohong");
        List<Student> students = query.list();
        for (Student student : students) {
            System.out.println(student);
        }
    }
```

注意`hibernate`当中是从 `0` 开始的，跟jdbc不同。直接结果为查询`loginName = xiaohong`的学生信息。

```java
Hibernate: 
    select
        student0_.id as id1_0_,
        student0_.age as age2_0_,
        student0_.loginName as loginNam3_0_,
        student0_.name as name4_0_,
        student0_.pwd as pwd5_0_ 
    from
        student student0_ 
    where
        student0_.loginName=?
Student{id='04fe838e6095a548016095a54b3c0000', name='小红', age=22, loginName='xiaohong', pwd='123456'}
```

- 使用`变量` 作为占位符。

```java
/**
     * 带参数的查询语句
     *
     * @param session session对象
     */
    private static void queryStudentByParam(Session session) {
        Query query = session.createQuery("from Student where loginName = :name");
        query.setString("name", "xiaohong");
        List<Student> students = query.list();
        for (Student student : students) {
            System.out.println(student);
        }
    }
```

此时使用的是`loginName = :name`这样一个`name`变量占位符，下面的`query.setString("name", "xiaohong");`当中同样使用`name`这个变量。注意使用变量时候需要在前面添加一个`:`.

结果如上不变。

### 新增

由于之前我们配置实体类当中生命了主键的生成策略，因此在这里使用时候不需要设置主键。同时进行`新增`操作时候需要使用事务。

```java
    /**
     * 保存实体信息
     * 需要开启事务 可以通过打印的sql语句来查看
     *
     * @param session session对象
     */
    private static void saveStudent(Session session) {
        Student student = new Student();
        student.setName("聪聪");
        student.setAge(22);
        student.setLoginName("ccoder");
        student.setPwd("123456");
        session.getTransaction().begin();
        session.save(student);  // 进行save()操作提交时候需要在事务当中进行
        session.getTransaction().commit();
    }
```

通过控制台打印日志新增数据成功。

```java
Hibernate: 
    insert 
    into
        student
        (age, loginName, name, pwd, id) 
    values
        (?, ?, ?, ?, ?)
```

### 修改

进行修改操作时候，可以先将数据查询出来，然后进行修改。这是使用`update(Object o)`时候的操作，数据已经存在于数据库当中。还可以使用`saveOrUpdate(Object o)` 当数据不存在时候进行save操作，存在时进行update操作。
- 使用对象进行操作

```java
    /**
     * 更新信息  同样需要打开事务
     *
     * @param session session对象
     */
    private static void updateStudent(Session session) {
        // 使用对象进行更新
        session.getTransaction().begin();
        Student student = (Student) session.get(Student.class, "04fe838e6096565a016096565d480000");
        student.setName("聪聪不匆匆");
        session.saveOrUpdate(student);
        session.getTransaction().commit();
    }
```

上述操作先进行查询`get()`,然后再进行`saveOrUpdate()`当中的`update()`操作。从控制台日志查看SQL记录。

- 使用HQL进行操作

``` java
    /**
     * 更新信息  同样需要打开事务
     *
     * @param session session对象
     */
    private static void updateStudent(Session session) {
        session.getTransaction().begin();
        session.createQuery("update Student set loginName = 'chencong' where id = '04fe838e6096565a016096565d480000' ").executeUpdate();
        session.getTransaction().commit();
    }
```

### 删除

删除操作，直接通过主键进行删除。同时提交事务。

```java
    /**
     * 删除
     *
     * @param session session对象
     */
    private static void deleteStudent(Session session) {
        session.getTransaction().begin();
        Student student = (Student) session.get(Student.class, "04fe838e6096565a016096565d480000");
        session.delete(student);
        session.getTransaction().commit();
    }
```
上述代码块，先通过主键查询到对象，然后再进行删除。控制台记录如下：

```java
Hibernate: 
    select
        student0_.id as id1_0_0_,
        student0_.age as age2_0_0_,
        student0_.loginName as loginNam3_0_0_,
        student0_.name as name4_0_0_,
        student0_.pwd as pwd5_0_0_ 
    from
        student student0_ 
    where
        student0_.id=?
Hibernate: 
    delete 
    from
        student 
    where
        id=?
```

## 后序

通过上面的配置和学习，了解了`hibernate`的基本配置和使用。下一节将总结一下`hibernate`当中的`分页、级联操作、关联查询`等问题。同时从上面的操作中看出`session`对象伴随着`CRUD`操作，那么之后也会说明`hibernate`当中对象的各种状态、生命周期以及它们之间是如何转换的。

学习之路，任重道远，加油。

## 联系

[聪聪](https://ccoder.cc/)的独立博客 ，一个喜欢技术，喜欢钻研的95后。如果你看到这篇文章，千里之外，我在等你联系。

- [Blog@ccoder's blog](https://ccoder.cc/)
- [CSDN@ccoder](http://blog.csdn.net/chencong3139)
- [Github@ccoder](https://github.com/chencong-plan)
- [Email@ccoder](mailto:admin@ccoder.top) *or* [Gmail@ccoder](mailto:chencong3139@gmail.com)
