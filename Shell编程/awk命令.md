# awk命令

## awk介绍

awk是一种可以处理数据、产生格式化报表的语言，功能十分强大。awk 认为文件中的每一行是一条记录 记录与记录的分隔符为换行符,每一列是一个字段 字段与字段的分隔符默认是一个或多个空格或tab制表符.

awk的工作方式是读取数据，将每一行数据视为一条记录（record）每条记录以字段分隔符分成若干字段，然后输出各个字段的值.

> awk \[options] \[BEGIN]{program} \[END]\[file]

```bash
常用命令选项
-F fs 指定描绘一行中数据字段的文件分隔符  默认为空格
-f file 指定读取程序的文件名
-v var=value 定义awk程序中使用的变量和默认值
注意：awk 程序脚本由左大括号和右大括号定义。脚本命令必须放置在两个大括号之间。由于awk命令行假定脚本是单文本字符串，所以必须将脚本包
括在单引号内。
awk程序运行优先级是:
    1)BEGIN: 在开始处理数据流之前执行，可选项
    2)program: 如何处理数据流，必选项
    3)END: 处理完数据流后执行，可选项
```

## awk对字段(列)的提取

```bash
字段提取:提取一个文本中的一列数据并打印输出
列默认是空格分开的
字段相关内置变量

$0 表示整行文本

$1 表示文本行中的第一个数据字段

$2 表示文本行中的第二个数据字段

$N 表示文本行中的第N个数据字段

$NF 表示文本行中的最后一个数据字段

awk '{print $0}' test 

```

## 命令选项详解

```bash
-F: 指定字段与字段的分隔符

当输出的数据流字段格式不是awk默认的字段格式时，我们可以使用-F命令选项来重新定义数据流字段分隔符。比如:

处理的文件是/etc/passwd，希望打印第一列、第三列、最后一列

awk -F ':' '{print $1,$3,$NF}' /etc/passwd
可以看的出，awk输出字段默认的分隔符也是空格

-f file: 如果awk命令是日常重复工作，而又没有太多变化，可以将程序写入文件，每次使用-f调用程序文件就好，方便，高效。

-v 定义变量，既然作者写awk的时候就是按着语言去写的，那么语言中最重要的要素—-变量肯定不能缺席，所以可以使用-v命令选项定义变量
awk -v name='baism' 'BEGIN{print name}'
baism

```

## awk对记录(行)的提取

```bash
记录提取：提取一个文本中的一行并打印输出

记录的提取方法有两种：a、通过行号 b、通过正则匹配

记录相关内置变量

NR: 指定行号

指定行号为3
[root@www ~]# awk 'NR==3{print $0}' test 
3 the quick brown fox         jumps over the lazy cat . dog
指定行的第一个字段精确匹配字符串为3
[root@www ~]# awk '$1=="3"{print $0}' test 
3 the quick brown fox         jumps over the lazy cat . dog

```

## awk对字符串提取

```bash
记录和字段的汇合点就是字符串
打印test第三行的第六个字段
awk 'NR==3{print $6}' test
jumps


```

### awk程序的优先级

关于awk程序的执行优先级，BEGIN是优先级最高的代码块，是在执行PROGRAM之前执行的，不需要提供数据源，因为不涉及到任何数据的处理，也不依赖与PROGRAM代码块；PROGRAM是对数据流干什么，是必选代码块，也是默认代码块。所以在执行时必须提供数据源；END是处理完数据流后的操作，如果需要执行END代码块，就必须需要PROGRAM的支持，单个无法执行。

```bash
优先级展示
awk 'BEGIN{print "hello ayitula"}{print $0}END{print "bye ayitula"}' test
hello ayitula
1 the quick brown fox jumps over the lazy cat . dog
2 the quick brown fox jumps over the lazy cat . dog
3 the quick brown fox         jumps over the lazy cat . dog
4 the quick brown fox jumps over the lazy cat . dog
5 the quick brown fox jumps over the lazy cat . dog
bye ayitula
不需要数据源，可以直接执行
awk 'BEGIN{print "hello world"}'
hello world
没有提供数据流，所以无法执行成功
awk '{print "hello world"}'
awk 'END{print "hello world"}'
```

# awk高级用法

awk是一门语言，那么就会符合语言的特性，除了可以定义变量外，还可以定义数组，还可以进行运算，流程控制。

### awk定义数组

```bash
数组定义方式: 数组名[索引]=值
定义数组array，有两个元素，分别是100，200，打印数组元素。
awk 'BEGIN{array[0]=100;array[1]=200;print array[0],array[1]}'
100 200

```

### awk运算

```bash
赋值运算 =

比较运算 > >= == < <= !=

数学运算 + - / % * ++ —

逻辑运算 && ||

匹配运算 ~ !~


匹配运算
awk -F ':' '$1 ~ "^ro" {print $0}' /etc/passwd
root:x:0:0:root:/root:/bin/bash

```

## awk 环境变量

| 变量          | 描述                          |
| ----------- | --------------------------- |
| FIELDWIDTHS | 以空格分隔的数字列表，用空格定义每个数据字段的精确宽度 |
| FS          | 输入字段分隔符号                    |
| OFS         | 输出字段分隔符号                    |
| RS          | 输入记录分隔符                     |
| ORS         | 输出记录分隔符号                    |

```bash
FIELDWIDTHS:重定义列宽并打印，注意不可以使用$0打印所有，因为$0是打印本行全内容，不会打印你定义的字段
awk 'BEGIN{FIELDWIDTHS="5 2 8"}NR==1{print $1,$2,$3}' /etc/passwd
root: x: 0:0:root
FS:指定数据源中字段分隔符，类似命令选项-F
awk 'BEGIN{FS=":"}NR==1{print $1,$3,$NF}' /etc/passwd
root 0 /bin/bash
OFS:指定输出到屏幕后字段的分隔符
awk 'BEGIN{FS=":";OFS="-"}NR==1{print $1,$3,$NF}' /etc/passwd
root-0-/bin/bash
RS:指定记录的分隔符
awk 'BEGIN{RS=""}{print $1,$13,$25,$37,$49}' test
1 2 3 4 5
将记录的分隔符修改为空行后，所有的行会变成一行，所以所有字段就在一行了。
ORS:输出到屏幕后记录的分隔符，默认为回车
awk 'BEGIN{RS="";ORS="*"}{print $1,$13,$25,$37,$49}' test
1 2 3 4 5*[root@www ~]# 
可以看出，提示符和输出在一行了，因为默认回车换成了*
```

## 流程控制

```bash
if判断语句
awk '{if($1>5)print $1/2;else print $1*2}' num

for循环语句
 awk '{sum=0;for (i=1;i<4;i++){sum+=$i}print sum}' num2
while循环语句
awk '{sum=0;i=1;while(sum<150){sum+=$i}print sum}' num2
do…while语句
awk '{sum=0;i=1;do{sum+=$i}while(sum<150);i++;print sum}' num2
循环控制
break   continue
```

## awk小技巧

```bash
打印test文本的行数 
awk 'END{print NR}' test 
5
打印test文本最后一行内容
awk 'END{print $0}' test 
5 the quick brown fox jumps over the lazy cat . dog
打印test文本列数
awk 'END{print NF}' test 
12
```
