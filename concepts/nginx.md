# nginx

## 什么是Nginx
Nginx 是俄罗斯人编写的十分轻量级的 HTTP 服务器,Nginx，它的发音为“engine X”，是一个高性能的HTTP和反向代理服务器，同时也是一个 IMAP/POP3/SMTP 代理服务器。Nginx 是由俄罗斯人 Igor Sysoev 为俄罗斯访问量第二的 Rambler.ru 站点开发的，它已经在该站点运行超过两年半了。

到 2013 年，目前有很多国内网站采用 Nginx 作为 Web 服务器，如国内知名的新浪、163、腾讯、Discuz、豆瓣等。据 netcraft 统计，Nginx 排名第 3，约占 15% 的份额(参见：http://news.netcraft.com/archives/category/web-server-survey/ )

Nginx 以事件驱动的方式编写，所以有非常好的性能，同时也是一个非常高效的反向代理、负载均衡服务器。



## 示例

### 反向代理

反向代理应该是Nginx使用最多的功能了，反向代理(Reverse Proxy)方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

简单来说就是真实的服务器不能直接被外部网络访问，所以需要一台代理服务器，而代理服务器能被外部网络访问的同时又跟真实服务器在同一个网络环境，当然也可能是同一台服务器，端口不同而已。

反向代理通过proxy_pass指令来实现。


```shell
server {
    listen       80;
    server_name  localhost;

    location / {
        proxy_pass http://localhost:8081;
        proxy_set_header Host $host:$server_port;
        # 设置用户ip地址
         proxy_set_header X-Forwarded-For $remote_addr;
         # 当请求服务器出错去寻找其他服务器
         proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
    }
}
```

- server: 用于定义服务，http中可以有多个server块

- listen : 指定服务器侦听请求的IP地址和端口，如果省略地址，服务器将侦听所有地址，如果省略端口，则使用标准端口

- server_name : 服务名称，用于配置域名

- location : 用于配置映射路径uri对应的配置，一个server中可以有多个location, location后面跟一个uri,可以是一个正则表达式, / 表示匹配任意路径, 当客户端访问的路径满足这个uri时就会执行location块里面的代码

- proxy_pass : 反向代理，当我们访问localhost的时候，就相当于访问 localhost:8081了

### 内置变量
nginx的配置文件中可以使用的内置变量以美元符$开始，也有人叫全局变量。其中，部分预定义的变量的值是可以改变的。

- $args ：#这个变量等于请求行中的参数，同$query_string

- $content_length ：请求头中的Content-length字段。

- $content_type ：请求头中的Content-Type字段。

- $document_root ：当前请求在root指令中指定的值。

- $host ：请求主机头字段，否则为服务器名称。

- $http_user_agent ：客户端agent信息

- $http_cookie ：客户端cookie信息

- $limit_rate ：这个变量可以限制连接速率。

- $request_method ：客户端请求的动作，通常为GET或POST。

- $remote_addr ：客户端的IP地址。

- $remote_port ：客户端的端口。

- $remote_user ：已经经过Auth Basic Module验证的用户名。

- $request_filename ：当前请求的文件路径，由root或alias指令与URI请求生成。

- $scheme ：HTTP方法（如http，https）。

- $server_protocol ：请求使用的协议，通常是HTTP/1.0或HTTP/1.1。

- $server_addr ：服务器地址，在完成一次系统调用后可以确定这个值。

- $server_name ：服务器名称。

- $server_port ：请求到达服务器的端口号。

- $request_uri ：包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。

- $uri ：不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。

- $document_uri ：与$uri相同

### 补充

假设访问springBoot 启动的1个服务 Ip:192.168.255.255:10010 使用nginx代理后项目的请求流程理解：

1、浏览器发起请求 如：http://www.wuyou.com 浏览器就会进行域名解析，转换成IP+端口号进行访问，所以浏览器将找到hosts 文件中的对应关系，如果找不到就到中央服务器那找（肯定能找到照只要你的网站做了备案并通过），所以你只需要增加一个该文件的配置即可：192.168.255.255 http://www.wuyou.com

2、这样当你访问 http://www.wuyou.com 该域名自动会被解析成192.168.255.255 该IP

3、在HTTP协议中，默认端口号是80 端口，所以你访问域名时IP：port 是192.168.255.255：80 这与我们实际想访问的服务器端口不匹配，这时候就需要用到nginx 了

4、在nginx 中修改conf 配置文件，监听80端口并配置类似路由的配置即可实现请求的转发。见以下配置：

```shell
server {
        listen       80; #监听的端口号
        server_name  www.wuyou.com; #域名
 
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        location / {
            proxy_pass http://192.168.255.255:10010; #转发的地址
            proxy_connect_timeout 600; #超时
            proxy_read_timeout 600;
        }
    }
```

以上就是整个执行流程。

## Reference
[Nginx](https://zhuanlan.zhihu.com/p/389438482)


















