---
title: Netty客户端重连的实现
date: 2019-04-02 11:46:06
categories:
- Netty
tags: 
- Netty
- TCP
---

# Netty连接失败时需要重连

针对连接失败时，可以采用``ChannelFutureListener``监听连接状态，通过``channelFuture.channel().eventLoop()``可以的``schedule``方法可以设定重连的延时。

``Client`` 代码示例：
``` JAVA
public class Client  
 {  
   private EventLoopGroup loop = new NioEventLoopGroup();  
   public static void main( String[] args )  
   {  
     new Client().run();  
   }  
   public Bootstrap createBootstrap(Bootstrap bootstrap, EventLoopGroup eventLoop) {  
     if (bootstrap != null) {  
       final MyInboundHandler handler = new MyInboundHandler(this);  
       bootstrap.group(eventLoop);  
       bootstrap.channel(NioSocketChannel.class);  
       bootstrap.option(ChannelOption.SO_KEEPALIVE, true);  
       bootstrap.handler(new ChannelInitializer<SocketChannel>() {  
         @Override  
         protected void initChannel(SocketChannel socketChannel) throws Exception {  
           socketChannel.pipeline().addLast(handler);  
         }  
       });  
       bootstrap.remoteAddress("localhost", 8888);
       bootstrap.connect().addListener(new ConnectionListener(this)); 
     }  
     return bootstrap;  
   }  
   public void run() {  
     createBootstrap(new Bootstrap(), loop);
   }  
 }

```
``ConnectionListener``代码示例：
``` JAVA
public class ConnectionListener implements ChannelFutureListener {  
  private Client client;  
  public ConnectionListener(Client client) {  
    this.client = client;  
  }  
  @Override  
  public void operationComplete(ChannelFuture channelFuture) throws Exception {  
    if (!channelFuture.isSuccess()) {  
      System.out.println("Reconnect");  
      final EventLoop loop = channelFuture.channel().eventLoop();  
      loop.schedule(new Runnable() {  
        @Override  
        public void run() {  
          client.createBootstrap(new Bootstrap(), loop);  
        }  
      }, 1L, TimeUnit.SECONDS);  
    }  
  }  
}
```

# Netty连接断开时需要重连

通过``ChannelListener``的``channelInactive``检测当前连接是否断开，断开后同样设定延时时间并重连。
``MyInboundHandler``代码示例：
``` JAVA
public class MyInboundHandler extends SimpleChannelInboundHandler {  
   private Client client;  
   public MyInboundHandler(Client client) {  
     this.client = client;  
   }  
   @Override  
   public void channelInactive(ChannelHandlerContext ctx) throws Exception {  
     final EventLoop eventLoop = ctx.channel().eventLoop();  
     eventLoop.schedule(new Runnable() {  
       @Override  
       public void run() {  
         client.createBootstrap(new Bootstrap(), eventLoop);  
       }  
     }, 1L, TimeUnit.SECONDS);  
     super.channelInactive(ctx);  
   }  
 }
```