# HTTP

-   它是超文本传输协议，HTTP是缩写，它的全英文名是HyperText Transfer Protocol
-   超文本指的是HTML，css，JavaScript和图片等，HTTP的出现是为了接收和发布HTML页面，经过不断的发展也可以用于接收一些音频，视频，文件等内容。

## HTTP特点

-   HTTP都是由客户端发起请求的，并且由服务器端回应响应消息的。
-   灵活，允许任何类型的数据对象，包括音频，视频，图片，文件等等。
-   无状态，HTTP就是说，每次HTTP请求都是独立的，任何两个请求之间没有必然的联系。
-   无连接的，每次服务器在处理完客户端的请求后，并收到客户的应答后，就断开了通信，当客户端再次发送请求时就是一个新的连接。
-   **这是HTTP/1.0版的主要缺点，** 每个TCP连接只能发送一个请求，发送数据完毕后，连接就关闭了，如果还要请求就必须要新建一个请求连接。

### HTTP/1.1引入Cookie解决无状态问题

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_TKlQUcXxho.png)

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_vyUWHXEAmp.png)

-   HTTP/1.1版本是最流行的版本，**可以持久连接，TCP连接默认不关闭**，可以被多个请求复用，只有在一段时间内，没有请求，就可以自动关闭。

```java
// 不用声明：
Connection: keep-alive
// 发送关闭
Connection: close
// 要求服务器关闭TCP连接
```

## HTTP的消息格式

-   一个请求消息是由**请求行，请求头字段，一个空行和消息主体**构成。

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_rQjkN9mbNy.png)

-   一个响应消息由响应行，响应头字段，空行，响应消息体组成

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_Puioo7UcPd.png)

-   URL和URI

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_eEFF3WDEhC.png)

### HTTP/1.1中的请求方法

-   GET ：获取服务器端资源
-   POST：向服务器端传输消息
-   PUT：上传传输
-   DELETE：删除文件
-   HEAD：获取报文首部
-   OPTIONS：询问服务器支持的请求方法
-   TRACE为回显服务器收到额请求

### HTTP状态码

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_QKbobyfnIs.png)
