# Java socket 多线程任务

## 一、问题引出

1、我们上篇文章写了一个最简单的socket服务端，但这个服务端还有待优化。为了测试问题，我们在写回给客户端数据之前，让程序睡眠5秒，表示我们这个业务需要处理5秒才能完成，然后我们通过浏览器测试一下这个服务端。服务端代码如下：

```
package com.suncy.article.article2;

import org.springframework.boot.logging.java.SimpleFormatter;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;
import java.text.SimpleDateFormat;
import java.util.Date;


public class HttpTestServer {
    public static void main(String[] args) throws IOException, InterruptedException {
        //1、创建服务端Socket对象，绑定端口号。
        ServerSocket ss = new ServerSocket(8000);

        //2、服务端需要一直等待连接状态
        while (true) {

            //3、调用accept()方法来侦听端口号是否有连接请求，拿到与客户端通信的socket句柄，此接口会阻塞在这里，指到有客户端连接
            Socket s = ss.accept();

            //4、打印开始时间
            SimpleDateFormat sdt = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
            System.out.println("接收客户端连接世界：" + sdt.format(new Date()));

            //5、向服务端写入http协议数据
            OutputStream outputStream = s.getOutputStream();
            String httpData = "HTTP/1.1 200 OK\n" +
                    "Server: server" +
                    "Content-Type: text/html\n" +
                    "\r\n" +
                    "hello Client\n";

            //6、睡眠5秒
            Thread.sleep(5000);

            //7、打印结束时间
            outputStream.write(httpData.getBytes(StandardCharsets.UTF_8));

            //8.关闭socket流
            s.close();
            System.out.println("关闭客户端连接世界：" + sdt.format(new Date()));
        }
    }
}
```

我们同时打开两个浏览器页面，输入127.0.0.1:8000访问这个服务端， 然后我们查看一下服务端打印的日志：\


![服务端打印日志](<../.gitbook/assets/image (38).png>)

我们发现竟然有4条请求信息，暂时先不管为什么有4条，但是我们可以看到这个时间是按顺序访问的，并不支持并发。

### 那么为什么只能顺序访问呢？

原因就在于我们编写的服务端程序，在while死循环中，是接收到一个客户端的连接在处理业务，但如果这个客户端连接不关闭的话，我们的代码就不可能再次走到**Socket s = ss.accept()**&#x8FD9;个地方，就不能接收客户端的连接。

## 二、并发的socket服务端程序

为了能支持并发的任务处理，我们需要多线程的编程基础，现在我们重新修改一下这个服务端代码，主要修改地方就是将写入客户端的操作放入到线程当中去处理。

```
package com.suncy.article.article2;

import lombok.SneakyThrows;

import java.io.IOException;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;
import java.text.SimpleDateFormat;
import java.util.Date;

public class HttpMultithreadingServer {
    public static void main(String[] args) throws IOException, InterruptedException {
        //1、创建服务端Socket对象，绑定端口号。
        ServerSocket ss = new ServerSocket(8000);

        //2、服务端需要一直等待连接状态
        while (true) {

            //3、调用accept()方法来侦听端口号是否有连接请求，拿到与客户端通信的socket句柄，此接口会阻塞在这里，指到有客户端连接
            Socket s = ss.accept();

            //开启一个新的线程
            new Thread(new Runnable() {
                @SneakyThrows
                @Override
                public void run() {
                    //4、打印开始时间
                    SimpleDateFormat sdt = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
                    System.out.println("接收客户端连接世界：" + sdt.format(new Date()));

                    //5、向服务端写入http协议数据
                    OutputStream outputStream = s.getOutputStream();
                    String httpData = "HTTP/1.1 200 OK\n" +
                            "Server: server" +
                            "Content-Type: text/html\n" +
                            "\r\n" +
                            "hello Client\n";

                    //6、睡眠5秒
                    Thread.sleep(5000);

                    //7、打印结束时间
                    outputStream.write(httpData.getBytes(StandardCharsets.UTF_8));

                    //8.关闭socket流
                    s.close();
                    System.out.println("关闭客户端连接世界：" + sdt.format(new Date()));
                }
            }).start();
        }
    }
}
```

再次打开两个浏览器，输入127.0.0.1:8000访问，我们查询一下服务端的日志：\


![服务端打印日志](<../.gitbook/assets/image (18).png>)

在同时接收到了客户端请求，并在5秒之后关闭了客户端的连接，但还有一个问题，那就是我们就在浏览器发了两条请求，那么为什么会有4次连接呢？\
我们通过在浏览器中，右键检查，并打开network按钮，我们发现，浏览器还会存在一个favicon.ico的请求，这应该是浏览器自己的逻辑了，但是并不影响我们的测试结果\


![浏览器响应结果](<../.gitbook/assets/image (15).png>)

## 三、总结

通过增加多线程，我们就完成了简单的java并发处理socket的的方法。
