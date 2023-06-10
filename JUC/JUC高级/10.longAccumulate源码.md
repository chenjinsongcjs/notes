# longAccumulate源码

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_HLnYWwDdQH.png)



![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Ek5ZUVzOKv.png)

> 上述代码首先给当前线程分配一个hash值，然后进入一个for(;;)自旋，这个自旋分为三个分支：
> CASE1：Cell\[]数组已经初始化
> CASE2：Cell\[]数组未初始化(首次新建)
> CASE3：Cell\[]数组正在初始化中

-   未初始化过Cell\[]数组，尝试占有锁并首次初始化cells数组

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_rLp1DEFF9D.png)

-   多个线程尝试CAS修改失败的线程会走到这个分支（当前Cell数组正在初始化，对Base进行CAS）

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_w3M5F9_GyT.png)

-   Cell数组不再为空且可能存在Cell数组扩容

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_m6P4A_I2_c.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_HCU_odvT-c.png)
