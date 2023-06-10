# LongAdder源码

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_O9occ1b3mW.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_9vEkc8ToLI.png)

> 无竞争状态只更新base
> base更新失败，创建Cell数组，多线程去更新Cell数组
> 多线程竞争激烈，对Cell数组进行扩容
