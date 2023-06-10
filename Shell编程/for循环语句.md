# for循环语句

## for语法1

```bash
for var in value1 value2 ......
    do
        commands
done
```

```bash
#!/bin/bash

for i in `seq 1 2 9`
do
    echo -n "$i "

done
```

## for语法2

```bash
for ((变量;条件;自增减运算  )) # 可以有多个变量
   do
          代码块
done
# 无限循环
for((;;))
do
    command
done

```

```bash
#!/bin/bash

for ((i=0;i<9;i++))
do
    echo -n "$i "

done
```

```bash
#!/bin/bash
i=1
for((;;))
do
    if [ $i -lt 10 ];then
            echo -n -e "\b$i"
    elif [ $i -lt 100 ];then
            echo -n -e "\b\b$i"
    elif [ $i -lt 1000 ];then
            echo -n -e "\b\b\b$i"
    else
        exit
    fi
        i=$((i+1))
        sleep 1
done
```

## continue:跳过本次循环进入下一次循环

## break：跳出循环

## break N ：跳出多重循环，从1开始

## 案例

1.扫描一下你的网段中的那些机器是存活的。

2.手动写一个同步拉脚本，要求B机器每隔10分钟把A机器的/opt/cache/下的内容拉取到B机器的/opt/cache，并做完整性验证

3.新建user01-user20用户，要求密码是随机6位数 密码取值范围a-zA-Z0-9，要求密码不能只是单一的数字或小写或大写字母。

4.写一个mysql分库备份脚本

5.写一个猜数字脚本，数字范围是1-100，定制计数器，每次猜完都要告诉用户猜大或猜小了，如果猜对了跳出脚本并输出计数器的值

6.为DNS写一个自动判断WEB解析的脚本

公司DNS将域名www\.ayitula.com解析到了两个WEB服务器，以实现dns负载均衡。但是当某个WEB服务器出现故障，那么DNS依然会将用户解析到宕机WEB，造成不能正常访问。故要求写一个运行在DNS的检查脚本，当发现哪台WEB宕机后，自己修改DNS的解析记录，关闭对其的解析。当WEB恢复，DNS在打开对其的解析，恢复解析。

7.写一个99乘法表
