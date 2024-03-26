---
title: art
date: 2024-03-26 18:21:10
tags: test
cover: /assets/sea.jpg
banner: /assets/talk.png
categories: [设计开发, iOS开发]

---

  SPI机制的缺陷

通过上面的解析，可以发现，我们使用SPI机制的缺陷：

- 不能按需加载，需要遍历所有的实现，并实例化，然后在循环中才能找到我们需要的实现。如果不想用某些实现类，或者某些类实例化很耗时，它也被载入并实例化了，这就造成了浪费。
- 获取某个实现类的方式不够灵活，只能通过 Iterator 形式获取，不能根据某个参数来获取对应的实现类。
- 多个并发多线程使用 ServiceLoader 类的实例是不安全的。



<!--more-->



当我把else注释掉，编译不会通过：`The local variable response may not have been initializedJava(536870963)` 

```java
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;

public class TestServer {

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(9090);
        System.out.println("9090端口正在被监听...");

        while (true) {

            Socket accept = serverSocket.accept();
            System.out.println("客户端连接成功: " + accept.getInetAddress());

            // Read client's request
            InputStream inputStream = accept.getInputStream();

            byte[] requestBytes = new byte[1024];
            int readLen = inputStream.read(requestBytes);
            String clientRequest = new String(requestBytes, 0, readLen);
            System.out.println("客户端请求内容: " + clientRequest);

            // Extract the requested path from the request
            String[] requestLines = clientRequest.split("\r\n");
            String[] requestLineParts = requestLines[0].split(" ");
            String path = requestLineParts[1];

            // Handle the request based on the path
            String response;
            if ("/login".equals(path)) {
                response = "HTTP/1.1 200 OK\r\n\r\nWelcome to the login page!";
            }
            //当我把else注释掉，编译不会通过：The local variable response may not have been initializedJava(536870963)
            // else {

            // response = "HTTP/1.1 404 Not Found\r\n\r\nPage not found!";

            // }


            // Send the response back to the client

            OutputStream outputStream = accept.getOutputStream();

            outputStream.write(response.getBytes());

  

            // Close the streams and socket

            inputStream.close();

            outputStream.close();
            accept.close();
        }
    }
}
```

因为你把else注释掉，response就可能存在没有赋初始值的情况。

java语言规定局部变量必须显示初始化。全局变量保存在堆中，在创建对象时就必须确定变量的大小，所以jvm会初始化全局变量。而局部变量保存在方法栈中，编译器无法确定局部变量的大小，更好的选择是程序员手动进行初始化。

除了在使用局部变量之前手动赋一个初始值，也可以使用`if else`来确保局部变量在使用之前可以被初始化。这也就是为什么你有时候不手动给局部变量赋值也能使用，因为已经用条件语句确保它有值。就像上面的例子。