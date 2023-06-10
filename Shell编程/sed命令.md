# sed命令

-   sed是linux中提供的一个外部命令,它是一个行(流)编辑器，非交互式的对文件内容进行增删改查的操作，使用者只能在命令行输入编辑命令、指定文件名，然后在屏幕上查看输出。它和文本编辑器有本质的区别
-   区别是：
    -   文本编辑器: 编辑对象是文件
    -   行编辑器：编辑对象是文件中的行
-   也就是前者一次处理一个文本，而后者是一次处理一个文本中的一行。这个是我们应该弄清楚且必须牢记的，否者可能无法理解sed的运行原理和使用精髓。

## sed数据处理原理

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_EgzOz2IhIf.png)

## sed命令

> sed \[options] ‘{command}\[flags]’ \[filename]

```bash
命令选项
-e script 将脚本中指定的命令添加到处理输入时执行的命令中  多条件，一行中要有多个操作
-f script 将文件中指定的命令添加到处理输入时执行的命令中
-n        抑制自动输出
-i        编辑文件内容
-i.bak    修改时同时创建.bak备份文件。
-r        使用扩展的正则表达式
!         取反 （跟在模式条件后与shell有所区别）
sed常用内部命令
a   在匹配后面添加
i   在匹配前面添加
p   打印
d   删除
s   查找替换
c   更改
y   转换   N D P 
flags
数字             表示新文本替换的模式
g：             表示用新文本替换现有文本的全部实例
p：             表示打印原始的内容
w filename:     将替换的结果写入文件
```

## 案例

```bash
在data1的每行后追加一行新数据内容: append data "haha"
[root@www ~]# sed 'a\append data "haha"' data1
1 the quick brown fox jumps over the lazy dog.
append data "haha"
2 the quick brown fox jumps over the lazy dog.
append data "haha"
3 the quick brown fox jumps over the lazy dog.
append data "haha"
4 the quick brown fox jumps over the lazy dog.
append data "haha"
5 the quick brown fox jumps over the lazy dog.
append data "haha"
在第二行后新开一行追加数据: append data "haha"
[root@www ~]# sed '2a\append data "haha"' data1
1 the quick brown fox jumps over the lazy dog.
2 the quick brown fox jumps over the lazy dog.
append data "haha"
3 the quick brown fox jumps over the lazy dog.
4 the quick brown fox jumps over the lazy dog.
5 the quick brown fox jumps over the lazy dog.
在第二到四行每行后新开一行追加数据: append data "haha"
[root@www ~]# sed '2,4a\append data "haha"' data1
1 the quick brown fox jumps over the lazy dog.
2 the quick brown fox jumps over the lazy dog.
append data "haha"
3 the quick brown fox jumps over the lazy dog.
append data "haha"
4 the quick brown fox jumps over the lazy dog.
append data "haha"
5 the quick brown fox jumps over the lazy dog.
匹配字符串追加: 找到包含"3 the"的行，在其后新开一行追加内容: append data "haha"
[root@www ~]# sed '/3 the/a\append data "haha"' data1
1 the quick brown fox jumps over the lazy dog.
2 the quick brown fox jumps over the lazy dog.
3 the quick brown fox jumps over the lazy dog.
append data "haha"
4 the quick brown fox jumps over the lazy dog.
5 the quick brown fox jumps over the lazy dog.
//开启匹配模式  /要匹配的字符串/
```

## sed小技巧

-   \$= 统计文本有多少行

```bash
统计data2有多少行
[root@www ~]# sed -n '$=' data2
5
打印data2内容时加上行号
[root@www ~]# sed  '=' data2
1
1 the quick brown fox jumps over the lazy dog . dog
2
2 the quick brown fox jumps over the lazy dog . dog
3
3 the quick brown fox jumps over the lazy dog . dog
4
4 the quick brown fox jumps over the lazy dog . dog
5
5 the quick brown fox jumps over the lazy dog . dog
```

## DNS监测WEB服务状态，并根据其状态实现高可用解析，场景：通过DNS进行单域名多条A记录解析做负载均衡。

```bash
#!/bin/bash
CP1=0
CP2=0
while :
  do
#tong
     ping -c1 192.168.18.240 > /dev/null
     if [ $? -eq 1 ] && [ $CP1 -eq 0 ]
        then
           sed -i '/192.168.18.240/s/^/;/' /var/named/baidu.zone
           /etc/init.d/named reload
           CP1=1
      fi
#butong
     ping -c1 192.168.18.240 > /dev/null 
     if [ $? -eq 0 ] && [ $CP1 -eq 1 ]
        then
            sed -i '/192.168.18.240/s/;//' /var/named/baidu.zone
            /etc/init.d/named reload
            CP1=0
     fi 
    ping -c1 192.168.18.241 > /dev/null
    if [ $? -eq 1 ] && [ $CP2 -eq 0 ]
        then
          sed -i '/192.168.18.241/s/^/;/' /var/named/baidu.zone
          /etc/init.d/named reload
          CP2=1
    fi
       ping -c1 192.168.18.241 > /dev/null
     if [ $? -eq 0 ] && [ $CP2 -eq 1 ]
        then
            sed -i '/192.168.18.241/s/;//' /var/named/baidu.zone
            /etc/init.d/named reload
            CP2=0
     fi
sleep 5
done
```
