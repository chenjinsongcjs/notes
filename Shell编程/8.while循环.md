# while循环

## while循环介绍

-   知道循环次数就选择for，否则就选择while

## while循环语法

```bash
while  [ condition ]      #注意，条件为真while才会循环，条件为假，while停止循环
 do
             commands
done
```

## 遍历文件内容

```bash
#!/bin/bash

while read i
do
    echo "$i"
done < $1
```
