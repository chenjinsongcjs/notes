# shell基本输入

## read命令

```bash
默认接受键盘的输入，回车符代表输入结束

read 命令选项

-p打印信息

-t限定时间

-s不回显

-n输入字符个数
```

```bash
#! /bin/bash
read -p "login: " acc
read -p "password: " -n6 -s -t60 pw
echo "account: $acc  password: $pw"
```
