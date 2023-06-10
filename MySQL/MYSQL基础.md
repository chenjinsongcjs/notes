# MYSQL基础

# 一、数据库的相关概念

1.  DB：数据库，存储数据的容器
2.  DBMS：数据库管理系统，又称数据库软件或数据库产品，用于创建和管理DB。
3.  SQL：结构化查询语言，用于和数据库通信的语言，不是某个数据库所特有的，而是几乎所有主流数据库软件通用的语言。

**数据库存储数据的特点 ：**

1.  数据存储到表中，然后表再放到库中
2.  一个库中可以有多张表，每张表具有与唯一的表明用来标识自己
3.  表中有一个或多个列，列又称为字段，相当于java中属性
4.  表中的每一行数据，相当于java中对象

**常见的数据库管理系统：**
mysql、oracle、db2、sqlServer
\*\*MySQL服务的登录和退出: \*\*
登录：mysql 【-h 主机名 -P 端口号】 -u 用户名 -p密码
退出：exit或ctrl+C

# 二、基础查询（DQL语言）

## 1、语法

```sql
select 查询列表
from 表名;

 `:着重号用于区分关键字和字段名。
```

## 2、特点

1.  查询列表可以是字段、常量、表达式、函数、也可以是多个
2.  查询结果是一个虚拟表

## 3、示例

1.  查询单个字段

```sql
select 字段名
from 表名
```

1.  查询多个字段

```sql
select 字段名、字段名
from 表名;
```

1.  查询所有字段

```sql
select * 
from 表名；
```

1.  查询常量值

```sql
select 常量；
```

注：字符型和日期型的常量值必须用单引号引起来，数值型不需要

1.  查询函数

```sql
select 函数名（实参列表）;
```

1.  查询表达式

```sql
select 100/1234 # 相当于执行函数
```

1.  起别名
    1.  在字段后面使用as关键字
    2.  在字段后面使用空格，隔开字段和别名

	 ```sql
SELECT salary AS "out put" FROM employees;  -- 有特殊符号时用引号包围
```

1.  去重

```sql
select distinct 字段名
from 表名；
```

1.  -   做加法运算

```sql
select 数值+数值；直接运算
select 字符+数值；先试图将字符转换为数值，如果转换成功，则继续运算；否则转换为0继续运算
select null+数值；结果都为null
```

1.  concat函数

```sql
# 拼接字符串
select concat(字符1，字符2，。。。)
```

1.  ifnull函数

```sql
[[判断某字段或表达式是否为null，如果为null，返回指定的值，否则返回原来的值。]]
select ifnull(commission_pct,0) from employees;
```

1.  isnull函数

判断某字段或表达式是否为null，如果是，则返回1，否则返回0

# 三、条件查询

## 1、语法

```sql
select 查询列表     ③
from 表名       ①
where 筛选条件     ②
```

## 2、筛选条件的分类

1.  简单条件运算符

> < =  <> != >=   <=    <=>安全等于

1.  逻辑运算符

| &&   | and |   |
| ---- | --- | - |
| \|\| | or  |   |
| !    | not |   |

1.  模糊查询

`like`：一般搭配通配符使用，可以判断字符型或数值型
通配符：`% `匹配任意多个字符，`_` 匹配任意单个字符
当通配符作为一个普通字符的时候要进行转义：

```sql
[[转义字符的使用，当通配符]] 作为一个普通的字符使用时要进行转义
SELECT
  *
FROM 
  employees
WHERE  
  last_name LIKE '_$_%' ESCAPE '$';#表示转义的字符可以自己指定
```

1.  between..and  匹配左闭右闭的区间
2.  in : 在什么范围内

```sql
1.提高语言简洁度
2.in 里面不能使用通配符
3.in里面必须是同一种类型才行
```

1.  is null/is not null:用于判断空值

```sql
is null PK <=>
        普通类型的数值  null值    可读性
is null    ×            √          √
<=>        √            √          ×

=  != 不能用于判断null

[[安全等于]] ： <=>
select 
  *
from
  employees
where 
  commission_pct <=> null;
[[安全等于可以替换is]] null中的is，但是普通的等于不行。
```

# 四、排序查询

## 1、语法

```sql
select 查询列表
from 表名
where 筛选条件
order by 排序列表 【asc/desc】
```

## 2、特点

1.  asc：升序排序，默认值

desc：降序

1.  排序列表支持单个字段、多个字段、函数、表达式、别名
2.  order by的位置一般放在查询语句的最后（limit之前）

# 五、常见函数

函数调用使用：select 函数名(实参)

## 1、单行函数

### 1.1、字符函数

concat：连接
substr：截取子串
upper：变大写
lower：变小写
replace：替换
length：获取字节长度
trim：去前后空格
lpad：左填充
rpad：右填充
instr：获取字符串第一次出现的索引

### 1.2、数学函数

ceil：向上取整
round：四舍五入
mod：取余
floor：向下取整
truncate：截断（此函数用于返回X的截断到小数位D号的值。 如果D为0，则小数点被除去。如果D是负的，那么D的值的整数部分值的数量被截断。）
rand：获取随机整数，返回0-1之间的小数。

### 1.3、日期函数

now：返回当前日期+时间
year：返回年
month：返回月
day：返回日
date\_format：将日期转换为字符
curdate：返回当前日期
str\_to\_date：将字符转换为日期
curtime：返回当前时间
hour：小时
minutes：分钟
second：秒
datediff：返回两个日期相差的天数
monthname：以英文形式返回月

### 1.4、其他函数

version：当前数据库服务器的版本
database：当前打开的数据库
user：当前用户
password('字符')：返回该字符的密码形式（MySQL8.0移除，换成SHA1）
md5('字符')：返回该字符的MD5加密形式

### 1.5、流程控制函数

1.  if(条件表达式，表达式1，表达式2)：如果条件表达式成立，返回表达式1，否则返回表达式2
2.  case情况1

case 变量或表达式或字段
when 常量1 then 值1
when 常量2 then 值2
。。。
else 值n
end

1.  case情况2

case
when 条件1 then 值1
when 条件2 then 值2
。。。
else 值n
end

## 2、分组函数

1.  分类

max：最大值
min：最小值
sum：和
avg：平均值
count：计数

1.  特点：

-   语法：
    -   select max(字段) from 表名；
-   支持的类型：
    -   sum和avg一般用于处理数值型
    -   max、min、count可以处理任何数据类型
-   以上分组函数都忽略null
-   都可以搭配distinct使用，实现去重统计
    -   select sum(distinct  字段) from 表；
-   count函数
    -   count(字段)：统计字段非空的个数
    -   count( \*)：统计结果集的行数
    -   count(1):统计结果集的行数
    -   效率：
        -   myIsam存储引擎：count( \*)最高
        -   innoDB存储引擎：count( \*)和count(1)效率 》 count(字段)
-   **和分组函数一同查询的字段，要求是group by后出现的字段**

# 六、分组查询

## 1、语法

```sql
select 分组函数 ，分组函数后的字段
form 表
where 筛选条件
group by 分组字段
having 分组后筛选
order by 排序列表
```

## 2、特点

```sql

          使用关键字    筛选的表    位置
分组前筛选  where        原始表        group by的前面
分组后筛选  having      分组后的结果  group by 的后面
```

# 七、连接查询

## 1、含义

当查询中涉及到了到了多个表的字段，需要使用多表连接
select 字段1，字段2
from 表1，表2，。。。。
笛卡尔乘积：当查询多个表时，没有添加有效的连接条件导致多个表所有行实现完全连接
如何解决：添加有效的连接条件

## 2、分类

按年代分：
SQL92：

-   等值连接
-   非等值连接
-   自连接
-   也支持一部分外连接（用于oracle、sqlserver，mysql不支持）

SQL99

-   内连接
    -   等值连接
    -   非等值连接
    -   自连接
-   外连接
    -   左外连接
    -   右外连接
    -   全外连接（mysql不支持）
-   交叉连接用来实现笛卡尔积

## 3、SQL92语法（只支持内连接）

### 1、等值连接

-   select 查询列表
-   from  表1 别名，表2 别名
-   where 表1,.key = 表2.key
-   and 筛选条件
-   group by 分组字段
-   having 分组后的筛选
-   order by排序字段
-   特点：
    -   一般为表起别名
    -   多表的顺序可以调换
    -   n表连接至少需要n-1个连接条件
    -   等值连接的结果是多表的交际部分

### 2、非等值连接

-   select 查询列表
-   from表1 别名，表2 别名
-   where 非等值连接条件
-   and 筛选条件
-   group by 分组字段
-   having 分组后的筛选条件
-   order by 排序字段

### 3、自连接

-   select 查询列表
-   from表  别名1，表 别名2  # 这是同一个表
-   where 等值的连接条件
-   and 筛选条件
-   group by分组字段
-   having 分组后的筛选
-   order by排序字段

## 4、SQL99语法

### 1、内连接

-   select 查询列表
-   from 表1 别名
-   inner join 表2 别名
-   on 连接条件
-   where 筛选条件
-   group by 分组列表
-   having 分组后筛选
-   order by排序列表
-   limit 字句

特点：

-   表的顺序可以调换
-   内连接的结果=多表的交集
-   n表相连至少需要n-1个连接条件

分类：
等值连接
非等值连接
自连接

### 2、外连接

-   select 查询列表
-   from 表1 别名
-   left | right | full 【outer】join 表2 别名
-   on 连接条件
-   where 筛选条件
-   group by 分组列表
-   having 分组后的筛选
-   order by 排序列表
-   limit 字句

特点：

-   查询的结果=主表中的所有行，如果从表和他匹配的将显示匹配，没有匹配的就显示为null
-   left join 左边的是主表，right join 右边的是主表  full join两边都是主表
-   一般用于查询处理交集部分的剩余的不匹配的行。

### 3、交叉连接

-   select 查询列表
-   from 表1 别名
-   cross join 表2 别名

特点：产生笛卡尔积

# 八、子查询

## 1、含义

嵌套在其他语句内部的select语句称为子查询或内查询，
外面的语句可以是insert、update、delete、select等一般使用select较多。
外面如果为select 语句，则此语句称为外查询或主查询

## 2、分类

### 1、按出现的位置

select后面：仅仅支持标量子查询
from后面：表子查询
where或having后面：标量子查询、列子查询、行子查询
exists后面：标量子查询、列子查询、行子查询、表子查询

### 2、按结果集的行列

-   标量子查询（单行子查询）：结果集为一行一列
-   列子查询（多行子查询）：结果集为多行一列
-   行子查询：结果集为多行多列
-   表子查询：结果集为多行多列

​

相关子查询：是否有值返回
exists(完整的子查询)
结果：0或者1

## 3、示例

where或having后面
1、标量子查询
案例：查询最低工资的员工姓名和工资
①最低工资
select min(salary) from employees

②查询员工的姓名和工资，要求工资=①
select last\_name,salary
from employees
where salary=(
select min(salary) from employees
);

2、列子查询
案例：查询所有是领导的员工姓名
①查询所有员工的 manager\_id
select manager\_id
from employees

②查询姓名，employee\_id属于①列表的一个
select last\_name
from employees
where employee\_id in(
select manager\_id
from employees
);

# 九、分页查询

## 1、应用场景

-   当查询的条目太多，一页显示不全

## 2、语法

-   select  查询列表
-   from 表
-   limit 【offset，】size

注意：
offset代表的是其实的条目索引，默认从0开始
size代表的是显示的条目数
公式：
假如要显示的页数为page，每一页条目数为size
select 查询列表
from 表
limit （page-1）\*size，size；

# 十、联合查询

## 1、含义

union：合并、联合，将多次查询结果合并成一个结果

## 2、语法

查询语句1
union 【all】
查询语句2
union 【all】
。。。。

## 3、意义

1.  将一条比较复杂的查询语句拆分成多条语句
2.  适用于查询多个表的时候，查询的列基本是一致。

## 4、特点

1.  要求多条查询语句的查询列数必须一致
2.  要求多条查询语句的查询的各列类型，顺序最好一致
3.  union去重，union  all包含重复项

# 十一、查询顺序

语法：
select 查询列表    ⑦
from 表1 别名       ①
连接类型 join 表2   ②
on 连接条件         ③
where 筛选          ④
group by 分组列表   ⑤
having 筛选         ⑥
order by排序列表    ⑧
limit 起始条目索引，条目数;  ⑨

# 十二、DML语言

## 1、插入

### 1、方式一：

-   语法：

insert into 表名（字段名） values（值。。。。）；

-   特点
    -   要求值的类型和字段的类型要一致或兼容。
    -   字段的个数和顺序不一定与原始表中的字段个数和顺序一致但必须保证值和字段一一对应
    -   加入表中有可以为null的字段，注意可以通过一下两种方式插入null。
        -   字段和值都省略
        -   字段写上使用null
    -   字段和值的个数必须一致
    -   字段名可以省略，默认所有列。

## 2.方式二：

-   语法：

insert into 表名 set 字段=值， 字段=值，。。。

-   两种方式的区别：
    -   方式一支持一次插入多行，语法如下：
        -   insert into 表名【（字段名，。。。）】values （值，。。），（值，。。）。。。
    -   方式一支持子查询：
        -   insert into 表名  查询语句

## 2、修改

### 1、修改单表的记录

语法：update  表名 set  字段=值，字段=值 【where 筛选条件】

### 2、修改多表的记录

语法：
update 表1 别名
left|right|inner join 表2 别名
on 连接条件
set 字段=值，字段=值
where 筛选条件

```sql
UPDATE beauty b
INNER JOIN boys bo
ON b.`boyfriend_id` = bo.`id`
SET phone = '110'
WHERE bo.`boyName` = '张无忌';
```

## 3、删除

### 1、使用delete

-   删除表的记录

delete from 表名 【where 筛选条件】【limit 条目数】

-   级联删除

delete from 别名1，别名2 from表1 别名
inner|left|right join 表2 别名
on 连接条件
【where 筛选条件】

### 2、使用truncate

truncate table 表名

### 3、两种方式的区别

1.  truncate 删除后，如果在插入数据，标识从1开始，delete删除后，如果在插入，标识从断点开始。
2.  delete可以添加筛选条件，truncate不可以添加筛选条件
3.  truncate效率高
4.  truncate没有返回值，delete可以返回受影响的行数
5.  truncate 不可以回滚，delete可以回滚

# 十三、DDL语言

## 1、数据库的管理

### 1、创建数据库

create database 【if not exists】 库名【character set 字符集名】

### 2、修改数据库

alter database 库名 character set 字符集名

### 3、删除库

drop database 【if exists】 库名；

## 2、表的管理

### 1、创建表

create table 【if not exists】 表名(
字段名  字段类型 【约束】，
字段名  字段类型 【约束】，
。。。
字段名  字段类型 【约束】
)

### 2、修改表

1.  添加列

alter table 表名 add column 列名 类型 【first|after 字段名    （这是指定位置）】

1.  修改列的类型或约束

alter table 表名 modify column 列名 新类型 【新的约束】；

1.  修改列名

alter table 表名 change column 旧列名  新列名 类型；

1.  删除列

alter table 表名 drop column 列名

1.  修改表名

alter  table 表名 rename 【to】 新表名；

### 3、删除表

drop table 【if exists】 表名；

### 4、复制表

-   复制表的结构
    -   create table 表名 like 旧表；
-   复制表的结构加数据
    -   create table 表名 select 查询列表 from 旧表 【where 筛选】

## 3、数据类型

### 1、数值型

-   整型

tinyint(1byte)、 smallint(2byte)、 mediumint(3byte)  、int/integer(4byte)、bigint(8byte)
特点：

1.  都可以设置无符号和有符号，默认有符号，通过unsigned设置无符号
2.  如果超出了范围，会报out of range异常，插入临界值
3.  长度可以不指定，默认会有一个长度
4.  长度代表显示的最大宽度，如果不够则左边用0填充，但需要搭配zerofill，并且默认变为无符号整型。

-   浮点型
    -   定点数：decimal(M,D);
    -   浮点数：
        -   float(M,D)  4byte
        -   double(M,D) 8byte
-   特点：

1.  M代表整数部位+小数部位的个数，D代表小数部位
2.  如果超出范围，则报out of range异常，并且插入临界值
3.  M和D都可以省略，但对于定点数，M默认为10，D默认为0
4.  如果精度要求较高，则优先考虑使用定点数。

### 2、字符型

char、varchar、binary、varbinary、enum、set、text、blob
char：固定长度的字符，写法为char(M),最大长度不能超过M，其中M
可以省略，默认为1
varchar：可变长度字符，写法为varchar（M），最大长度不能超过M，其中M不能省略的。

### 3、日期型

year 、date、time、datetime、timestamp

# 十四、常见的约束

-   NOT NULL：非空约束，该字段的值必填
-   UNIQUE:唯一约束，该字段的值不能重复
-   DEFAULT：默认约束，该字段的值不用手动插入就有默认值
-   CHECK:检查约束，mysql不支持
-   PRIMARY KEY：主键约束，该字段的值不可重复不能为空
-   FOREIGN KEY：外键约束，该字段的值引用另外的表的字段

## 1、主键和唯一

区别：

-   一个表中最多一个主键，但可以有多个唯一
-   主键不能为空，唯一可以为空，但只能一个为空

相同点：

-   都具有唯一性
-   都支持组合键，但不推荐

## 2、外键

1.  用于限制两个表的关系，从表的字段值引用了主表的某个字段值。
2.  外键列和主表的被引用列要去类型一致，意义一样，名称无要求。
3.  主表的被引用列要求是一个key（一般就是主键）
4.  插入数据，先插入主表
5.  删除数据，先删除从表
6.  可以通过一下两种方式删除主表的记录
    1.  级联删除

ALTER TABLE stuinfo ADD CONSTRAINT fk\_stu\_major FOREIGN KEY(majorid) REFERENCES major(id) ON DELETE CASCADE;

1.  级联置空

ALTER TABLE stuinfo ADD CONSTRAINT fk\_stu\_major FOREIGN KEY(majorid) REFERENCES major(id) ON DELETE SET NULL;

## 3、创建表时添加约束

create table 表名{
字段名 字段类型 not null,#非空
字段名 字段类型 primary key,#主键
字段名 字段类型 unique,#唯一
字段名 字段类型 default 值,#默认
constraint 约束名 foreign key(字段名) references 主表（被引用列）
}
注意：
支持类型			可以起约束名
列级约束		除了外键			不可以
表级约束		除了非空和默认	可以，但对主键无效

列级约束可以在一个字段上追加多个，中间用空格隔开，没有顺序要求

## 4、修改表时添加或删除约束

1、非空
添加非空
alter table 表名 modify column 字段名 字段类型 not null;
删除非空
alter table 表名 modify column 字段名 字段类型 ;

2、默认
添加默认
alter table 表名 modify column 字段名 字段类型 default 值;
删除默认
alter table 表名 modify column 字段名 字段类型 ;
3、主键
添加主键
alter table 表名 add【 constraint 约束名】 primary key(字段名);
删除主键
alter table 表名 drop primary key;

4、唯一
添加唯一
alter table 表名 add【 constraint 约束名】 unique(字段名);
删除唯一
alter table 表名 drop index 索引名;
5、外键
添加外键
alter table 表名 add【 constraint 约束名】 foreign key(字段名) references 主表（被引用列）;
删除外键
alter table 表名 drop foreign key 约束名;

## 5、自增长列

特点：

1.  不用手动插入值，可以自动提供序列值，默认从1开始，步长为1

auto\_increment\_increment
如果要更改起始值：手动插入值
如果要更改步长：更改系统变量
set auto\_increment\_increment = 值

1.  一个表最多有一个自增长列
2.  自增长列只能支持数值型
3.  自增长列必须为key

-   创建表时设置自增长列

create table 表(
字段名 字段类型 约束 auto\_increment
)

-   修改表时设置自增长列

alter table 表 modify column 字段名 字段类型 约束 auto\_increment

-   删除自增长列

alter table 表 modify column 字段名 字段类型 约束

# 十五、事务

## 1、含义

事务：一条或多条sql语句组成一个执行单位，要么都执行，要么都不执行

## 2、特点（ACID属性）

-   原子性：一个事务是不可分割的整体，要么都执行要么都不执行
-   一致性：一个事务可以使数据从一种一致状态切换到另一种一致状态
-   隔离性：一个事务不受其他事务的干扰，多个事务互相隔离的
-   持久性：一个事务一旦提交了，则永久的持久化到本地

## 3、事务的使用步骤

隐式事务：没有显示的开启和关闭，本身就是一个事务：insert、update、delete
显示事务：具有明显的开启和结束
使用显示事务：

1.  开启事务
    1.  set autocommit = 0;
    2.  start transaction [[可以省略]]
2.  编写一组逻辑SQL语句
    1.  注意：SQL语句支持 insert、update、delete
    2.  设置回滚点：
        1.  savepoint 回滚点名；
3.  结束事务
    1.  提交：commit
    2.  回滚：rollback
    3.  回滚到指定的地方：rollback to 回滚点名；

## 4、并发事务

1.  事务的并发问题是如何发生的？
    1.  多个事务同时操作同一个数据库的相同数据时
2.  并发问题都有哪些？
    1.  脏读：一个事务读取了其他事务还没有提交的数据，读到的是其他事务更新的数据
    2.  不可重复读：一个事务多次读取，结果不一样
    3.  幻读：一个事务读取了其他事务还没有提交的数据，只是读到的是其他事务插入的数据。
3.  如何解决并发问题
    1.  通过设置隔离级别来解决并发问题
4.  隔离级别
    ```纯文本
               脏读    不可重复读    幻读
    ```

read uncommitted:读未提交   	     ×                ×              	 ×
read committed：读已提交     	      √                ×              	×
repeatable read：可重复读     	      √                √              	×
serializable：串行化          		      √                √              	√
