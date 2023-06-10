# if判断语句

## 数学比较运算

```bash
        -eq         等于
        -gt         大于
        -lt         小于
        -ge         大于或等于
        -le         小于或等于
        -ne         不等于
```

## 字符串比较运算

```bash
 运算符解释，注意字符串一定别忘了使用引号引起来
         ==          等于   
         !=          不等于
         -n          检查字符串的长度是否大于0  
         -z          检查字符串的长度是否为0
```

## 文件比较与检查

```bash
         # test 命令可以获取以下说明
         -d  检查文件是否存在且为目录
         -e  检查文件是否存在
         -f  检查文件是否存在且为文件
         -r  检查文件是否存在且可读
         -s  检查文件是否存在且不为空
         -w  检查文件是否存在且可写
         -x  检查文件是否存在且可执行
         -O  检查文件是否存在并且被当前用户拥有
         -G  检查文件是否存在并且默认组为当前用户组
         file1 -nt file2  检查file1是否比file2新
         file1 -ot file2  检查file1是否比file2旧
```

## 逻辑运算

```bash
          逻辑与运算       &&   
          逻辑或运算       ||  
          逻辑非运算       ！
逻辑运算注意事项：
    逻辑与 或 运算都需要两个或以上条件，逻辑非运算只能一个条件。
    口诀:    逻辑与运算               真真为真 真假为假   假假为假
             逻辑或运算               真真为真 真假为真   假假为假
             逻辑非运算               非假为真   非真为假
```

## 赋值运算

```bash
=      赋值运算符         a=10   name='baism'
```

***

## 单if语句

```bash
if [ condition ]  # condition 的值为true或者false ，注意空格
then
  command
fi
```

## if-then-else语句

```bash
if [ condition ]
then
  command1
else
  command2
fi
```

## if-then-elif语句

```bash
if [ codition ]
then
  command1
elif [ condition ]
then
  command2
.....
else
  commandn
fi

```

## if高级使用

1、条件符号使用双圆括号，可以在条件中植入数学表达式

2、使用双方括号,可以在条件中使用通配符

```bash
#!/bin/bash

for var in rw rx wx rwx w
do
    if [[ $var == r* ]]
    then
        echo $var
    fi
done
```

## 案例

### 判断一个机器是否存活，能ping通就算存活

```bash
#!/bin/bash

ping -c 2 $1 &>/dev/null

if [ `echo $?` -eq 0 ]
then
    echo -e "\033[40;32m ok \033[0m"
else
    echo -e "\033[40;31m fail \033[0m"
fi
```

## 判断服务器上的某个服务是否开启

```bash
#!/bin/bash
# 判断某个端口是否在监听
lsof -i  :$1 &>/dev/null

if [ `echo $?` -eq 0 ]
then
    echo -e "\033[40;32m ok \033[0m"
else
    echo -e "\033[40;31m fail \033[0m"
fi
```

## 判断某年是否为闰年

```bash
#!/bin/bash

# 闰年判断
# 能够被4整除不能被100整除
# 能够被400整除
# 注：表达式运算问题

year=$1
if [ `expr $year % 4` -eq 0 ] && [ `expr $year % 100` -ne 0 ] || [ `expr $year % 400` -eq 0 ]
then
    echo "$year is leap year"
else
    echo "$year is not leap year"
fi
```

写一个Nginx安装脚本，加入判断，当上一步执行成功在执行下一步，否则退出脚本

写一个备份脚本，将A机器当天修改过的文件收集到/cache目录，打包并发送到B机器的/opt/backup目录下，通过MD5值判断是否B机器上的备份的完整性
