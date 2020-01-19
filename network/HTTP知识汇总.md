
# HTTP知识汇总


### 头信息解读

1. Keep-Alive

    Keep-Alive 是一个通用消息头，允许消息发送者暗示连接的状态，还可以用来设置超时时长和最大请求数。

    通常需要在将Connection头的值设为keep-alive后启用，另外在http2中将会忽略这两个头信息，有其他机制来管理连接。

    示例：
    ```http
    HTTP/1.1 200 OK
    Connection: Keep-Alive
    Content-Encoding: gzip
    Content-Type: text/html; charset=utf-8
    Date: Thu, 11 Aug 2016 15:23:13 GMT
    Keep-Alive: timeout=5, max=1000
    Last-Modified: Mon, 25 Jul 2016 04:32:39 GMT
    Server: Apache
    ```   

1. 
