# util循环

-   和while正好相反，until是条件为假开始执行，条件为真停止执行。

## util语法

```bash
until [ condition ]      #注意，条件为假until才会循环，条件为真，until停止循环
 do
         commands代码块
done
```

```bash
init_num=10
until [ $init_num -gt 20 ]
   do
       echo $init_num
       init_num=$((init_num + 1))
done

```
