# Java socket编程

## 一、服务端代码

```
package com.suncy.article.article1;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

public class Server {
    public static void main(String[] args) throws IOException {
        //1、创建服务端Socket对象，绑定端口号。
        ServerSocket ss = new ServerSocket(8000);

        //2、服务端需要一直等待连接状态
        while (true) {

            //3、调用accept()方法来侦听端口号是否有连接请求，拿到与客户端通信的socket句柄，此接口会阻塞在这里，指到有客户端连接
            Socket s = ss.accept();

            //4、有了与客户端连接的socket，通过这个输入输出流完成数据的交互
            InputStream in = s.getInputStream();
            byte[] buf = new byte[1024];
            int len = in.read(buf);
            System.out.println(new String(buf, 0, len));

            //5、向服务端写入数据
            OutputStream outputStream = s.getOutputStream();
            outputStream.write("Hello,Client!".getBytes(StandardCharsets.UTF_8));

            //6.关闭socket流
            s.close();
        }
    }
 }
```

## 二、客户端代码

```
package com.suncy.article.article1;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;

public class Client {
    public static void main(String[] args) throws IOException {
        //1、创建Socket服务，指定连接的地址和端口号，得到socket句柄，通过句柄实现数据的输入和输出
        Socket s = new Socket("127.0.0.1",8000);

        //2、连接成功后，将形成数据通道，即输入和输出IO流，可以通过getInputStream()和getOutputStream()方法获取IO流对象。
        OutputStream out = s.getOutputStream();

        //3、通过输出流向服务端写入数据
        out.write("Hello,TCPServer!".getBytes());

        //4、通过输入流接收服务端数据
        InputStream inputStream = s.getInputStream();
        byte[] buf = new byte[1024];
        int len = inputStream.read(buf);
        System.out.println(new String(buf,0,len));

        //5.关闭客户端连接
        s.close();
    }
}
```

## 三、实例结果

### 客户端执行结果

![客户端执行结果](<../.gitbook/assets/image (12).png>)

### 服务端执行结果

![服务端执行结果](<../.gitbook/assets/image (40).png>)

## 四、socket示例说明

1、这是Java中最简单的socket示例，模式很固定。主要就是起一个服务端，一直监听客户端连接，接收客户端数据之后打印出来，并且回复一条响应消息。客户端连接到服务端数据，发送数据，并打印接收到的服务端的数据。

## 五、问题引出

1、如果我们用浏览器访问socket服务端程序会是什么样子呢？ 打开浏览器，在上面的输入地址输入127.0.0.1:8000试试看&#x20;

![示例图片](<../.gitbook/assets/image (36).png>)

结果会提示该网页无法正常显示，原因就是浏览器使用的协议是Http的，我们返回的数据不符合Http协议格式要求，所以导致浏览器解析不到，为了使浏览器能够解析数据，我们重新修改一下socker服务端。

## 六、 浏览器访问服务端数据

重新编写socket服务端，完成一个最简单的http服务端的任务

```
package com.suncy.article.article1;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

public class HttpServer {
    public static void main(String[] args) throws IOException {
        //1、创建服务端Socket对象，绑定端口号。
        ServerSocket ss = new ServerSocket(8000);

        //2、服务端需要一直等待连接状态
        while (true) {

            //3、调用accept()方法来侦听端口号是否有连接请求，拿到与客户端通信的socket句柄，此接口会阻塞在这里，指到有客户端连接
            Socket s = ss.accept();

            //4、有了与客户端连接的socket，通过这个输入输出流完成数据的交互
            InputStream in = s.getInputStream();
            byte[] buf = new byte[1024];
            int len = in.read(buf);
            System.out.println(new String(buf, 0, len));

            //5、向服务端写入http协议数据
            OutputStream outputStream = s.getOutputStream();
            String httpData = "HTTP/1.1 200 OK\n" +
                    "Server: server" +
                    "Content-Type: text/html\n" +
                    "\r\n"+
                    "hello Client\n";

            outputStream.write(httpData.getBytes(StandardCharsets.UTF_8));

            //6.关闭socket流
            s.close();
        }
    }
}
```

此时，再次通过浏览器访问我们的这个http服务端，得到的结果如下&#x20;

![响应结果](<../.gitbook/assets/image (30).png>)

这时我们的浏览器就能够返回一个字符串，原因就是我们服务端编写的那个响应字符串属于Http协议，浏览器能够解析出来，所以就没有报无法响应的错误了
