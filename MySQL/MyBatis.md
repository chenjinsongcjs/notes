# MyBatis

# 一、MyBatis简介

-   MyBatis是支持定制化SQL，存储过程以及高级映射的持久层框架
-   MyBatis避免了几乎所有的JDBC代码和手动设置参数以及获取结果集
-   MyBatis可使用简单的XML或注解用于配置和原始映射，将接口和java 的POJO映射成数据库中的记录

## 1、为什么要用MyBatis

-   MyBatis是一个半自动化的持久层框架
-   JDBC
    -   SQL加载java代码后，耦合度高导致编码内伤
    -   维护不易且实际开发需求中sql有变化，频繁修改情况多见
-   Hibernate和JPA
    -   长难复杂SQL、对于Hibernate而言处理也不容易
    -   内部自动生产的SQL，不容易做特殊优化
    -   基于全面映射的自动框架，大量的POJO进行部分映射时比较困难。导致数据库性能下降。
-   对开发人员而言，核心sql还是需要自己优化
-   SQL和java编码分开，功能边界清晰，一个专注业务、一个专注数据。
-   [MyBatis官网](https://mybatis.org/mybatis-3/zh/index.html "MyBatis官网")

# 二、MyBatis-HelloWorld

## 1、建立数据库与Java对象之间的关系(ORM)

## 2、创建MyBatis的配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <!-- 指定sql映射文件的地址-->
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

## 3、创建MyBatis的SQL映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```

## 4、测试

### 4.1、根据全局配置文件，利用SqlSessionFactoryBuilder创建SqlSessionFactory

### 4.2、使用SqlSessionFactory获取SqlSession对象。一个SqlSession对象代表和数据库的一次会话。

### 4.3、使用SqlSession根据方法id进行操作

```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

SqlSession openSession = sqlSessionFactory.openSession();
try {
  Employee employee = openSession.selectOne(
          "com.atguigu.mybatis.EmployeeMapper.selectEmp", 1);
  System.out.println(employee);
  } finally {
      openSession.close();
}
```

### 4.4、接口式编程

-   创建Dao接口
-   修改mapper文件
-   测试

```java
// 1、获取sqlSessionFactory对象
    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
    // 2、获取sqlSession对象
    SqlSession openSession = sqlSessionFactory.openSession();
    try {
      // 3、获取接口的实现类对象
      //会为接口自动的创建一个代理对象，代理对象去执行增删改查方法
      EmployeeMapper mapper = openSession.getMapper(EmployeeMapper.class);
      Employee employee = mapper.getEmpById(1);
      System.out.println(mapper.getClass());
      System.out.println(employee);
    } finally {
      openSession.close();
    }
```

-   **注：**
    -   SqlSession是线程不安全的，因此不能共享。
    -   SQLSession每次用完都需要关闭。
    -   SqlSession可以直接调用方法的id进行数据库操 作，但是我们一般还是推荐使用SqlSession获取 到Dao接口的代理类，执行代理对象的方法，可 以更安全的进行类型检查操作

# 三、MyBatis-全局配置文件

-   MyBatis全局配置文件包含了影响MyBatis甚深的设置（settings）和属性（properties）信息。(MyBatis配置文件书写顺序要严格按照下面的顺序)
-   configuration（配置）
    -   [properties（属性）](https://mybatis.org/mybatis-3/zh/configuration.html#properties "properties（属性）")
    -   [settings（设置）](https://mybatis.org/mybatis-3/zh/configuration.html#settings "settings（设置）")
    -   [typeAliases（类型别名）](https://mybatis.org/mybatis-3/zh/configuration.html#typeAliases "typeAliases（类型别名）")
    -   [typeHandlers（类型处理器）](https://mybatis.org/mybatis-3/zh/configuration.html#typeHandlers "typeHandlers（类型处理器）")
    -   [objectFactory（对象工厂）](https://mybatis.org/mybatis-3/zh/configuration.html#objectFactory "objectFactory（对象工厂）")
    -   [plugins（插件）](https://mybatis.org/mybatis-3/zh/configuration.html#plugins "plugins（插件）")
    -   [environments（环境配置）](https://mybatis.org/mybatis-3/zh/configuration.html#environments "environments（环境配置）")
        -   environment（环境变量）
            -   transactionManager（事务管理器）
            -   dataSource（数据源）
    -   [databaseIdProvider（数据库厂商标识）](https://mybatis.org/mybatis-3/zh/configuration.html#databaseIdProvider "databaseIdProvider（数据库厂商标识）")
    -   [mappers（映射器）](https://mybatis.org/mybatis-3/zh/configuration.html#mappers "mappers（映射器）")

## 1、properties属性

-   读取外部配置文件
-   这些属性可以在外部进行配置，并可以进行动态替换，你既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

如果一个属性在不只一个地方进行了配置，那么，MyBatis 将按照下面的顺序来加载：

-   首先读取在 properties 元素体内指定的属性。
-   然后根据 properties 元素中的 resource 属性读取类路径下属性文件，或根据 url 属性指定的路径读取属性文件，并覆盖之前读取过的同名属性。
-   最后读取作为方法参数传递的属性，并覆盖之前读取过的同名属性。

**通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的则是 properties 元素中指定的属性。**

## 2、设置（settings）

-   这是MyBatis中极为重要的调整设置，他们会改变MyBatis的运行时行为。
-   配置的默认值

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

| **设置名**                                                                                         | **描述**                                                                                                                                                                                                         | **有效值**                                    | **默认值**                                               |
| ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ | ----------------------------------------------------- |
| cacheEnabled                                                                                    | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。                                                                                                                                                                                   | true                                       | false                                                 |
| lazyLoadingEnabled                                                                              | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 fetchType 属性来覆盖该项的开关状态。                                                                                                                                              | true                                       | false                                                 |
| aggressiveLazyLoading                                                                           | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 lazyLoadTriggerMethods)。                                                                                                                                      | true                                       | false                                                 |
| multipleResultSetsEnabled                                                                       | 是否允许单个语句返回多结果集（需要数据库驱动支持）。                                                                                                                                                                                     | true                                       | false                                                 |
| useColumnLabel                                                                                  | 使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。                                                                                                                                                             | true                                       | false                                                 |
| useGeneratedKeys                                                                                | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。                                                                                                                            | true                                       | false                                                 |
| autoMappingBehavior                                                                             | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。                                                                                                              | NONE, PARTIAL, FULL                        | PARTIAL                                               |
| autoMappingUnknownColumnBehavior                                                                | 指定发现自动映射目标未知列（或未知属性类型）的行为。                                                                                                                                                                                     |                                            |                                                       |
| - NONE: 不做任何反应                                                                                  |                                                                                                                                                                                                                |                                            |                                                       |
| - WARNING: 输出警告日志（'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 WARN） |                                                                                                                                                                                                                |                                            |                                                       |
| - FAILING: 映射失败 (抛出 SqlSessionException)                                                        |                                                                                                                                                                                                                |                                            |                                                       |
| NONE, WARNING, FAILING                                                                          | NONE                                                                                                                                                                                                           |                                            |                                                       |
| defaultExecutorType                                                                             | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。                                                                                                                        | SIMPLE REUSE BATCH                         | SIMPLE                                                |
| defaultStatementTimeout                                                                         | 设置超时时间，它决定数据库驱动等待数据库响应的秒数。                                                                                                                                                                                     | 任意正整数                                      | 未设置 (null)                                            |
| defaultFetchSize                                                                                | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。                                                                                                                                                                 | 任意正整数                                      | 未设置 (null)                                            |
| defaultResultSetType                                                                            | 指定语句默认的滚动策略。（新增于 3.5.2）                                                                                                                                                                                        | FORWARD\_ONLY                              | SCROLL\_SENSITIVE                                     |
| safeRowBoundsEnabled                                                                            | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。                                                                                                                                                                    | true                                       | false                                                 |
| safeResultHandlerEnabled                                                                        | 是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。                                                                                                                                                             | true                                       | false                                                 |
| mapUnderscoreToCamelCase                                                                        | 是否开启驼峰命名自动映射，即从经典数据库列名 A\_COLUMN 映射到经典 Java 属性名 aColumn。                                                                                                                                                       | true                                       | false                                                 |
| localCacheScope                                                                                 | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。                                                                         | SESSION                                    | STATEMENT                                             |
| jdbcTypeForNull                                                                                 | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。                                                                                                               | JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。      | OTHER                                                 |
| lazyLoadTriggerMethods                                                                          | 指定对象的哪些方法触发一次延迟加载。                                                                                                                                                                                             | 用逗号分隔的方法列表。                                | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage                                                                        | 指定动态 SQL 生成使用的默认脚本语言。                                                                                                                                                                                          | 一个类型别名或全限定类名。                              | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler                                                                          | 指定 Enum 使用的默认 TypeHandler 。（新增于 3.4.5）                                                                                                                                                                         | 一个类型别名或全限定类名。                              | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls                                                                              | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。                                                                                   | true                                       | false                                                 |
| returnInstanceForEmptyRow                                                                       | 当返回行的所有列都是空时，MyBatis默认返回 null。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2）                                                                                                                   | true                                       | false                                                 |
| logPrefix                                                                                       | 指定 MyBatis 增加到日志名称的前缀。                                                                                                                                                                                         | 任何字符串                                      | 未设置                                                   |
| logImpl                                                                                         | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。                                                                                                                                                                                | SLF4J                                      | LOG4J                                                 |
| proxyFactory                                                                                    | 指定 Mybatis 创建可延迟加载对象所用到的代理工具。                                                                                                                                                                                  | CGLIB                                      | JAVASSIST                                             |
| vfsImpl                                                                                         | 指定 VFS 的实现                                                                                                                                                                                                     | 自定义 VFS 的实现的类全限定名，以逗号分隔。                   | 未设置                                                   |
| useActualParamName                                                                              | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 -parameters 选项。（新增于 3.4.1）                                                                                                                               | true                                       | false                                                 |
| configurationFactory                                                                            | 指定一个提供 Configuration 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为static Configuration getConfiguration() 的方法。（新增于 3.2.3）                                                                     | 一个类型别名或完全限定类名。                             | 未设置                                                   |
| shrinkWhitespacesInSql                                                                          | 从SQL中删除多余的空格字符。请注意，这也会影响SQL中的文字字符串。 (新增于 3.5.5)                                                                                                                                                                | true                                       | false                                                 |
| defaultSqlProviderType                                                                          | Specifies an sql provider class that holds provider method (Since 3.5.6). This class apply to the type(or value) attribute on sql provider annotation(e.g. @SelectProvider), when these attribute was omitted. | A type alias or fully qualified class name | Not set                                               |

## 3、typeAliases别名处理器

-   类型别名是为 Java 类型设置一个短的名字，可以方便我们 引用某个类。  （别名不区分大小写）

```xml
<typeAliases>
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
```

-   类很多的情况下，可以批量设置别名这个包下的每一个类 创建一个默认的别名，就是简单类名小写。

```xml
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
```

-   也可以使用@Alias注解为其指定一个别名  (当别名冲突的时候)

```java
@Alias("author")
public class Author {
    ...
}
```

-   下面是一些为常见的 Java 类型内建的类型别名 **。它们都是不区分大小写的，注意，为了应对原始类型的命名重复，采取了特殊的命名风格。**

| ​**别名**    | ​**映射的类型** |
| ---------- | ---------- |
| \_byte     | byte       |
| \_long     | long       |
| \_short    | short      |
| \_int      | int        |
| \_integer  | int        |
| \_double   | double     |
| \_float    | float      |
| \_boolean  | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

## 4、typeHandlers类型处理器

-   MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器。
-   **提示** 从 3.4.5 开始，MyBatis 默认支持 JSR-310（日期和时间 API） 。

| ​**类型处理器**                 | ​**Java 类型**                  | ​**JDBC 类型**                                 |
| -------------------------- | ----------------------------- | -------------------------------------------- |
| BooleanTypeHandler         | java.lang.Boolean, boolean    | 数据库兼容的 BOOLEAN                               |
| ByteTypeHandler            | java.lang.Byte, byte          | 数据库兼容的 NUMERIC 或 BYTE                        |
| ShortTypeHandler           | java.lang.Short, short        | 数据库兼容的 NUMERIC 或 SMALLINT                    |
| IntegerTypeHandler         | java.lang.Integer, int        | 数据库兼容的 NUMERIC 或 INTEGER                     |
| LongTypeHandler            | java.lang.Long, long          | 数据库兼容的 NUMERIC 或 BIGINT                      |
| FloatTypeHandler           | java.lang.Float, float        | 数据库兼容的 NUMERIC 或 FLOAT                       |
| DoubleTypeHandler          | java.lang.Double, double      | 数据库兼容的 NUMERIC 或 DOUBLE                      |
| BigDecimalTypeHandler      | java.math.BigDecimal          | 数据库兼容的 NUMERIC 或 DECIMAL                     |
| StringTypeHandler          | java.lang.String              | CHAR, VARCHAR                                |
| ClobReaderTypeHandler      | java.io.Reader                | -                                            |
| ClobTypeHandler            | java.lang.String              | CLOB, LONGVARCHAR                            |
| NStringTypeHandler         | java.lang.String              | NVARCHAR, NCHAR                              |
| NClobTypeHandler           | java.lang.String              | NCLOB                                        |
| BlobInputStreamTypeHandler | java.io.InputStream           | -                                            |
| ByteArrayTypeHandler       | byte\[]                       | 数据库兼容的字节流类型                                  |
| BlobTypeHandler            | byte\[]                       | BLOB, LONGVARBINARY                          |
| DateTypeHandler            | java.util.Date                | TIMESTAMP                                    |
| DateOnlyTypeHandler        | java.util.Date                | DATE                                         |
| TimeOnlyTypeHandler        | java.util.Date                | TIME                                         |
| SqlTimestampTypeHandler    | java.sql.Timestamp            | TIMESTAMP                                    |
| SqlDateTypeHandler         | java.sql.Date                 | DATE                                         |
| SqlTimeTypeHandler         | java.sql.Time                 | TIME                                         |
| ObjectTypeHandler          | Any                           | OTHER 或未指定类型                                 |
| EnumTypeHandler            | Enumeration Type              | VARCHAR 或任何兼容的字符串类型，用来存储枚举的名称（而不是索引序数值）      |
| EnumOrdinalTypeHandler     | Enumeration Type              | 任何兼容的 NUMERIC 或 DOUBLE 类型，用来存储枚举的序数值（而不是名称）。 |
| SqlxmlTypeHandler          | java.lang.String              | SQLXML                                       |
| InstantTypeHandler         | java.time.Instant             | TIMESTAMP                                    |
| LocalDateTimeTypeHandler   | java.time.LocalDateTime       | TIMESTAMP                                    |
| LocalDateTypeHandler       | java.time.LocalDate           | DATE                                         |
| LocalTimeTypeHandler       | java.time.LocalTime           | TIME                                         |
| OffsetDateTimeTypeHandler  | java.time.OffsetDateTime      | TIMESTAMP                                    |
| OffsetTimeTypeHandler      | java.time.OffsetTime          | TIME                                         |
| ZonedDateTimeTypeHandler   | java.time.ZonedDateTime       | TIMESTAMP                                    |
| YearTypeHandler            | java.time.Year                | INTEGER                                      |
| MonthTypeHandler           | java.time.Month               | INTEGER                                      |
| YearMonthTypeHandler       | java.time.YearMonth           | VARCHAR 或 LONGVARCHAR                        |
| JapaneseDateTypeHandler    | java.time.chrono.JapaneseDate | DATE                                         |

-   时间类型的处理
    -   日期和时间的处理，JDK1.8以前一直是个头疼的 问题。我们通常使用JSR310规范领导者Stephen Colebourne创建的Joda-Time来操作。1.8已经实 现全部的JSR310规范了。
    -   日期时间处理上，我们可以使用MyBatis基于 JSR310（Date and Time API）编写的各种日期 时间类型处理器。 •
    -   MyBatis3.4以前的版本需要我们手动注册这些处 理器，以后的版本都是自动注册的
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630243682302-8ac8c2fd-02ee-4713-8b47-6acae31cfaca.png#clientId=u1c3513af-db7b-4\&from=paste\&height=233\&id=u63447d2f\&margin=\[object Object]\&name=image.png\&originHeight=466\&originWidth=1029\&originalType=binary\&ratio=1\&size=283182\&status=done\&style=none\&taskId=u45d07b9c-f071-4379-a506-b18abfe719e\&width=514.5>)
-   自定义类型处理器
    -   你可以重写已有的类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。 具体做法为：实现 org.apache.ibatis.type.TypeHandler 接口， 或继承一个很便利的类 org.apache.ibatis.type.BaseTypeHandler， 并且可以（可选地）将它映射到一个 JDBC 类型。

	```java
@MappedJdbcTypes(JdbcType.VARCHAR)
public class ExampleTypeHandler extends BaseTypeHandler<String> {

  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
  }

  @Override
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return rs.getString(columnName);
  }

  @Override
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    return rs.getString(columnIndex);
  }

  @Override
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    return cs.getString(columnIndex);
  }
}
```

```xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```

## 5、plugins插件

-   插件是MyBatis提供的一个非常强大的机制，我们 可以通过插件来修改MyBatis的一些核心行为。插 件通过动态代理机制，可以介入四大对象的任何 一个方法的执行。后面会有专门的章节我们来介 绍mybatis运行原理以及插件
-   MyBatis 允许你在映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：
    -   Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
    -   ParameterHandler (getParameterObject, setParameters)
    -   ResultSetHandler (handleResultSets, handleOutputParameters)
    -   StatementHandler (prepare, parameterize, batch, update, query)

## 6、environments环境

-   MyBatis可以配置多种环境，比如开发、测试和生产环境需要有不同的配置。
-   每种环境使用一个environment标签进行配置并指定唯一标识
-   可以通过environments标签中的default属性指定一个环境标识来快速切换环境。

## 7、environment指定具体环境

-   id：指定当前环境的唯一标识
-   transactionManager和dataSource是必须有的

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

-   transactionManager
    -   type ： JDBC或者MANAGED或者自定义
        -   **JDBC**： 使用了 JDBC 的提交和回滚设置，依赖于从数 据源得到的连接来管理事务范围。 JdbcTransactionFactory
        -   \*\* MANAGED\*\*：不提交或回滚一个连接、让容器来管理 事务的整个生命周期（比如 JEE 应用服务器的上下 文）ManagedTransactionFactory
        -   **自定义**： 实现TransactionFactory接口，type=全类名/ 别名
-   dataSource
    -   type：UNPOOLED或者POOLED或者JNDI或者自定义
        -   UNPOOLED：不使用连接池， UnpooledDataSourceFactory
        -   POOLED：使用连接池， PooledDataSourceFactory
        -   JNDI： 在EJB 或应用服务器这类容器中查找指定的数 据源
        -   自定义：实现DataSourceFactory接口，定义数据源的 获取方式。
    -   \*\*实际开发中我们使用Spring管理数据源，并进行 事务控制的配置来覆盖上述配置 \*\*

## 8、databaseidProvider环境

-   MyBatis可以根据不同的数据库厂商执行不同的语句

```xml
<databaseIdProvider type="DB_VENDOR">
  <property name="SQL Server" value="sqlserver"/>
  <property name="DB2" value="db2"/>
  <property name="Oracle" value="oracle" />
</databaseIdProvider>
```

-   Type： DB\_VENDOR
    -   使用MyBatis提供的VendorDatabaseIdProvider解析数据库 厂商标识。也可以实现DatabaseIdProvider接口来自定义。
-   Property-name：数据库厂商标识
-   Property-value：为标识起一个别名，方便SQL语句使用 databaseId属性引用
-   DB\_VENDOR
    -   会通过 DatabaseMetaData#getDatabaseProductName() 返回的字符 串进行设置。由于通常情况下这个字符串都非常长而且相同产品的不 同版本会返回不同的值，所以最好通过设置属性别名来使其变短
-   **MyBatis匹配规则如下：**
    -   如果没有配置databaseIdProvider标签，那么databaseId=null
    -   如果配置了databaseIdProvider标签，使用标签配置的name去匹 配数据库信息，匹配上设置databaseId=配置指定的值，否则依旧为 null
    -   如果databaseId不为null，他只会找到配置databaseId的sql语句
    -   MyBatis 会加载不带 databaseId 属性和带有匹配当前数据库 databaseId 属性的所有语句。如果同时找到带有 databaseId 和不带 databaseId 的相同语句，则后者会被舍弃

## 9、mapper映射文件

-   mapper逐个注册SQL映射文件
-   或者使用批量注册：
    -   这种方式要求SQL映射文件名必须和接口名相同并且在同一目录下

	```xml
<mappers>
  <!-- 使用相对于类路径的资源引用 -->
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <!-- 使用完全限定资源定位符（URL） -->
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <!-- 使用映射器接口实现类的完全限定类名 -->
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

# 四、MyBatis映射文件

-   映射文件指导着MyBatis如何进行数据库增删改查。有着非常重要的意义
-   映射文件中有很少的顶级元素（按照应该被声明的顺序列出）
    -   cache-命名空间的二级缓存配置
    -   cache-ref其他命名空间缓存配置的引用
    -   resultMap-自定义结果集映射
    -   sql-抽取可重用语句块
    -   insert-映射插入语句
    -   update-映射更新语句
    -   delete-映射删除语句
    -   select-映射查询语句

## 1、insert、update、delete元素

```sql
<insert
  id="insertAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  keyProperty=""
  keyColumn=""
  useGeneratedKeys=""
  timeout="20">

<update
  id="updateAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">

<delete
  id="deleteAuthor"
  parameterType="domain.blog.Author"
  flushCache="true"
  statementType="PREPARED"
  timeout="20">
```

| 属性               | 描述                                                                                                                                           |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| id               | 在命名空间中唯一的标识符，可以被用来引用这条语句。                                                                                                                    |
| parameterType    | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。                                                  |
|                  |                                                                                                                                              |
| flushCache       | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。                                                                 |
| timeout          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。                                                                                     |
| statementType    | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。                              |
| useGeneratedKeys | （仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。       |
| keyProperty      | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（unset）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| keyColumn        | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。                                     |
| databaseId       | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。                                       |

## 2、主键生成方式

-   若数据库支持自动生成主键的字段，如：MySQL，SQL server，就可以设置：**useGenerateKeys="true"**,然后**keyProperty**设置到目标属性。

```xml
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```

-   而对于不支持自增型主键的数据库（例如 Oracle），\*\*则可以使用 selectKey 子元素： selectKey 元素将会首先运行，id 会被设置，然 后插入语句会被调用  \*\*

```xml
<insert id="insertAuthor">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into Author
    (id, username, password, email,bio, favourite_section)
  values
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
</insert>
```

-   selectKey 元素的属性

| keyProperty   | selectKey 语句结果应该被设置到的目标属性。如果生成列不止一个，可以用逗号分隔多个属性名称。                                                                                                  |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| keyColumn     | 返回结果集中生成列属性的列名。如果生成列不止一个，可以用逗号分隔多个属性名称。                                                                                                             |
| resultType    | 结果的类型。通常 MyBatis 可以推断出来，但是为了更加准确，写上也不会有什么问题。MyBatis 允许将任何简单类型用作主键的类型，包括字符串。如果生成列不止一个，则可以使用包含期望属性的 Object 或 Map。                                     |
| order         | 可以设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它首先会生成主键，设置 keyProperty 再执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 中的语句 - 这和 Oracle 数据库的行为相似，在插入语句内部可能有嵌入索引调用。 |
| statementType | 和前面一样，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 类型的映射语句，分别代表 Statement, PreparedStatement 和 CallableStatement 类型。                                    |

## 3、参数传递

-   单个参数
    -   可以接收基本数据类型，对象类型(直接使用属性)，集合类型（list、array、collection别名）的值，这种情况MyBatis可以直接使用这个参数，不需要经过任何处理。
-   多个参数
    -   任意多个参数，都会被MyBatis重新包装成一个Map传入。
    -   Map的key是param1、param2、0 、1。值就是参数的值
-   命名参数
    -   为参数使用@Param注解起一个名字，MyBatis就会将这些参数封装进map，key就是我们指定的名字。
-   POJO
    -   当这些参数属于我们业务POJO时，我们直接传递POJO
-   Map
    -   我们也可以将多个参数直接封装为Map，指定好key的名称就行。

## 4、参数处理

-   参数也可以支付ing一个特殊的数据类型

```xml
#{property,javaType=int,jdbcType=NUMERIC}
对于数值类型，还可以设置 numericScale 指定小数点后保留的位数。
#{height,javaType=double,jdbcType=NUMERIC,numericScale=2}
```

-   javaType通常可以从参数对象中确定
-   如果null被当做值来传递，对所有可能为空的列，jdbcType需要被设置
    -   也可以全局设置jdbcForNull=other（mysql）/ null(orcal)
-   对于数值类型还可以设置小数点后保留的位数
-   mode属性允许指定IN/OUT或者INOUT参数，如果参数为OUT或INOUT，参数对象属性的真实值就会被改变，就像在获取输出参数时所期望的那样。
-   **参数位置支持的属性**
    -   javaType、jdbcType、mode、numericScale、 resultMap、typeHandler、jdbcTypeName、~~expression~~
-   实际上通常被设置的是：可能为空的列名指定jdbcType
-   \#{key}:获取参数的值，预编译到SQL中。安全
-   \${key}:获取参数的值，拼接到SQL中，有SQL注入安全问题。一般用在原生的SQL语句的拼装中 如： ORDER BY \${name}，或者分库分表的字段拼接

## 5、返回值处理

-   基本类型或对象类型可以使用ResultrType指定，或者不用指定TypeHandler自动处理
-   返回集合，ResultType指定为集合中的对象类型，MyBatis自定封装为集合
-   返回Map：
    -   对于只有一条记录返回，key为字段名，value为字段的值
    -   对于有多个记录返回，可以使用@MapKey在方法上，自己指定返回的Map的key和value类型，MyBatis自动帮我们封装。

## 6、select元素

-   select元素来定义查询操作。
-   id：唯一标识，用来引用这条语句，需要和接口 的方法名相同。
-   parameterType：参数类型，可以不传，MyBatis会根据TypeHandler自动判断
-   resultType：返回值类型。别名或者全类名，如果返回的是集合，定义集合中元素的类型。不能和resultMap同时使用。
-   Select 元素的属性

| id            | 在命名空间中唯一的标识符，可以被用来引用这条语句。                                                                                       |
| ------------- | --------------------------------------------------------------------------------------------------------------- |
| parameterType | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。                     |
|               |                                                                                                                 |
| resultType    | 期望从这条语句中返回结果的类全限定名或别名。 注意，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。                 |
| resultMap     | 对外部 resultMap 的命名引用。结果映射是 MyBatis 最强大的特性，如果你对其理解透彻，许多复杂的映射问题都能迎刃而解。 resultType 和 resultMap 之间只能同时使用一个。          |
| flushCache    | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。                                                                |
| useCache      | 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。                                                        |
| timeout       | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。                                                        |
| fetchSize     | 这是一个给驱动的建议值，尝试让驱动程序每次批量返回的结果行数等于这个设置值。 默认值为未设置（unset）（依赖驱动）。                                                    |
| statementType | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| resultSetType | FORWARD\_ONLY，SCROLL\_SENSITIVE, SCROLL\_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖数据库驱动）。            |
| databaseId    | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。          |
| resultOrdered | 这个设置仅针对嵌套结果 select 语句：如果为 true，将会假设包含了嵌套结果集或是分组，当返回一个主结果行时，就不会产生对前面结果集的引用。 这就使得在获取嵌套结果集的时候不至于内存不够用。默认值：false。   |
| resultSets    | 这个设置仅适用于多结果集的情况。它将列出语句执行后返回的结果集并赋予每个结果集一个名称，多个名称之间以逗号分隔。                                                        |

## 7、自动映射

-   全局setting设置
    -   autoMappingBehavior默认是PARTIAL，开启自动映射的功能，唯一的要求是列名和javaBean属性名一致。
    -   如果autoMappingBehavior设置为null则会取消自动映射
    -   数据库字段命名规范，POJO属性符合驼峰命名法，如A\_COLUM→aColum，我们可以开启驼峰命名规则映射功能， **mapUnderscoreToCamelCase=true。**
-   **自定义resultMap，实现高级结果集映射**

## 8、resultMap

-   constructor  - 类在实例化时，用来注入结果到构造方法中
    -   idArg;ID参数：标记结果作为ID可以帮助整体提高效能
    -   arg-注入到构造方法的一个普通结果
-   id-一个ID结果；标记结果为Id可以帮助提高整体效能
-   result-注入到字段或JavaBean属性的普通结果
-   association-一个复杂的类型关联；许多结果将包成这种类型
    -   嵌入结果映射-结果映射自身的关联，或者参考一个
-   collection-复杂类型的集合
    -   嵌入结果映射
-   discriminator-使用结果集的值来决定使用哪一个结果集映射
    -   case-基于某些值的结果映射

### 8.1、id和result

-   id 和 result 映射一个单独列的值到简单数据类型 (字符串,整型,双精度浮点数,日期等)的属性或字段。

| property    | 映射到列结果的字段或属性。如果 JavaBean 有这个名字的属性（property），会先使用该属性。否则 MyBatis 将会寻找给定名称的字段（field）。 无论是哪一种情形，你都可以使用常见的点式分隔形式进行复杂属性导航。 比如，你可以这样映射一些简单的东西：“username”，或者映射到一些复杂的东西上：“address.street.number”。 |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| column      | 数据库中的列名，或者是列的别名。一般情况下，这和传递给 resultSet.getString(columnName) 方法的参数一样。                                                                                                                     |
| javaType    | 一个 Java 类的全限定名，或一个类型别名（关于内置的类型别名，可以参考上面的表格）。 如果你映射到一个 JavaBean，MyBatis 通常可以推断类型。然而，如果你映射到的是 HashMap，那么你应该明确地指定 javaType 来保证行为与期望的相一致。                                                    |
| jdbcType    | JDBC 类型，所支持的 JDBC 类型参见这个表格之后的“支持的 JDBC 类型”。 只需要在可能执行插入、更新和删除的且允许空值的列上指定 JDBC 类型。这是 JDBC 的要求而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可以为空值的列指定这个类型。                                             |
| typeHandler | 我们在前面讨论过默认的类型处理器。使用这个属性，你可以覆盖默认的类型处理器。 这个属性值是一个类型处理器实现类的全限定名，或者是类型别名。                                                                                                                    |

### 8.2、association

-   复杂对象映射
-   POJO中的属性可能是一个对象
-   我们可以是会用联合查询，并以级联属性的方式封装对象。
-
![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630374639227-2de15eb5-3922-403a-8783-c46d801180ec.png#clientId=u725fc5f1-2140-4\&from=paste\&height=100\&id=u3e7f3167\&margin=\[object Object]\&name=image.png\&originHeight=200\&originWidth=835\&originalType=binary\&ratio=1\&size=99869\&status=done\&style=none\&taskId=u32d57353-261a-428a-993b-32f02bc0198\&width=417.5>)

-   使用association标签定义对象的封装规则
-   ## **association-嵌套结果集**
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630374791981-76bfa01a-0c36-4859-a6b5-5945df70ec98.png#clientId=u725fc5f1-2140-4\&from=paste\&height=129\&id=ub80126a3\&margin=\[object Object]\&name=image.png\&originHeight=258\&originWidth=1020\&originalType=binary\&ratio=1\&size=131333\&status=done\&style=none\&taskId=uc953a7d0-46a2-49da-8dee-475df95a7ce\&width=510>)
-   **association-分段查询**
    -   **select：调用目标方法查询当前属性的值**
    -   **column：将指定列的值传入目标方法中**
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630374897968-d439e337-86ff-4912-9f78-c0b1d13bd053.png#clientId=u725fc5f1-2140-4\&from=paste\&height=127\&id=u1da68f7a\&margin=\[object Object]\&name=image.png\&originHeight=254\&originWidth=873\&originalType=binary\&ratio=1\&size=110834\&status=done\&style=none\&taskId=uafcb43c5-624d-4dcd-bf39-c7ad1ec68fd\&width=436.5>)
-   association -分段查询和延迟加载
    -   开启延迟加载和属性按需加载
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630375068274-00c65998-002b-40c0-b6f6-ef49bfa86dbc.png#clientId=u725fc5f1-2140-4\&from=paste\&height=62\&id=uec72b4e9\&margin=\[object Object]\&name=image.png\&originHeight=124\&originWidth=929\&originalType=binary\&ratio=1\&size=54809\&status=done\&style=none\&taskId=u3fb2d251-1a36-41be-92fe-601eb38a3eb\&width=464.5>)

## 9、collection-集合类型和嵌套结果集

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630375231066-9f6fb57b-e459-44ff-a900-ef9585b0a55b.png#clientId=u725fc5f1-2140-4\&from=paste\&height=135\&id=u33989e29\&margin=\[object Object]\&name=image.png\&originHeight=269\&originWidth=888\&originalType=binary\&ratio=1\&size=101790\&status=done\&style=none\&taskId=u80506772-5679-4748-bd72-aeae1ebb13d\&width=444>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630375240627-9e273e36-8c62-411e-a38c-f34017c6ec62.png#clientId=u725fc5f1-2140-4\&from=paste\&height=177\&id=u6086bc5d\&margin=\[object Object]\&name=image.png\&originHeight=354\&originWidth=1043\&originalType=binary\&ratio=1\&size=176489\&status=done\&style=none\&taskId=udf5b5bca-7f7d-4b35-9336-bcfe12828b7\&width=521.5>)

-   ## **collection-分步查询和延迟加载**
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630375291508-b7cec8c6-a3f1-4128-8027-f49e0dd5b717.png#clientId=u725fc5f1-2140-4\&from=paste\&height=126\&id=uc4842ae2\&margin=\[object Object]\&name=image.png\&originHeight=251\&originWidth=1009\&originalType=binary\&ratio=1\&size=118047\&status=done\&style=none\&taskId=u8e049c5f-19e5-4b40-bcd1-eaccd9f4bed\&width=504.5>)

## 10、多列封装map传递

-   分步查询的时候通过column指定，将对应的列的数据传递过去，我们有时需要传递多列数据。
-   ## 使用{key1=column1，key2=column2.。。。}的形式
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630375448430-98179473-d556-46ab-bb2c-51b346f69577.png#clientId=u725fc5f1-2140-4\&from=paste\&height=128\&id=u45db682d\&margin=\[object Object]\&name=image.png\&originHeight=256\&originWidth=1008\&originalType=binary\&ratio=1\&size=110475\&status=done\&style=none\&taskId=u186e6d52-2dce-46c5-ba70-a501892f074\&width=504>)
-   **association或者collection标签的fetchType=eager/lazy可以覆盖全局的延迟加载策略，指定立即加载（eager）或者延迟加载（lazy）**

# 五、MyBatis-动态SQL

-   动态SQL是MyBatis强大特性之一。极大的简化我们拼装SQL的操作
-   动态 SQL 元素和使用 JSTL 或其他类似基于 XML 的文本处 理器相似。
-   MyBatis采用功能强大的基于OGNL的表达式来简化操作。
    -   if
    -   choose(when,otherwise)
    -   trim(where,set)
    -   foreach

### 5.1、if

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630375952229-4baad119-9955-4d54-b1a9-605635118de6.png#clientId=u725fc5f1-2140-4\&from=paste\&height=188\&id=ubacb7a57\&margin=\[object Object]\&name=image.png\&originHeight=376\&originWidth=1111\&originalType=binary\&ratio=1\&size=132952\&status=done\&style=none\&taskId=u27245a51-8e72-45a8-bf1a-ec39c2db5b2\&width=555.5>)

-   当id为null的时候拼装SQL会出错
    -   可以在where之后添加 1=1 ，是的where和and之间永远有条件
    -   使用where标签，适合and写在条件签名的情况
    -   and写在条件的后面，可以使用trim标签覆盖调最后的a多余的and

### 5.2、choose（when，otherwise）

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630376198976-4db472d7-55a1-4ef9-ab74-7bd0354e3bac.png#clientId=u725fc5f1-2140-4\&from=paste\&height=250\&id=ufd4e8faf\&margin=\[object Object]\&name=image.png\&originHeight=500\&originWidth=1185\&originalType=binary\&ratio=1\&size=136834\&status=done\&style=none\&taskId=u18077bef-2f3e-4804-a88b-82c32328b5d\&width=592.5>)

-   相当于switch-case多选以的情况

### 5.3、trim（where，set）

-   ## where
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630376331539-d6d88149-8f10-4549-a1ca-1814c1fc85bc.png#clientId=u725fc5f1-2140-4\&from=paste\&height=215\&id=u52ad775b\&margin=\[object Object]\&name=image.png\&originHeight=430\&originWidth=1090\&originalType=binary\&ratio=1\&size=138265\&status=done\&style=none\&taskId=u7d7febf7-f6d2-4da7-9837-68edae46f83\&width=545>)
    -   第一个条件不用and
-   ## set
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630376458866-d4034e20-0829-4252-9028-0b80715e3916.png#clientId=u725fc5f1-2140-4\&from=paste\&height=235\&id=u1eda9d06\&margin=\[object Object]\&name=image.png\&originHeight=469\&originWidth=1124\&originalType=binary\&ratio=1\&size=138366\&status=done\&style=none\&taskId=u6aac1cf0-8bf1-4647-b73a-0d6e0e3f891\&width=562>)
-   ## trim
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630376527290-6d5f6e07-599c-447e-be68-6d51f8aea425.png#clientId=u725fc5f1-2140-4\&from=paste\&height=223\&id=u331a4905\&margin=\[object Object]\&name=image.png\&originHeight=446\&originWidth=1089\&originalType=binary\&ratio=1\&size=153704\&status=done\&style=none\&taskId=u9244a7b1-b4bd-4b2f-b753-9504f3bbb2e\&width=544.5>)

### 5.4、foreach

-   动态 SQL 的另外一个常用的必要操作是需要对一个集合 进行遍历，通常是在构建 IN 条件语句的时候。
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630376647146-a4e7c82f-3aac-42b1-88b1-05e7a71f9210.png#clientId=u725fc5f1-2140-4\&from=paste\&height=114\&id=uc8776671\&margin=\[object Object]\&name=image.png\&originHeight=228\&originWidth=1026\&originalType=binary\&ratio=1\&size=84291\&status=done\&style=none\&taskId=u728ef78a-bab6-422a-b04e-d828adfc7b3\&width=513>)

-   当迭代列表、集合等可迭代对象或者数组时
    -   index是当前迭代的次数，item的值是本次迭代获取的元素
-   当使用字典（或者Map。Entry对象的集合）时
    -   index是键，item是值

### 5.5、bind

-   bind元素可以从OGNL表达式中创建一个变量并将其绑定到上下文。比如
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630376933124-af7459e8-054b-40d9-9152-0419250842c3.png#clientId=u725fc5f1-2140-4\&from=paste\&height=77\&id=u0fe01393\&margin=\[object Object]\&name=image.png\&originHeight=154\&originWidth=1177\&originalType=binary\&ratio=1\&size=85785\&status=done\&style=none\&taskId=ub8a0451e-55b1-4b76-8311-a16da045029\&width=588.5>)

### 5.6、增删改查标签中两个常用的内置参数（\_parameter和\_databaseId）

-   \_parameter:代表传入参数的整体
    -   可以通过这个内置参数获取传入的参数
-   \_databaseId:代表数据库厂商唯一标识
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630377205501-e5bad4bf-a0db-4a2a-b326-ac81458686d7.png#clientId=u725fc5f1-2140-4\&from=paste\&height=120\&id=ucef8aa12\&margin=\[object Object]\&name=image.png\&originHeight=239\&originWidth=1142\&originalType=binary\&ratio=1\&size=88679\&status=done\&style=none\&taskId=u221f8cf2-4cde-4ef5-95e1-aea77d23d25\&width=571>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630377307559-ed9d00a9-1c3b-45ac-97b8-576626deff6d.png#clientId=u725fc5f1-2140-4\&from=paste\&height=356\&id=ud31d4f5e\&margin=\[object Object]\&name=image.png\&originHeight=712\&originWidth=1145\&originalType=binary\&ratio=1\&size=285739\&status=done\&style=none\&taskId=ucd3ee072-932b-429c-9bf8-a9099a0a28f\&width=572.5>)

# 六、MyBatis-缓存机制

-   MyBatis包含了一个非常强大的查询缓存特性，他是可以非常方便地配置和定制。缓存可以极大的提升查询效率。
-   MyBatis系统中默认定义了两级缓存
-   **一级缓存和二级缓存**
    -   **默认情况下，只有一级缓存（SQLSession级别的缓存，也叫本地缓存）开启。**
    -   **二级缓存需要手动开启和配置，它是基于namespace级别的缓存**
    -   **为了提高扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现cache接口来定义二级缓存。**

## 6.1、一级缓存

-   一级缓存，即本地缓存，作用域默认为sqlSession。当session flush或者close后，该session中的所有cache将被清空。
-   本地缓存不能被关闭，但是可以调用sqlSession的clearCache()方法清空本地缓存，或者改变缓存的作用域 （SqlSession关闭一级缓存进入二级缓存）
-   在mybatis3.1之后, 可以配置本地缓存的作用域. 在 mybatis.xml 中配置
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630378094871-93641e10-eef7-4eb9-8f7b-92294f3211c2.png#clientId=u725fc5f1-2140-4\&from=paste\&height=91\&id=ua5831b5e\&margin=\[object Object]\&name=image.png\&originHeight=181\&originWidth=1185\&originalType=binary\&ratio=1\&size=109989\&status=done\&style=none\&taskId=u54f47ad5-ce6d-42f8-ad38-e06e28e7fad\&width=592.5>)

-   **一级缓存失效情况**
    -   **同一次会话期间之只要查询过的数据都会保存在当前SqlSession的一个Map中**
        -   **key：** :hashCode+查询的SqlId+编写的sql查询语句+参数
    -   不同的sqlSession对应不同的一级缓存
    -   同一个sqlSession但是查询条件不同
    -   同一个sqlSession两次查询期间执行任何一次增删改操作（他们的flushCache=true）
    -   同一个sqlSession两次查询期间手动清空缓存。

## 6.2、二级缓存

-   二级缓存，全局作用域缓存
-   二级缓存默认不开启，需要手动配置
-   MyBatis提供二级缓存的解救一级实现，缓存实现要去POJO实现Serializable接口
-   **二级缓存在sqlSession关闭或者提交之后才会生效**
-   **使用步骤：**
    -   **全局配置文件中开启二级缓存**

	```xml
<setting name="cacheEnabled" value="true">
```

-   需要使用二级缓存的映射文件处使用cache标签配置缓存
-   注意：POJO需要实现Serializable接口
-   **缓存相关属性**
    -   **eviction：缓存回收策略：**
        -   **LRU：最近最少使用；移除最长时间不被使用的对象**
        -   **FIFO：先进先出；按照对象进入缓存的顺序来移除他们**
        -   **SOFT：软引用：移除基于垃圾回收器状态和软引用规则的对象**
        -   **WEAK: 弱引用：** 更积极地移除基于垃圾收集器状态和弱引用规则的对象。
        -   默认是：LRU
    -   flushInterval：刷新间隔，单位毫秒
        -   默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新
    -   size：引用数目，正整数
        -   代表缓存最多可以存储多少对象，太大容易导致内存溢出
    -   readOnly：只读，true/false
        -   true：只读缓存；会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势
        -   false；读写缓存：会返回缓存队形的拷贝（通过序列化）。这会慢一些，但是安全，因此默认是false；
-   **缓存有关设置**
    -   **全局setting的cacheEnable**
        -   **配置二级缓存的开关，一级缓存一直是打开的**
    -   select标签的useCache属性
        -   配置这个select是否使用二级缓存。一级缓存一直是使用的
    -   sql标签的flushCache属性
        -   增删改默认flushCache=true。sql执行以后，会同时清空一级和二级缓存。 查询默认flushCache=false。
    -   sqlSession.clearCache();
        -   只是用来清空一级缓存。
    -   当在某一个作用域 (一级缓存Session/二级缓存 Namespaces) 进行了 C/U/D 操作后，默认该作用域下所 有 select 中的缓存将被clear
-   **第三方缓存整合**
    -   EhCache 是一个纯Java的进程内缓存框架，具有快速、精 干等特点，是Hibernate中默认的CacheProvider。
    -   MyBatis定义了Cache接口方便我们进行自定义扩展。
    -   步骤：
        -   导入ehcache依赖
        -   编写ehcache.xml配置文件
        -   配置cache标签

	```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
```

-   **参照缓存**：若想在命名空间中共享相同的缓存配置和实例可以使用cache-ref元素来引用另外一个缓存
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630380164174-f9ccfaf0-7735-42be-98de-3adaab59f242.png#clientId=u725fc5f1-2140-4\&from=paste\&height=20\&id=ue0d43e07\&margin=\[object Object]\&name=image.png\&originHeight=39\&originWidth=1089\&originalType=binary\&ratio=1\&size=29751\&status=done\&style=none\&taskId=ubc319fd3-26f0-466a-a7d7-2e36b751b43\&width=544.5>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630380241264-3bc222a2-502c-4088-a833-f95c3d5573a4.png#clientId=u725fc5f1-2140-4\&from=paste\&height=381\&id=ub8d0750f\&margin=\[object Object]\&name=image.png\&originHeight=762\&originWidth=1154\&originalType=binary\&ratio=1\&size=543987\&status=done\&style=none\&taskId=ub600c68c-5c83-4cf8-bbce-5be3a51c05b\&width=577>)

# 七、MyBatis-Spring整合

-   [参考官方文档](http://mybatis.org/spring/zh/index.html "参考官方文档")

# 八、MyBatis-逆向工程

-   [参考官方文档](http://mybatis.org/generator/ "参考官方文档")

# 九、MyBatis-工作原理

![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630380937673-2f5a96bd-572e-4fb9-af94-af0746d4a4ca.png#clientId=u725fc5f1-2140-4\&from=paste\&height=487\&id=u0e7fc775\&margin=\[object Object]\&name=image.png\&originHeight=974\&originWidth=1724\&originalType=binary\&ratio=1\&size=1051261\&status=done\&style=none\&taskId=u3669d689-3cfc-4b46-9a33-da6e1435e45\&width=862>)

![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630381085891-853aed15-5746-4497-a72e-3201d303fd57.png#clientId=u725fc5f1-2140-4\&from=paste\&height=364\&id=u63eed7db\&margin=\[object Object]\&name=image.png\&originHeight=728\&originWidth=1009\&originalType=binary\&ratio=1\&size=177219\&status=done\&style=none\&taskId=u5f52c695-4201-47d6-874d-8294a6909b8\&width=504.5>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630381200518-4a8d05a2-ca99-4091-923d-36293ad23804.png#clientId=u725fc5f1-2140-4\&from=paste\&height=520\&id=ucfb34f7f\&margin=\[object Object]\&name=image.png\&originHeight=1040\&originWidth=1863\&originalType=binary\&ratio=1\&size=563886\&status=done\&style=none\&taskId=u32cf3ef0-2c14-4d64-91ca-ff50f0c0b5c\&width=931.5>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630381405200-93b7dde4-9019-4bf3-a752-4ccc26056b99.png#clientId=u725fc5f1-2140-4\&from=paste\&height=503\&id=u498bef8e\&margin=\[object Object]\&name=image.png\&originHeight=1006\&originWidth=1737\&originalType=binary\&ratio=1\&size=388993\&status=done\&style=none\&taskId=uc45402b8-a6fb-45bb-98d2-3702246edd0\&width=868.5>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630381429247-c6f2a107-caec-4b98-983c-220bbeb30a4a.png#clientId=u725fc5f1-2140-4\&from=paste\&height=508\&id=u1c891392\&margin=\[object Object]\&name=image.png\&originHeight=1016\&originWidth=1822\&originalType=binary\&ratio=1\&size=440374\&status=done\&style=none\&taskId=u49eb1473-ec8a-4f03-8301-02a98f7f55d\&width=911>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630381440500-379e4052-c729-4a4a-beee-a57b387596c5.png#clientId=u725fc5f1-2140-4\&from=paste\&id=ubccd05eb\&margin=\[object Object]\&name=image.png\&originHeight=720\&originWidth=855\&originalType=binary\&ratio=1\&size=209268\&status=done\&style=none\&taskId=ua593889a-2330-4f9b-93d5-a0ebf7cab1d>)

-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630381461043-4c35e19c-f927-451a-a548-bddd4579ea4d.png#clientId=u725fc5f1-2140-4\&from=paste\&height=526\&id=u2eb4ed5a\&margin=\[object Object]\&name=image.png\&originHeight=1051\&originWidth=1782\&originalType=binary\&ratio=1\&size=137133\&status=done\&style=none\&taskId=u2b85f346-28e1-4fbb-9bc8-5788e02f962\&width=891>)

```java
/**
   * 1、获取sqlSessionFactory对象:
   *     解析文件的每一个信息保存在Configuration中，返回包含Configuration的DefaultSqlSession；
   *     注意：【MappedStatement】：代表一个增删改查的详细信息
   * 
   * 2、获取sqlSession对象
   *     返回一个DefaultSQlSession对象，包含Executor和Configuration;
   *     这一步会创建Executor对象；
   * 
   * 3、获取接口的代理对象（MapperProxy）
   *     getMapper，使用MapperProxyFactory创建一个MapperProxy的代理对象
   *     代理对象里面包含了，DefaultSqlSession（Executor）
   * 4、执行增删改查方法
   * 
   * 总结：
   *   1、根据配置文件（全局，sql映射）初始化出Configuration对象
   *   2、创建一个DefaultSqlSession对象，
   *     他里面包含Configuration以及
   *     Executor（根据全局配置文件中的defaultExecutorType创建出对应的Executor）
   *  3、DefaultSqlSession.getMapper（）：拿到Mapper接口对应的MapperProxy；
   *  4、MapperProxy里面有（DefaultSqlSession）；
   *  5、执行增删改查方法：
   *      1）、调用DefaultSqlSession的增删改查（Executor）；
   *      2）、会创建一个StatementHandler对象。
   *        （同时也会创建出ParameterHandler和ResultSetHandler）
   *      3）、调用StatementHandler预编译参数以及设置参数值;
   *        使用ParameterHandler来给sql设置参数
   *      4）、调用StatementHandler的增删改查方法；
   *      5）、ResultSetHandler封装结果
   *  注意：
   *    四大对象每个创建的时候都有一个interceptorChain.pluginAll(parameterHandler);
   * 
     */
```

# 十、MyBatis插件开发

-   MyBatis在四大对象的创建过程中，都会有插件进行介入。插件可以利用动态代理机制一层层的包装目标对象，而实现在目标对象执行目标方法之前拦截的效果。
-   MyBatis允许在已映射语句执行过程中的某一点进行拦截调用。
-   默认情况下，MyBatis 允许使用插件来拦截的方法调 用包括：
    -   \*\*Executor \*\*(update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
    -   \*\*ParameterHandler \*\*(getParameterObject, setParameters)
    -   \*\*ResultSetHandler \*\*(handleResultSets, handleOutputParameters)
    -   \*\*StatementHandler \*\*(prepare, parameterize, batch, update, query

## 1、插件开发

-   插件开发步骤
    -   编写插件实现Interceptor接口，并使用@Intercepts注解 完成插件签名
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630382810614-ccd55faf-b464-47fc-bd5e-910af1fff3d0.png#clientId=u725fc5f1-2140-4\&from=paste\&height=95\&id=u91b2d843\&margin=\[object Object]\&name=image.png\&originHeight=190\&originWidth=1044\&originalType=binary\&ratio=1\&size=74581\&status=done\&style=none\&taskId=u2d780dcc-9c05-4535-88b7-4b6834ad6cb\&width=522>)
    -   在全局配置文件中注册插件
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630382865350-b73aa24a-b8cb-4124-8341-8abb6a41abe1.png#clientId=u725fc5f1-2140-4\&from=paste\&height=85\&id=u14e6d5e1\&margin=\[object Object]\&name=image.png\&originHeight=170\&originWidth=1017\&originalType=binary\&ratio=1\&size=68689\&status=done\&style=none\&taskId=ua398d331-94bf-4d5f-bcb8-2669d182f22\&width=508.5>)

## 2、插件原理

-   按照插件注解声明，按照插件配置顺序调用插件plugin方 法，生成被拦截对象的动态代理
-   多个插件依次生成目标对象的代理对象，层层包裹，先声 明的先包裹；形成代理链
-   目标方法执行时依次从外到内执行插件的intercept方法。 &#x20;

    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630383005900-1e99282d-d33d-499e-87fb-4df53c026cd5.png#clientId=u725fc5f1-2140-4\&from=paste\&height=270\&id=u22972fc2\&margin=\[object Object]\&name=image.png\&originHeight=540\&originWidth=1129\&originalType=binary\&ratio=1\&size=264618\&status=done\&style=none\&taskId=u351275a3-ae56-49f2-b082-c853775d97e\&width=564.5>)
-   多个插件情况下，我们往往需要在某个插件中分离出目标 对象。可以借助MyBatis提供的SystemMetaObject类来进行获 取最后一层的h以及target属性的值
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630383110209-feff384d-5200-467c-8807-fae16484fd19.png#clientId=u725fc5f1-2140-4\&from=paste\&height=512\&id=udc91e4cb\&margin=\[object Object]\&name=image.png\&originHeight=1024\&originWidth=1803\&originalType=binary\&ratio=1\&size=283899\&status=done\&style=none\&taskId=ue940c55c-467f-43b0-9fd0-462fc8c86c7\&width=901.5>)

-   **interceptor接口**
    -   **intercept：拦截目标方法执行**
    -   **plugin：生成动态代理对象，可以使用MyBatis提供的Plugin类的wrap方法**
    -   **setProperties：注入插件配置时设置属性**
-   常用代码：

![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630383405101-b899390e-4525-41a4-ab8a-11c54aeec6d6.png#clientId=u725fc5f1-2140-4\&from=paste\&height=333\&id=u75df1e99\&margin=\[object Object]\&name=image.png\&originHeight=666\&originWidth=1082\&originalType=binary\&ratio=1\&size=219522\&status=done\&style=none\&taskId=u2fd7b42e-53e4-4792-99ad-c8c369fe20e\&width=541>)

# 十一、扩展：MyBatis实用场景

-   PageHelper插件进行分页
-   批量操作
-   存储过程
-   typeHandler处理枚举

## 1、pageHelper

-   参考官网

## 2、批量操作

-   默认的openSession()方法没有参数，它会创建有如下特性的：
    -   会开启一个事务（不会自动提交）
    -   连接对象会从由活动环境配置的数据源实例得到。
    -   事务隔离级别将会使用驱动或数据源的默认设置。
    -   预处理语句不会被复用,也不会批量处理更新。
-   openSession方法的ExecutorType类型的参数，枚举类型：
    -   \*\* ExecutorType.SIMPLE\*\*: 这个执行器类型不做特殊的事情（这是默认装配 的）。它为每个语句的执行创建一个新的预处理语句。
    -   **ExecutorType.REUSE:** 这个执行器类型会复用预处理语句。
    -   **ExecutorType.BATCH**: 这个执行器会批量执行所有更新语句
    -
        ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630383879161-4b465f92-919a-4c33-b617-98a5273d17d6.png#clientId=u725fc5f1-2140-4\&from=paste\&height=103\&id=ua922d537\&margin=\[object Object]\&name=image.png\&originHeight=206\&originWidth=1024\&originalType=binary\&ratio=1\&size=124216\&status=done\&style=none\&taskId=u6663a4b3-e0f4-48b5-a877-cf824a39811\&width=512>)
-   批量操作我们是使用MyBatis提供的BatchExecutor进行的， 他的底层就是通过jdbc攒sql的方式进行的。我们可以让他 攒够一定数量后发给数据库一次。
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630383976978-781e9cb2-98f9-4f23-aa19-c773eda54f69.png#clientId=u725fc5f1-2140-4\&from=paste\&height=297\&id=u5ff77df4\&margin=\[object Object]\&name=image.png\&originHeight=593\&originWidth=1053\&originalType=binary\&ratio=1\&size=170757\&status=done\&style=none\&taskId=u50406d39-25d8-48ca-a9d5-1894d37270f\&width=526.5>)

-   与Spring整合中，我们推荐，额外的配置一个可以专 门用来执行批量操作的sqlSession
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630384016395-3cfe2005-ef7c-47a8-b52d-2adc273fb9df.png#clientId=u725fc5f1-2140-4\&from=paste\&height=58\&id=uc44211ed\&margin=\[object Object]\&name=image.png\&originHeight=116\&originWidth=1073\&originalType=binary\&ratio=1\&size=57775\&status=done\&style=none\&taskId=u284cef3f-2b85-42c4-a3ce-bd5d47043b8\&width=536.5>)

-   需要用到批量操作的时候，我们可以注入配置的这个批量 SqlSession。通过他获取到mapper映射器进行操作。
-   注意：
    -   1、批量操作是在session.commit()以后才发送sql语句给数 据库进行执行的
    -   2、如果我们想让其提前执行，以方便后续可能的查询操作 获取数据，我们可以使用sqlSession.flushStatements()方 法，让其直接冲刷到数据库进行执行

## 3、存储过程

-   实际开发中，我们通常也会写一些存储过程， MyBatis也支持对存储过程的调用
-   存储过程的调用
    -   1、select标签中statementType=“CALLABLE”
    -   2、标签体中调用语法：
        -   {call procedure\_name(#{param1\_info},#{param2\_info})}

## 4、 自定义TypeHandler处理枚举

-   我们可以通过自定义TypeHandler的形式来在设置参数或 者取出结果集的时候自定义参数封装策略。
-   步骤：
    -   1、实现TypeHandler接口或者继承BaseTypeHandler
    -   2、使用@MappedTypes定义处理的java类型 使用@MappedJdbcTypes定义jdbcType类型
    -   3、在自定义结果集标签或者参数处理的时候声明使用自定义 TypeHandler进行处理 或者在全局配置TypeHandler要处理的javaType
-
    ![](<https://cdn.nlark.com/yuque/0/2021/png/21698573/1630384270656-241bd8ed-7540-4813-9817-6c1c0694c360.png#clientId=u725fc5f1-2140-4\&from=paste\&height=415\&id=u76eabd0c\&margin=\[object Object]\&name=image.png\&originHeight=829\&originWidth=1018\&originalType=binary\&ratio=1\&size=270495\&status=done\&style=none\&taskId=uf69ba5ad-41a9-43b4-9fb5-c05a550fb75\&width=509>)
