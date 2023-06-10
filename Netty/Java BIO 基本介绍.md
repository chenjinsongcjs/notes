# Java BIO 基本介绍

介绍

-   java BIO是传统的java io模型，其相关的类和接口在java.io包下
-   BIO（Blocking IO）：同步阻塞IO，服务器的实现模式为一个连接对应一个线程处理，即客户端每次发起一个连接请求服务器都会创建出一个新的线程进行处理，如果这个连接不做任何事情，那么就会进入阻塞状态，这样就会导致不必要的线程消耗，可以使用线程池技术来缓解。

图例

![](https://notes-pic-cjs.oss-cn-chengdu.aliyuncs.com/obsidian/image_F1WJlLlA9h.png)

### Java BIO的编程流程

1.  服务器端启动一个ServerSocket监听客户端的请求
2.  客户端创建socket与服务器端建立连接，默认情况下，一个客户端的请求服务器端会创建一个线程连接
3.  客户端发起请求后，先咨询服务器端是否有线程进行处理，如果没有就等待或者被拒绝
4.  如果有响应，等响应结束后，可以发送下一次请求

### 代码示例

```java
public class Server {
    public static void main(String[] args) throws IOException {
        //使用线程池缓解服务器端的压力
        ExecutorService threadPool = Executors.newCachedThreadPool();
        //服务器端创建ServerSocket，并绑定端口进行监听
        ServerSocket serverSocket = new ServerSocket(8888);
        while(true){
            //监听来自客户端的请求
            Socket accept = serverSocket.accept();
            threadPool.execute(()->{
                System.out.println("thread:"+Thread.currentThread().getName()+"--> accept");
                handle(accept);
            });

        }
    }

    private static void handle(Socket socket){
        try (
                InputStream inputStream = socket.getInputStream();
                ){
            byte[] buffer = new byte[1024];

            int read = 0;
            while((read = inputStream.read(buffer)) != -1){
                System.out.println( "thread: "+Thread.currentThread().getName()+" msg:"+new String(buffer,0,read));
            }
        } catch (IOException e) {
            //发生异常关闭资源
            try {
                socket.close();
            } catch (IOException ioException) {
                ioException.printStackTrace();
            }
            e.printStackTrace();
        }

    }

}

```

```java
public class Client {
    public static void main(String[] args) throws IOException {
        InetSocketAddress address = new InetSocketAddress("127.0.0.1",8888);
        Socket socket = new Socket();
        //与服务器端建立连接
        socket.connect(address);
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()){
            String line = scanner.nextLine();
            OutputStream outputStream = socket.getOutputStream();
            outputStream.write(line.getBytes());
        }
    }
}

```

### Java BIO的问题分析

-   Client的每一个请求都需要Server开启一个线程去处理读写操作
-   当并发量大的时候，需要创建大量的线程，需要耗费大量的系统资源
-   当连接创建之后如果此次请求没有数据传输，那么线程就会阻塞等待，浪费线程资源
