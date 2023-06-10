# Redis的缓存过期淘汰策略

### Redis内存满了怎么办

-   redis默认内存多少？在哪里查看?如何设置修改?
    -   打开redis配置文件，设置maxmemory参数，maxmemory是bytes字节类型，注意转换。
-   redis默认内存多少可以用？

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_YefO3_vOlE.png)
-   一般生产上你如何配置？
    -   一般推荐Redis设置内存为最大物理内存的四分之三
-   如何修改redis内存设置
    -   通过修改文件配置
    -   通过命令修改

        ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_9-ZzkrXyho.png)
-   什么命令查看redis内存使用情况?
    -   info memory
-   真要打满了会怎么样？如果Redis内存使用超出了设置的最大值会怎样？

    ![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Z6B5-ERGkG.png)

### 往redis里写的数据是怎么没了的?它如何删除的？

#### redis过期键的删除策略

-   立即删除
    -   CPU不断遍历那些设置了过期时间的key，如果发现过期，立即删除。（不用）
    -   对CPU不友好，使用处理器性能换空间
-   惰性删除
    -   对于一个已经过期的key我们不会立即去处理他，在下一次访问的过程中去判断他是否过期，如果过期，饥饿执行删除操作，如果没有过期，就返回数据
    -   对内存不友好，有些key可能永远都不会被访问到，就会导致内存的泄漏，用空间换时间
-   定期删除
    -   定期删除时对上面两种删除策略的折中，
    -   定期删除每个一段时间执行一次过期键删除操作，通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响。
    -   定期抽样key，判断是否过期

### Redis缓存淘汰策略

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_66DLin80vs.png)
