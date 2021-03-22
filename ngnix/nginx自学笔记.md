# Nginx学习纪要





## 一、正向代理

### 1、概念

一个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。

（可以再浏览器中配置代理服务器，比如翻墙工具）



### 2、配置文件

配置文件位置：/usr/local/nginx/conf/nginx.conf

include子配置文件



### 3、配置项：

```ini

server {
				# resolver DNS服务器 
        resolver 192.168.0.1;
        # location 匹配用户访问的资源，并作进一步转交和处理，可用正则表达式匹配 
        location / {
        				# proxy_pass 代理的地址 
								# $http_host 用户访问资源的主机部分
                # $request_uri 用户访问资源的URI部分
                proxy_pass http://$http_host$request_uri;
        }
}
```



### 4、查看端口项

```shell
# 用netstat命令查看监听端口
netstat -antp | grep nginx

# curl 访问
curl -x 192.168.0.112 "https://badu.com" -I
```







## 二、反向代理



### 1、概念

、代理服务器接受网络上的连接请求。转发给内部网络上的服务器，得到结果返回给网络上请求连接的客户端，此时代理服务器对外就表现为一个服务器。



正向代理：代理服务器和客户端处于同一个局域网内；

反向代理：代理服务器和源站则处于同一个局域网内。



### 2、配置文件

配置文件位置：/usr/local/nginx/conf/nginx.conf

include子配置文件

其中，server_name字段用于匹配用户访问资源的主机名称。proxy_pass字段同样表示转交的访问。

与正向代理不同，反向代理是指定了特定的网址。其实也就是限定了访问的对象。指定用户访问www.baidu.com时，只能去百度。如果你把proxy_pass的网址替换为新浪的，那就会出现用户明明要访问百度，却跑到新浪去了。这就是代理劫持了。另外，server_name可以匹配多个主机名，用空格分开即可，也可以用正则表达式匹配主机名。

但是特别要注意一点，对于没有匹配到的访问，nginx会默认走第一条server配置，所以一般我们将第一条server设置为阻止页面。

### 3、配置项：

```ini
server {
        server_name www.baidu.com;
        location / {
                proxy_pass http://www.baidu.com/;
        }
}
```



### 4、查看端口项

```shell
# 用netstat命令查看监听端口
netstat -antp | grep nginx

# curl 访问
curl -x 192.168.0.112 "https://badu.com" -I
```



# 三、Nginx配置



## 1、全局块

```ini
# 并发数量，这个值越大，处理的并发数越多，不过这个值是受限于硬件数
worker_processes  1;

#error_log  logs/error.log  notice;
#user  nobody;

#制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
error_log log/error.log debug;  
```



## 2、events块



```ini

events {
		#设置网路连接序列化，防止惊群现象发生，默认为on
		accept_mutex on;
    #设置一个进程是否同时接受多个网络连接，默认为off
    multi_accept on;  
     
    #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    use epoll; 
    
		# 影响nginx服务器与网络的链接，支持最大的链接数
    worker_connections  1024;
}
```



### 3、Http块

#### 1）http全局块

```ini
http {

#文件引入
include /etc/nginx/conf.d/*.conf;

# mime.types
include       /etc/nginx/mime.types;

#默认类型
default_type  application/octet-stream;

# 超时时间 65秒
keepalive_timeout  65;

#允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
sendfile on;   

#每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
sendfile_max_chunk 100k;  

upstream mysvr {   
	server 127.0.0.1:7878;
	server 192.168.10.121:3333 backup;  #热备
}

#错误页
error_page 404 https://www.baidu.com; 

# gzip 配置
gzip  on;
# gzip http协议版本 gzip_http_version 1.1;
gzip_http_version 1.0;
#若果是IE的话 禁止
gzip_disable 'MSIE [1-6].';
#压缩的文件格式 设置哪压缩种文本文件可参考 conf/mime.types
gzip_types application/javascript text/css text/xml;
# 设置压缩最小单位，小于不压缩		 设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。建议设置成大于1k的字节数，小于1k可能会越压越大。
gzip_min_length 1k;   
# 压缩比
gzip_comp_level 4;
# 缓存比 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。 例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。 4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。
gzip_buffers 4 16k;
# nginx对于静态文件的处理模块，该模块可以读取预先压缩的gz文件，这样可以减少每次请求进行gzip压缩的CPU资源消耗。该模块启用后，nginx首先检查是否存在请求静态文件的gz结尾的文件，如果有则直接返回该gz文件内容。为了要兼容不支持gzip的浏览器，启用gzip_static模块就必须同时保留原始静态文件和gz文件。这样的话，在有大量静态文件的情况下，将会大大增加磁盘空间。我们可以利用nginx的反向代理功能实现只保留gz文件。
gzip_static on|off;

}
```





#### 2）server块

> server全局块

```ini
#端口号监听
listen 08;

# 主机名称
server_name localhost;

#单连接请求上限次数。
keepalive_requests 120; 

location {

}

#请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
location  ~*^.+$ {       
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           #请求转向mysvr 定义的服务器列表
           proxy_pass  http://mysvr; 
           
           #拒绝的ip
           deny 127.0.0.1;  
           
           #允许的ip
           allow 172.18.5.54;     
        } 
```



> location块

```ini
    location ~ \.(jpeg|jpg|png)$ {
			#缓存时间
			expires 1d;
    }
```



#### 3）gzip压缩



##### gzip on|off

> 默认值: gzip off
> 开启或者关闭gzip模块



##### gzip_static on|off

> nginx对于静态文件的处理模块
> 该模块可以读取预先压缩的gz文件，这样可以减少每次请求进行gzip压缩的CPU资源消耗。该模块启用后，nginx首先检查是否存在请求静态文件的gz结尾的文件，如果有则直接返回该gz文件内容。为了要兼容不支持gzip的浏览器，启用gzip_static模块就必须同时保留原始静态文件和gz文件。这样的话，在有大量静态文件的情况下，将会大大增加磁盘空间。我们可以利用nginx的反向代理功能实现只保留gz文件。



##### gzip_comp_level 4

> 默认值：1(建议选择为4)
> gzip压缩比/压缩级别，压缩级别 1-9，级别越高压缩率越大，当然压缩时间也就越长（传输快但比较消耗cpu）。



##### gzip_buffers 4 16k

> 默认值: gzip_buffers 4 4k/8k
> 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。 例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。 4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。
> 如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。

##### gzip_types mime-type [mime-type ...]

> 默认值: gzip_types text/html (默认不对js/css文件进行压缩)
> 压缩类型，匹配MIME类型进行压缩
> 不能用通配符 text/*
> (无论是否指定)text/html默认已经压缩
> 设置哪压缩种文本文件可参考 conf/mime.types



##### gzip_min_length 1k

> 默认值: 0 ，不管页面多大都压缩
> 设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。
> 建议设置成大于1k的字节数，小于1k可能会越压越大。 即: gzip_min_length 102



##### gzip_http_version 1.0|1.1

> 默认值: gzip_http_version 1.1(就是说对HTTP/1.1协议的请求才会进行gzip压缩)
> 识别http的协议版本。由于早期的一些浏览器或者http客户端，可能不支持gzip自解压，用户就会看到乱码，所以做一些判断还是有必要的。
> 注：99.99%的浏览器基本上都支持gzip解压了，所以可以不用设这个值,保持系统默认即可。
> 假设我们使用的是默认值1.1，如果我们使用了proxy_pass进行反向代理，那么nginx和后端的upstream server之间是用HTTP/1.0协议通信的，如果我们使用nginx通过反向代理做Cache Server，而且前端的nginx没有开启gzip，同时，我们后端的nginx上没有设置gzip_http_version为1.0，那么Cache的url将不会进行gzip压缩



##### gzip_proxied[off|expired|nocache|nostore|private|no_last_modified|no_etag|auth|any] ...

> 默认值：off
> Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含"Via"的 header头。
> off - 关闭所有的代理结果数据的压缩
> expired - 启用压缩，如果header头中包含 "Expires" 头信息
> no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
> no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
> private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
> no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
> no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
> auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
> any - 无条件启用压缩



##### gzip_vary on

> 和http头有关系，加个vary头，给代理服务器用的，有的浏览器支持压缩，有的不支持，所以避免浪费不支持的也压缩，所以根据客户端的HTTP头来判断，是否需要压缩





#### 4) 几个常见配置项

> 1、几个常见配置项：
>
> - 1.$remote_addr 与 $http_x_forwarded_for 用以记录客户端的ip地址； 
> - 2.$remote_user ：用来记录客户端用户名称； 
> - 3.$time_local ： 用来记录访问时间与时区；
> - 4.$request ： 用来记录请求的url与http协议；
> - 5.$status ： 用来记录请求状态；成功是200；
> - 6.$body_bytes_s ent ：记录发送给客户端文件主体内容大小；
> - 7.$http_referer ：用来记录从那个页面链接访问过来的；
> - 8.$http_user_agent ：记录客户端浏览器的相关信息；
>
> 2、惊群现象：一个网路连接到来，多个睡眠的进程被同时叫醒，但只有一个进程能获得链接，这样会影响系统性能。
>
> 3、每个指令必须有分号结束。

