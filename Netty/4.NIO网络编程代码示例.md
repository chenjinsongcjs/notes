# NIO网络编程代码示例

### 服务器端

```java
//群聊服务器
//接收消息并且转发消息
public class GroupChatServer {
    private ServerSocketChannel serverSocketChannel;
    private final int PORT = 8888;
    private Selector selector;

    //初始化服务器
    public GroupChatServer() {
        try {
            //创建ServerSocketChannel
            serverSocketChannel = ServerSocketChannel.open();
            //绑定端口
            InetSocketAddress socketAddress = new InetSocketAddress(PORT);
            serverSocketChannel.socket().bind(socketAddress);
            //创建Selector
            selector = Selector.open();
            //设置通道为非阻塞
            serverSocketChannel.configureBlocking(false);
            //Channel注册到selector上,事件为accept,通道注册要在最后
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //监听客户端请求
    public void listen() throws Exception {
        while (true) {
            //监听Selector上的事件
            int select = selector.select(2000);
            //监听到事件发生
            if (select > 0) {
                //获取内部集合，有事件发生的集合
                Iterator<SelectionKey> keys = selector.selectedKeys().iterator();
                while (keys.hasNext()) {
                    SelectionKey key = keys.next();
                    //判断是什么时间分贝进行处理
                    if (key.isAcceptable()) {
                        //第一次请求连接，上线
                        SocketChannel socketChannel = serverSocketChannel.accept();
                        System.out.println(socketChannel.getRemoteAddress() + ":上线");
                        //通道设置非阻塞
                        socketChannel.configureBlocking(false);
                        //通道注册,将通道中的数据写到缓存区中,,通道注册一定最后
                        socketChannel.register(selector, SelectionKey.OP_READ);
                    }
                    if (key.isReadable()) {
                        //读取信息,通道已经建立完毕 转发消息
                        SocketChannel channel = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        channel.read(buffer);
                        String msg = new String(buffer.array());
                        System.out.println("from:" + msg);
                        msg = msg.trim();
                        if (msg != null && !"".equals(msg)) {
                            forwardMsgToOther(msg, key);
                        }
                    }
                    //处理完每一个SelectionKey都要删除，否则会重复处理
                    keys.remove();
                }
            }
        }
    }

    private void forwardMsgToOther(String msg, SelectionKey key) {
        //获取所有的通道并且发送消息
        Iterator<SelectionKey> iterator = selector.keys().iterator();
        System.out.println("消息转发中...");
        while (iterator.hasNext()) {
            SelectionKey sk = iterator.next();
            if (sk == key) {
                //如果是发送消息的本人不用转发
                continue;
            }
            if (sk.channel() instanceof SocketChannel) {
                SocketChannel channel = (SocketChannel) sk.channel();
                try {
                    //设置非阻塞
                    channel.configureBlocking(false);
                    channel.write(ByteBuffer.wrap(msg.getBytes()));
                } catch (IOException e) {
                    //转发出现异常，认为用户下线
                    try {
                        System.out.println(channel.getRemoteAddress() + "：下线");
                        //取消注册
                        sk.cancel();
                        //关闭通道
                        channel.close();
                    } catch (IOException ioException) {
                        ioException.printStackTrace();
                    }
                }
            }
        }
    }

    public static void main(String[] args) throws Exception {
        System.out.println("服务器启动...");
        GroupChatServer server = new GroupChatServer();
        server.listen();
    }
}
```

### 客户端

```java
public class GroupChatClient {
    private SocketChannel socketChannel;
    private Selector selector;
    private String userName;

    public GroupChatClient(String serverHost, int serverPort) {
        try {
            //创建socketChannel,建立连接
            socketChannel = SocketChannel.open(new InetSocketAddress(serverHost, serverPort));
            //设置非阻塞
            socketChannel.configureBlocking(false);
            //注册到selector中，主要应对多个连接情况
            selector = Selector.open();
            socketChannel.register(selector, SelectionKey.OP_READ);
            userName = socketChannel.getLocalAddress().toString().substring(1);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    //接收消息
    public void receiveData() throws IOException {
        System.out.println("客户端读取消息...");
        while (true) {
            int select = selector.select(2000);
            if (select > 0) {
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                while (iterator.hasNext()) {
                    SelectionKey key = iterator.next();
                    if (key.isReadable()) {
                        SocketChannel channel = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        int read = channel.read(buffer);
                        System.out.println(new String(buffer.array(), 0, read));
                    }
                    //防重
                    iterator.remove();
                }
            }
        }
    }

    //发送消息
    public void sendMsg() {
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNextLine()) {
            String msg = userName +": "+ scanner.nextLine();
            try {
                socketChannel.write(ByteBuffer.wrap(msg.getBytes()));
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        GroupChatClient client = new GroupChatClient("127.0.0.1", 8888);
        new Thread(() -> {
            try {
                client.receiveData();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
        client.sendMsg();
    }
}

```
