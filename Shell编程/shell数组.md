# shell数组

## 数组介绍

-   数组中可以存放多个值。Bash Shell 只支持一维数组（不支持多维数组），初始化时不需要定义数组大小（与 PHP 类似。**与大部分编程语言类似，数组元素的下标由 0 开始**。

## 数组语法

```bash
数组名称=(元素1 元素2 元素3 ...)  #数组元素使用空格分开
array=(1 2 3 4 5)

```

## 数组读出

```bash
语法：${数组名称[索引]} 
echo ${array[0]}
1
```

## 数组赋值

-   一次赋一个值

```bash
array[0]=10
array[1]=11
array[3]=12
...
```

-   一次赋值多个

```bash
array=(1 2 3 4 5)

array=( `cat /etc/passwd` )  #将文件中的每一行作为一个元素赋值给array

array=( `ls /var/xxx` ) #将输出的每一行作为一个元素赋值给array
```

## 查看数组

```bash
declare -a
```

## 访问数组元素

```bash
echo ${array1[0]} #访问数组中的第一个元素

echo ${array1[@]} #访问数组中所有元素 等同于 echo ${array1[*]}

echo ${#array1[@]} #统计数组元素的个数

echo ${!array2[@]} #获取数组元素的索引

echo ${array1[@]:1} #从数组下标1开始

echo ${array1[@]:1:2} #从数组下标1开始，访问两个元素
```

## 遍历数组

```bash
#!/bin/bash
array=('a' 'b' 'c' 'd')

for i in ${array[@]}
do
    echo $i
done
```

## 关联数组

-   关联数组允许用户自定义数组索引，这样使用起来更加方便，类似于map

## 定义关联数组

> declare -A  数组名

```bash
declare -A ass_array
```

## 关联数组赋值

-   一次赋值一个

```bash
数组名[索引]=值
ass_array["name"]="张三"

```

-   一次赋值多个

```bash
数组名=(["name"]="李四" ["age"]=19)
```

## 查看数组

```bash
declare -A
```

## 访问数组元素

```bash
echo ${ass_array2[index2]} 访问数组中的第二个元数

echo ${ass_array2[@]} 访问数组中所有元数 等同于 echo ${array1[*]}

echo ${#ass_array2[@]} 获得数组元数的个数

echo ${!ass_array2[@]} 获得数组元数的索引

```

```bash
echo ${ass_array2["name"]}
```

## 案例

```bash
#!/bin/bash

# input student info

for ((i=0;i<3;i++))
do
    read -p "请输入第$((i+1))个人的姓名：" name[$i]
    read -p "请输入第$[i+1]个人的年龄：" age[$i]
    read -p "请输入第`expr $i + 1`个人的性别： " gender[$i]
done

clear
echo -e "\t\t学员查询系统"
while :
do
    read -p "请输入学员的姓名：" xm
    [ $xm == 'Q' ]&&exit
    flag=0
    for ((i=0;i<3;i++))
    do
        if [ $xm == ${name[$i]} ]
        then
            echo "${name[$i]}  ${age[$i]}  ${gender[$i]}"
            flag=1
        fi
    done
    [ $flag -eq 0 ]&&echo "not found $xm"
done
```

## 使用关联数组统计文件/etc/passwd中用户使用的不同类型shell的数量

```bash
#!/bin/bash

shell_num=( `awk -F: '{print $NF}' /etc/passwd` )
declare -A shell_type
for shell in ${shell_num[@]}
do
    shell_type[$shell]=0
done

for s in ${shell_num[@]}
do
    shell_type[$s]=`expr ${shell_type[$s]} + 1`
done

for sh in ${!shell_type[@]}
do
    echo -e "shell: $sh \t num: ${shell_type[$sh]}"
done
```

## 使用关联数组按扩展名统计指定目录中文件的数量

```bash
#!/bin/bash

# statistic the num of same subfix file

read -p "输入文件夹绝对路径：" filePath

dir=/tmp/subfix.txt

#目录不存在
[ ! -d $filPath ]&&exit

ls $filePath > $dir

cut -d '.' -f2 $dir > /tmp/subfix.sub

arr=( `cat /tmp/subfix.sub` )
declare -A subfix_type
for subfix in ${arr[@]}
do
    subfix_type[$subfix]=0
done

for subfix in ${arr[@]}
do
 #   echo $subfix
    subfix_type[$subfix]=`expr ${subfix_type[$subfix]} + 1`
done

for t in ${!subfix_type[@]}
do
    echo "subfix: $t  -->num: ${subfix_type[$t]}"
done
# delete temporary dir
rm -fr $dir
rm -fr /tmp/subfix.sub

```
