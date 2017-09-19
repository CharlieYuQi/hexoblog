---
layout: post
title: Netty连接池ChannelPool,FixedChannelPool应用
date: 2017-09-18
categories: IT
tags: [IT,Netty,ChannelPool]
---

# 问题描述
> Netty is an asynchronous event-driven network application framework
for rapid development of maintainable high performance protocol servers & clients.

如今越来越多的应用采用Netty作为服务端高性能异步通讯框架，对于客户端而言，大部分需求只需和服务端建立一条链接收发消息。但如果客户端需要和服务端建立多条链接的例子就比较少了。
最简单的实现就是一个for循环，建立多个NioEventLoopGroup与服务端交互。另外还有如果要和多个服务端进行交互又该如何解决。

其实Netty从4.0版本就提供了连接池ChannelPool，可以解决与多个服务端交互以及与单个服务端建立连接池的问题。

![](http://ww1.sinaimg.cn/large/e5aac86bgy1fi3zjslhvaj20d30210sm.jpg)

# 服务端程序
首先我们完成服务端的代码，用户测试客户端的连接池。服务端不需要做任何特殊处理，代码如下。
```java
package tk.yuqibit.nio.pool;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.DelimiterBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.util.concurrent.DefaultEventExecutorGroup;

/**
 * Created by YuQi on 2017/7/31.
 */
public class NettyServer {
    public void run(final int port) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
                            ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, delimiter))
                                    .addLast(new StringDecoder()).addLast(new StringEncoder())
                                    .addLast(new DefaultEventExecutorGroup(8),
                                            new NettyServerHandler());
                        }
                    }).option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true);
            ChannelFuture future = b.bind(port).sync();
            future.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        int port = 8080;
        if (args.length > 0) {
            try {
                port = Integer.parseInt(args[0]);
            } catch (NumberFormatException e) {
                e.printStackTrace();
            }
        }
        new NettyServer().run(port);
    }
}

```
服务端Handler，打印客户端发送的字符串，并回复另一个字符串
```java
package tk.yuqibit.nio.pool;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * Created by YuQi on 2017/7/31.
 */
public class NettyServerHandler extends SimpleChannelInboundHandler<Object> {
    static AtomicInteger count = new AtomicInteger(1);

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        System.out.println("channelActived");
        super.channelActive(ctx);
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.println(count.getAndIncrement() + ":" + body);
        ctx.writeAndFlush("Welcome to Netty.$_");
    }
}

```

# 客户端代码
重点是以下客户端代码，首先要实现自己的```ChannelPoolHandler```,主要是```channelCreated```,当链接创建的时候添加channelhandler。
```java
package tk.yuqibit.nio.pool;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.Channel;
import io.netty.channel.pool.ChannelPoolHandler;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.DelimiterBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;

/**
 * Created by YuQi on 2017/7/31.
 */
public class NettyChannelPoolHandler implements ChannelPoolHandler {
    @Override
    public void channelReleased(Channel ch) throws Exception {
        System.out.println("channelReleased. Channel ID: " + ch.id());
    }

    @Override
    public void channelAcquired(Channel ch) throws Exception {
        System.out.println("channelAcquired. Channel ID: " + ch.id());
    }

    @Override
    public void channelCreated(Channel ch) throws Exception {
        ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
        System.out.println("channelCreated. Channel ID: " + ch.id());
        SocketChannel channel = (SocketChannel) ch;
        channel.config().setKeepAlive(true);
        channel.config().setTcpNoDelay(true);
        channel.pipeline()
                .addLast(new DelimiterBasedFrameDecoder(1024, delimiter))
                .addLast(new StringDecoder()).addLast(new StringEncoder()).addLast(new NettyClientHander());

    }
}

```
客户端Handler,打印服务端的response。
```java
package tk.yuqibit.nio.pool;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * Created by YuQi on 2017/7/31.
 */
public class NettyClientHander extends ChannelInboundHandlerAdapter {
    static AtomicInteger count = new AtomicInteger(1);

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println(count.getAndIncrement() + ":" + msg);
    }
}

```
客户端实现连接池，其中ChannelPoolMap可用于与多个服务端建立链接，本例中采用FixedChannelPool建立与单个服务端最大连接数为2的连接池。在main函数里通过向连接池获取channel发送了十条消息。
```java
package tk.yuqibit.nio.pool;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.Channel;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.pool.AbstractChannelPoolMap;
import io.netty.channel.pool.ChannelPoolMap;
import io.netty.channel.pool.FixedChannelPool;
import io.netty.channel.pool.SimpleChannelPool;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.util.concurrent.Future;
import io.netty.util.concurrent.FutureListener;

import java.net.InetSocketAddress;

/**
 * Created by YuQi on 2017/7/31.
 */
public class NettyPoolClient {
    final EventLoopGroup group = new NioEventLoopGroup();
    final Bootstrap strap = new Bootstrap();
    InetSocketAddress addr1 = new InetSocketAddress("127.0.0.1", 8080);
    InetSocketAddress addr2 = new InetSocketAddress("10.0.0.11", 8888);

    ChannelPoolMap<InetSocketAddress, SimpleChannelPool> poolMap;
    public void build() throws Exception {
        strap.group(group).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, true)
                .option(ChannelOption.SO_KEEPALIVE, true);

        poolMap = new AbstractChannelPoolMap<InetSocketAddress, SimpleChannelPool>() {
            @Override
            protected SimpleChannelPool newPool(InetSocketAddress key) {
                return new FixedChannelPool(strap.remoteAddress(key), new NettyChannelPoolHandler(), 2);
            }
        };
    }

    public static void main(String[] args) throws Exception {
        NettyPoolClient client = new NettyPoolClient();
        client.build();
        final String ECHO_REQ = "Hello Netty.$_";
        for (int i = 0; i < 10; i++) {
            // depending on when you use addr1 or addr2 you will get different pools.
            final SimpleChannelPool pool = client.poolMap.get(client.addr1);
            Future<Channel> f = pool.acquire();
            f.addListener((FutureListener<Channel>) f1 -> {
                if (f1.isSuccess()) {
                    Channel ch = f1.getNow();
                    ch.writeAndFlush(ECHO_REQ);
                    // Release back to pool
                    pool.release(ch);
                }
            });
        }
    }
}

```
# 输出结果
首先启动服务端，然后启动客户端，for循环里向服务端发送了10条消息。
服务端的输出如下，可以看到总共与服务端建立了两个channel，收到10条消息。
```
channelActived
channelActived
1:Hello Netty.
2:Hello Netty.
3:Hello Netty.
4:Hello Netty.
5:Hello Netty.
6:Hello Netty.
7:Hello Netty.
8:Hello Netty.
9:Hello Netty.
10:Hello Netty.
```
客户端输入如下，可以看到channelCreated了两次，剩下都是从连接池里请求连接和释放连接
```java
channelCreated. Channel ID: ea8504a8
channelCreated. Channel ID: 77c8857b
channelReleased. Channel ID: ea8504a8
channelReleased. Channel ID: 77c8857b
channelAcquired. Channel ID: ea8504a8
channelAcquired. Channel ID: 77c8857b
channelReleased. Channel ID: ea8504a8
channelReleased. Channel ID: 77c8857b
channelAcquired. Channel ID: 77c8857b
channelAcquired. Channel ID: ea8504a8
channelReleased. Channel ID: ea8504a8
channelAcquired. Channel ID: ea8504a8
channelReleased. Channel ID: 77c8857b
channelReleased. Channel ID: ea8504a8
channelAcquired. Channel ID: 77c8857b
channelAcquired. Channel ID: ea8504a8
channelReleased. Channel ID: 77c8857b
channelAcquired. Channel ID: 77c8857b
channelReleased. Channel ID: ea8504a8
channelReleased. Channel ID: 77c8857b
1:Welcome to Netty.
2:Welcome to Netty.
3:Welcome to Netty.
4:Welcome to Netty.
5:Welcome to Netty.
6:Welcome to Netty.
7:Welcome to Netty.
8:Welcome to Netty.
9:Welcome to Netty.
10:Welcome to Netty.
```
原始代码见[GitHub](https://github.com/CharlieYuQi/java-learn/tree/master/nio/src/main/java/tk/yuqibit/nio/pool)
