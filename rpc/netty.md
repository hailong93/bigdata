[TOC]

# netty

## 简介

Netty is an asynchronous event-driven network application framework 
for rapid development of maintainable high performance protocol servers & clients.

netty是一个异步的基于事件驱动的网络申请框架。

能够快速开发易维护高性能的服务或者客户端。

## 应用场景

1. 基于socket实现远程过程的调用
2. 基于websocket实现客户端与服务端的长连接通信
3. 基于http协议，实现http服务器，但没有遵循servlet标准

##  快速入门

### 入门一：基于http协议

1. 功能：客户端向服务端发送请求,服务器返回hello world.

2. 环境准备：idea gradle

3. gradle的常用命令

   ~~~
   gradle clrean bulid
   ~~~

4. idea新建项目netty_gradle,引入netty的依赖

   ~~~groovy
   plugins {
       id 'java'
   }
   
   sourceCompatibility = '1.8'
   targetCompatibility = '1.8'
   
   group = "com.netty.gradle"
   version = "1.0"
   
   repositories {
       mavenCentral()
   }
   
   dependencies {
       testCompile(
               "junit:junit:4.12"
       )
       compile(
               "io.netty:netty-all:4.1.10.Final"
       )
   }
   ~~~

   

5. 代码编写

   ~~~java
   /**
    * 编写主类
    */
   public class HttpNettyServer {
       public static void main(String[] args) throws InterruptedException {
           //创建两个事件组，一个用来调度，一个用来真正运行程序
           EventLoopGroup bossGroup = new NioEventLoopGroup();
           EventLoopGroup workerGroup = new NioEventLoopGroup();
           try{
               //创建服务启动器
               ServerBootstrap bootServer = new ServerBootstrap();
               bootServer.group(bossGroup,workerGroup)
                       .channel(NioServerSocketChannel.class)
                       .childHandler(new TestChannelInitializer());
   
               //绑定端口号
               ChannelFuture future = bootServer.bind(8899).sync();
               System.out.println("服务已启动");
               future.channel().closeFuture().sync();
           }finally {
               //关闭资源
               bossGroup.shutdownGracefully();
               workerGroup.shutdownGracefully();
           }
       }
   }
   
   public class TestChannelInitializer extends ChannelInitializer<SocketChannel> {
       @Override
       protected void initChannel(SocketChannel ch) throws Exception {
           ChannelPipeline pipeline = ch.pipeline();
           pipeline.addLast("httpServerCodec",new HttpServerCodec());
           pipeline.addLast("testHttpHandler",new TestHttpHandler());
       }
   }
   //编写handler处理请求和响应
   public class TestHttpHandler extends SimpleChannelInboundHandler<HttpObject> {
       //读取客户端的请求，并响应
       @Override
       protected void channelRead0(ChannelHandlerContext ctx, HttpObject msg) throws Exception {
           if (msg instanceof HttpRequest) {
               HttpRequest request = (HttpRequest) msg;
               String uri = request.uri();
               if(uri.equals("/favicon.ico")){
                   ctx.channel().close();
                   return;
               }
               System.out.println("请求的uri:" + uri);
               HttpMethod method = request.method();
               System.out.println("请求的方法："+method);
               //设置响应内容
               ByteBuf content = Unpooled.copiedBuffer("hello world", CharsetUtil.UTF_8);
               //进行响应
               FullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK, content);
               //设置头信息
               response.headers().set(HttpHeaderNames.CONTENT_TYPE, "text/plain");
               response.headers().set(HttpHeaderNames.CONTENT_LENGTH, content.readableBytes());
               ctx.writeAndFlush(response);
   
               ctx.channel().close();
           }
       }
   
       //生命周期的方法，测试用
       @Override
       public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
           System.out.println("channelRegistered");
           super.channelRegistered(ctx);
       }
   
       @Override
       public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
           System.out.println("channelUnregistered");
           super.channelUnregistered(ctx);
       }
   
       @Override
       public void channelActive(ChannelHandlerContext ctx) throws Exception {
           System.out.println("channelActive");
           super.channelActive(ctx);
       }
   
       @Override
       public void channelInactive(ChannelHandlerContext ctx) throws Exception {
           System.out.println("channelInactive");
           super.channelInactive(ctx);
       }
   
       @Override
       public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
           System.out.println("channelReadComplete");
           super.channelReadComplete(ctx);
       }
   }
   ~~~

   

6. 测试

   浏览器输入：http://localhost:8899

### 入门二：基于socke协议

1. 需求：创建一个聊天室，服务器转发每个客户端发送的消息和每个客户端上下线的消息给各个客户端。

   客户端：读取控制台标准输入进行聊天，接收有服务器转发的消息。

2. 代码实现

   * 服务器

   ~~~java
       public class ChatServer {
           public static void main(String[] args) throws InterruptedException {
               EventLoopGroup bossGroup = new NioEventLoopGroup();
               EventLoopGroup workerGroup = new NioEventLoopGroup();
               try{
                   ServerBootstrap server = new ServerBootstrap();
                   server.group(bossGroup,workerGroup).handler(new LoggingHandler(LogLevel.INFO))
                           .childHandler(new ChatServerInisializer()).channel(NioServerSocketChannel.class);
   
                   ChannelFuture future = server.bind(8899).sync();
                   future.channel().closeFuture().sync();
               }finally {
                   bossGroup.shutdownGracefully();
                   workerGroup.shutdownGracefully();
               }
           }
       }
   
   
       public class ChatServerInisializer extends ChannelInitializer<SocketChannel> {
           @Override
           protected void initChannel(SocketChannel ch) throws Exception {
               ChannelPipeline pipeline = ch.pipeline();
               pipeline.addLast(new DelimiterBasedFrameDecoder(4096, Delimiters.lineDelimiter()));
               pipeline.addLast(new StringDecoder(CharsetUtil.UTF_8));
               pipeline.addLast(new StringEncoder(CharsetUtil.UTF_8));
               pipeline.addLast(new MyChatServerHandler());
           }
       }
   
       public class MyChatServerHandler extends SimpleChannelInboundHandler<String> {
           //创建一个channelGroup 用于存储所用连接服务的channel
           private static ChannelGroup channelGroup = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);
           @Override
           protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
               //获取当前向服务器发送消息的channel
               Channel channel = ctx.channel();
               channelGroup.forEach(ch ->{
                   if(channel != ch){
                       ch.writeAndFlush(channel.remoteAddress()+"发送的消息："+msg+"\n");
                   }else{
                       ch.writeAndFlush("【自己】"+msg+"\n");
                   }
               });
   
           }
   
   
           @Override
           public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
               Channel channel = ctx.channel();
               channelGroup.writeAndFlush("【服务器】_"+channel.remoteAddress()+"加入\n");
               channelGroup.add(channel);
   
           }
   
           @Override
           public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
               Channel channel = ctx.channel();
               channelGroup.writeAndFlush("【服务器】_"+channel.remoteAddress()+"加入\n");
               //可以自动处理不用手动写
               //channelGroup.remove(channel);
           }
   
           @Override
           public void channelActive(ChannelHandlerContext ctx) throws Exception {
               Channel channel = ctx.channel();
               System.out.println(channel.remoteAddress()+"上线");
           }
   
           @Override
           public void channelInactive(ChannelHandlerContext ctx) throws Exception {
               Channel channel = ctx.channel();
               System.out.println(channel.remoteAddress()+"下线");
           }
   
           @Override
           public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
               cause.printStackTrace();
               ctx.close();
           }
       }
   ~~~

   

   * 客户端代码

     ~~~java
     public class ChatClent {
         public static void main(String[] args) throws Exception {
             EventLoopGroup eventLoop = new NioEventLoopGroup();
             try{
                 Bootstrap client = new Bootstrap();
                 client.group(eventLoop).channel(NioSocketChannel.class)
                         .handler(new ChatClentInisializer());
                 Channel channel = client.connect("localhost",8899).sync().channel();
     
                 BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
                 for(;;){
                     channel.writeAndFlush(reader.readLine()+"\r\n");
                 }
             }finally{
                 eventLoop.shutdownGracefully();
             }
         }
     }
     
     public class ChatClentInisializer extends ChannelInitializer<SocketChannel> {
         @Override
         protected void initChannel(SocketChannel ch) throws Exception {
             ChannelPipeline pipeline = ch.pipeline();
             pipeline.addLast(new DelimiterBasedFrameDecoder(4096, Delimiters.lineDelimiter()));
             pipeline.addLast(new StringDecoder(CharsetUtil.UTF_8));
             pipeline.addLast(new StringEncoder(CharsetUtil.UTF_8));
             pipeline.addLast(new ChatClientHandler());
         }
     }
     
     public class ChatClientHandler extends SimpleChannelInboundHandler<String> {
         @Override
         protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
             System.out.println("client接收到数据"+msg);
         }
     }
     ~~~

     

### 入门三：websocket长连接

1. 代码实现

   ~~~java
   public class MyServer {
       public static void main(String[] args) throws InterruptedException {
           EventLoopGroup bossGroup = new NioEventLoopGroup();
           EventLoopGroup workerGroup = new NioEventLoopGroup();
           try{
               ServerBootstrap server = new ServerBootstrap();
               server.group(bossGroup,workerGroup).handler(new LoggingHandler(LogLevel.INFO))
                       .childHandler(new WebSocketInitializer()).channel(NioServerSocketChannel.class);
               ChannelFuture future = server.bind(8888).sync();
               future.channel().closeFuture().sync();
           }finally {
               bossGroup.shutdownGracefully();
               workerGroup.shutdownGracefully();
           }
       }
   }
   
   public class WebSocketInitializer extends ChannelInitializer<SocketChannel> {
       @Override
       protected void initChannel(SocketChannel ch) throws Exception {
           ChannelPipeline pipeline = ch.pipeline();
           pipeline.addLast(new HttpServerCodec());
           pipeline.addLast(new ChunkedWriteHandler());
           //拼接成完整的http请求
           pipeline.addLast(new HttpObjectAggregator(8129));
           //处理frame帧
           pipeline.addLast(new WebSocketServerProtocolHandler("/ws"));
           pipeline.addLast(new TextWebSocketHandler());
       }
   }
   
   public class TextWebSocketHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {
       @Override
       protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame msg) throws Exception {
           System.out.println("收到消息："+msg.text());
           ctx.channel().writeAndFlush(new TextWebSocketFrame("服务器时间："+ LocalDateTime.now()));
       }
   
       @Override
       public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
           System.out.println("handAdded"+ctx.channel().id().asLongText());
       }
   
       @Override
       public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
           System.out.println("handlerRemoved:"+ctx.channel().id().asLongText());
       }
   
       @Override
       public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
           System.out.println("发生了异常");
           ctx.close();
       }
   }
   ~~~

   

2. 测试代码

   ~~~html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <title>WebSocket客户端</title>
   </head>
   <body>
   <script type="text/javascript">
   
       var socket;
       if(window.WebSocket){
           socket = new WebSocket("ws://localhost:8888/ws")
           socket.onmessage=function (ev) {
               var ta = document.getElementById("responseText")
               ta.value=ta.value+"\n"+ev.data;
           }
           socket.onopen = function (ev) {
               var ta = document.getElementById("responseText")
               ta.value="连接开启！"
           }
   
           socket.onclose= function (ev) {
               var ta = document.getElementById("responseText")
               ta.value=ta.value+"\n"+"连接关闭！"
           }
       }else{
           alert('浏览器不支持websocket')
       }
   
       function send(message){
           if(!window.WebSocket){
               return;
           }
           if(socket.readyState==WebSocket.OPEN){
               socket.send(message);
           }else{
               alert("连接尚未开启！")
           }
       }
   
   </script>
   
   
   <form onsubmit="return false;">
       <textarea name="message" style="width:400px;height: 200px;"></textarea>
   
       <input type="button" value="发送数据" onclick="send(this.form.message.value)">
   
       <h3>服务端输出：</h3>
   
       <textarea id="responseText" style="width: 400px;height: 300px;"></textarea>
   
       <input type="button" onclick="javascript: document.getElementById('responseText').value=''" value="清空内容">
   </form>
   </body>
   </html>
   ~~~

   

# protocol-buffers

## 简介

Protocol buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.

是一个语言无关平台无关用于序列haul结构化数据的框架。

## 下载安装

protoc-3.7.1-win64.zip

配置环境变量

命令：protoc

## 快速入门

### 编码

1. 编写Student.proto文件

   ~~~protobuf
   syntax = "proto2";
   
   package com.study.protobuf;
   option optimize_for = SPEED;
   option java_package = "com.study.protobuf";
   option java_outer_classname = "DataInfo";
   
   message Student{
       required string name = 1;
       optional int32 age = 2;
       optional string address = 3;
   }
   ~~~

   

2. 添加包

   ~~~groovy
   "com.google.protobuf:protobuf-java:3.7.1"
   ~~~

   

3. 编译成java文件

   protoc -java_out=src/main/java src/protocobuf/Student.proto

3. 测试

   ~~~java
   public class DataInfoTest {
       public static void main(String[] args) throws InvalidProtocolBufferException {
           DataInfo.Student student = DataInfo.Student.newBuilder()
                   .setName("xiaoqiang")
                   .setAge(10)
                   .setAddress("beijing").build();
           byte[] bytes = student.toByteArray();
           DataInfo.Student student1 = DataInfo.Student.parseFrom(bytes);
           System.out.println(student1.getName());
           System.out.println(student1.getAge());
           System.out.println(student1.getAddress());
       }
   }
   ~~~

   

### 总结

* java_outer_classname :定义的一个整个外部类的名称，例子中外部类的名称是AddressBookProtos。

  ​					如果不指定，就默认以文件名转换成驼峰的形式作为类的名字。

* java_package：定义的生成java类的包名。

* 类型：`bool`, `int32`, `float`, `double`, and `string`，自定义。

* required：必须存在，否则会抛出异常。一般不使用。
* optional：可以存在，也可以不设定，不设定为默认值。常用
* repeated：可以重复任一多次。可以认为是一个动态的数组。
* required string name = 1：其中1是一个在二进制中的一个唯一标记。1-15：最为常用的元素，可以加快传输，15以上，可用于不常用的元素。

### 注意

* 不支持java类似的继承操作。



## protobuf和netty的整合

1. 编写.proto文件

   ~~~protobuf
   syntax = "proto2";
   
   package com.study.protobuf;
   option optimize_for = SPEED;
   option java_package = "com.study.netty.protobuf";
   option java_outer_classname = "MyDataInfo";
   
   message MyMessage{
       enum DataType{
           PERSON =0;
           CAT=1;
       }
       required DataType data_type =2;
       //oneof 里面的属性只有一个可以同时被赋值，另一个就会被清空。
       oneof data_body{
           Person person=3;
           Cat cat=4;
       }
   }
   
   message Person{
       optional string name = 1;
       optional int32 age = 2;
       optional string address = 3;
   }
   
   message Cat{
       optional string name =1;
       optional int32 age = 2;
   }
   ~~~

   

2. 编译成java文件

   ~~~protobuf
   protoc --java_out=src/main/java src/protocobuf/Person.proto
   ~~~

   

3. 客户端代码

   ~~~java
   public class MyClient {
       public static void main(String[] args) throws InterruptedException {
           EventLoopGroup eventLoop = new NioEventLoopGroup();
           try{
               Bootstrap client = new Bootstrap();
               client.group(eventLoop).channel(NioSocketChannel.class)
                       .handler(new MyClientProtobufInisializer());
               ChannelFuture future = client.connect("localhost",8899).sync();
               future.channel().closeFuture().sync();
           }finally{
               eventLoop.shutdownGracefully();
           }
       }
   }
   public class MyClientProtobufInisializer extends ChannelInitializer<SocketChannel> {
       @Override
       protected void initChannel(SocketChannel ch) throws Exception {
           ChannelPipeline pipeline = ch.pipeline();
           pipeline.addLast(new ProtobufVarint32FrameDecoder());
           pipeline.addLast(new ProtobufDecoder(MyDataInfo.MyMessage.getDefaultInstance()));
           pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
           pipeline.addLast(new ProtobufEncoder());
   
           pipeline.addLast(new MyClientHandler());
       }
   }
   
   public class MyClientHandler extends SimpleChannelInboundHandler<MyDataInfo.Person> {
       @Override
       protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo.Person msg) throws Exception {
   
       }
   
       @Override
       public void channelActive(ChannelHandlerContext ctx) throws Exception {
           int rondom = new Random().nextInt(2);
           MyDataInfo.MyMessage myMessage=null;
           System.out.println("rondom:"+rondom);
           if(rondom==0){
               myMessage = MyDataInfo.MyMessage.newBuilder()
                       .setDataType(MyDataInfo.MyMessage.DataType.PERSON)
                       .setPerson(MyDataInfo.Person.newBuilder()
                       .setName("一个人").setAddress("beijing").setAge(10).build())
                       .build();
   
           }else{
               myMessage=MyDataInfo.MyMessage.newBuilder()
                       .setDataType(MyDataInfo.MyMessage.DataType.CAT)
                       .setCat(MyDataInfo.Cat.newBuilder()
                               .setName("一只猫").setAge(10).build())
                       .build();
           }
   
           ctx.channel().writeAndFlush(myMessage);
       }
   }
   ~~~

   

4. 服务器端代码

   ~~~java
   public class MyServer {
       public static void main(String[] args) throws InterruptedException {
           EventLoopGroup bossGroup = new NioEventLoopGroup();
           EventLoopGroup workerGroup = new NioEventLoopGroup();
           try{
               ServerBootstrap server = new ServerBootstrap();
               server.group(bossGroup,workerGroup).handler(new LoggingHandler(LogLevel.INFO))
                       .childHandler(new MyprotobufInisializer()).channel(NioServerSocketChannel.class);
               ChannelFuture future = server.bind(8899).sync();
               future.channel().closeFuture().sync();
           }finally {
               bossGroup.shutdownGracefully();
               workerGroup.shutdownGracefully();
           }
       }
   }
   public class MyprotobufInisializer extends ChannelInitializer<SocketChannel> {
       @Override
       protected void initChannel(SocketChannel ch) throws Exception {
           ChannelPipeline pipeline = ch.pipeline();
           pipeline.addLast(new ProtobufVarint32FrameDecoder());
           pipeline.addLast(new ProtobufDecoder(MyDataInfo.MyMessage.getDefaultInstance()));
           pipeline.addLast(new ProtobufVarint32LengthFieldPrepender());
           pipeline.addLast(new ProtobufEncoder());
           pipeline.addLast(new MyTestServerHandler());
       }
   }
   
   public class MyTestServerHandler extends SimpleChannelInboundHandler<MyDataInfo.MyMessage> {
       @Override
       protected void channelRead0(ChannelHandlerContext ctx, MyDataInfo.MyMessage msg) throws Exception {
           System.out.println("channelRead0....");
           MyDataInfo.MyMessage.DataType dataType =msg.getDataType();
           System.out.println("dataType:"+dataType);
   
           if(dataType== MyDataInfo.MyMessage.DataType.PERSON){
               System.out.println(msg.getPerson().getName());
               System.out.println(msg.getPerson().getAddress());
               System.out.println(msg.getPerson().getAge());
           }else{
               System.out.println(msg.getCat().getName());
               System.out.println(msg.getCat().getAge());
           }
       }
   }
   ~~~

   