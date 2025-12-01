# 记录一次线上链接半开问题

## 链接半开问题

TCP连接的一方已经关闭或崩溃，但另一方并不知道，还以为连接是正常的。所以两端状态不一致：

* A 端：连接已关闭、或机器挂了、网络断了
* B 端：连接还在（ESTABLISHED），继续 write()/flush()

这种不一致就叫半开连接。

## 线上问题描述

核心数据和携号转网程序里的数据不一致。

原因：携转转网部署之后，核心重启了，那么下次发送的包会识别为成功，但是实际上失败了，而基于java代码，在有异常的情况下会重试，但异常首次下发的数据包就会丢失，导致核心数据域携转转网里面的数据不一致。

<figure><img src="../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

## 源码如下

```
@Data
@Slf4j
public class KernelQueue implements Closeable {

    private final ArrayBlockingQueue<PortabFileRead> queue = new ArrayBlockingQueue<>(500000);
    private final JkernelHost jkernelHost;
    private Socket socket;
    private OutputStream outputStream;

    public KernelQueue(JkernelHost jkernelHost) {
        this.jkernelHost = jkernelHost;
        socket = new Socket();
        try {
            socket.setKeepAlive(true);
            socket.connect(new InetSocketAddress(jkernelHost.getHost(),jkernelHost.getPort()),60000);
            outputStream = socket.getOutputStream();
        } catch (IOException e) {
            log.error("连接远程服务器异常");
        }
        new Thread(()->{
            for (;;){
                try {
                    PortabFileRead take = queue.take();
                    Integer type = take.getType();
                    List<Portab> portabs = take.getPortabs();
                    for (Portab portab : portabs) {
                        PortabCmdMsg send;
                        if (type == 0){
                            send =  PortabCmdMsg.delete(portab.getMobile());
                        }else {
                            send = PortabCmdMsg.add(portab.getMobile(), portab.getNetwork(), portab.getProvId());
                        }
                        for (int i = 0; i < 120; i++) {
                            try {
                                //问题出在这个过程
                                outputStream.write(send.getBytes());
                                outputStream.flush();
                                break;
                            }catch (Exception e){
                                log.error("发往核心[{}:{}]重连异常",jkernelHost.getHost(),jkernelHost.getPort(),e);
                                try {
                                    socket = new Socket();
                                    socket.setKeepAlive(true);
                                    socket.connect(new InetSocketAddress(jkernelHost.getHost(),jkernelHost.getPort()),60000);
                                    outputStream = socket.getOutputStream();
                                } catch (IOException ex) {
                                    log.error("发往核心[{}:{}]重连异常",jkernelHost.getHost(),jkernelHost.getPort(),ex);
                                    TimeUnit.SECONDS.sleep(30);
                                }
                            }
                            if (i == 119){
                                log.error("发往核心[{}:{}]重试n次异常,未更新数据mobile:{},type:{},net:{}",jkernelHost.getHost()
                                        ,jkernelHost.getPort(),portab.getMobile(),portab.getType(),portab.getNetwork());
                            }
                        }
                    }

                } catch (InterruptedException e) {
                    log.error("通知核心[{}:{}]消费队列出现异常，异常信息：",jkernelHost.getHost(),jkernelHost.getPort(),e);
                }
            }
        },"kernel").start();
    }

    public boolean put(PortabFileRead portabFileRead) {
        return queue.add(portabFileRead);
    }


    @Override
    public void close() throws IOException {
        if (outputStream != null){
            outputStream.close();
        }
        if (socket != null){
            if (!socket.isClosed()){
                socket.close();
            }
        }
    }
}
```

根源就在于如下方法

```
outputStream.write(send.getBytes());
outputStream.flush();
```

写失败只有两种情况（这是 TCP 协议保证的）

1. 对端明确返回 RST

client → server（发送数据包）

server 发现这个连接已经不存在（比如 socket 已关闭）

server → client：RST

客户端 OS 内核把 socket 状态标记为 CLOSED，

下一次 Java write() 直接抛异常：java.net.SocketException: Connection reset



2. TCP 重传超时

如果对端死机 / 宕机 / 网络断掉，不能返回 ACK 或 RSTclient →（无人接收）OS 按照 TCP RTO（重传定时器）进行重传：常见序列如下：

| 重传序号  | 等待时间（指数退避）       |
| ----- | ---------------- |
| 第 1 次 | RTO（300ms\~1s之间） |
| 第 2 次 | 2 × RTO          |
| 第 3 次 | 4 × RTO          |
| 第 4 次 | 8 × RTO          |
| ...   | 最大可持续几分钟         |

直到超过：

* /proc/sys/net/ipv4/tcp\_retries2（Linux）
* 默认是 15（大概 13\~30 分钟）

才会本地报错：ETIMEDOUT java.net.SocketException: connection timed out

关键点：write() 不是发送，它只是写本地发送缓冲区

## 本地测试结果日志截图和说明

<figure><img src="../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

第一个发送成功是在服务端正常的情况下发送的

第二个发送成功是在服务器已经关闭情况下，客户端仍然使用旧的outputStream尝试发送，但这个只是写入缓存当中，所以在java看来是成功的，但是实际上已经是发不出去了。

可以参考抓包结果

<figure><img src="../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

这个抓包结果的1126是第二条发送数据，实际上是发送失败的，对端返回了RST，这个时候系统会识别到链接有问题，java再次写入就可以发现IO异常了

## 解决方案

1. 不维护一个长连接，在批量任务之前构建新的连接，虽然断开的情况也会发送半链接，但是异常之后这批数据重新发送就可以了
2. 服务端修改配置，关闭退出情况下直接发送RST数据包
3. 维护客户端服务端心跳数据（客户端服务端都需要修改）
