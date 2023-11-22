# shadowtunnel

## 介绍

shadowtunnel 是一款可以在你本地机器和远程服务之间建立安全的加密隧道，保护你的 tcp 流量，

能高效压缩传输，流量无特征.

local machine <----> shadowtunnel <----> service on remote.

## 使用

```text
Usage of ./shadowtunnel:
  -E    outbound connection is encrypted
  -U    outbound connection is udp
  -c    compress traffic
  -cache string
        dns query cache file path (default "cache.dat")
  -daemon
        daemon mode
  -debug
        show debug info
  -dns string
        local dns server listen on address
  -dns-hosts string
        path of dns hosts file
  -dns-proxy
        is dns endpoint or not
  -dns-server string
        remote dns server to resolve domain (default "8.8.8.8:53")
  -e    inbound connection is encrypted
  -f weight
        forward address,such as : 127.0.0.1:8080 or with @weight: 127.0.0.1:8080@1
  -forever
        forever mode
  -l string
        local listen address, such as : 0.0.0.0:33000 (default ":50000")
  -lb-activeafter int
        host going actived after this success count (default 1)
  -lb-hashtarget hash
        use target address to choose parent for LB, only worked for LB's hash
        method and using `-redir` (default true)
  -lb-inactiveafter int
        host going inactived after this fail count (default 2)
  -lb-method string
        load balance method when use multiple parent,can be
        <roundrobin|leastconn|leasttime|hash|weight> (default "leasttime")
  -lb-onlyha high availability mode
        use only high availability mode to choose parent for LB
  -lb-retrytime int
        sleep time milliseconds after checking (default 2000)
  -lb-timeout int
        tcp milliseconds timeout of connecting to parent (default 3000)
  -log string
        logging output to file
  -m string
        method of encrypt/decrypt, these below are supported :
        aes-128-cfb,aes-192-cfb,des-cfb,cast5-cfb,rc4-md5,chacha20,aes-256-cfb,
        aes-128-ctr,aes-192-ctr,aes-256-ctr,bf-cfb,rc4-md5-6,chacha20-ietf
        (default "aes-192-cfb")
  -nolog
        turn off logging
  -p string
        password of encrypt/decrypt (default "shadowtunnel")
  -profiling
        profiling mode, in this mode, you should stopping process
        by : Ctrl+C or 'kill -s SIGHUP $PID_OF_shadowtunnel'
  -redir
        read target from socket's redirect opts of iptables
  -t int
        connection timeout seconds (default 3)
  -ttl int
        cache seconds of dns query , if zero , default ttl used. (default 300)
  -u    inbound connection is udp
  -v    show version
```

## 示例

1.http 代理

假设有一个 vps，它的 IP 是 2.2.2.2

首先在 2.2.2.2 启动一个 http 代理
然后下载 http 代理程序，使用 root 权限在 vps 上执行下面的命令：

wget https://github.com/snail007/goproxy/releases/download/v4.9/proxy-linux-amd64.tar.gz && tar zxfv proxy-linux-amd64.tar.gz && rm proxy-linux-amd64.tar.gz && mv proxy /usr/bin/ && proxy http -p 127.0.0.1:38080 &

2.在 vps 启动一个隧道

下载 shadowtunnel 程序，使用 root 权限在 vps 上执行下面的命令：

wget https://github.com/bhzhu203/shadowtunnel/releases/download/v1.1/shadowtunnel-linux-amd64.tar.gz && tar zxfv shadowtunnel-linux-amd64.tar.gz && rm shadowtunnel-linux-amd64.tar.gz && mv shadowtunnel /usr/bin/

在 vps 上监听 :50000 并转发到 127.0.0.1:38080 ：

`shadowtunnel -e -f 127.0.0.1:38080 -l :50000`

3.在本地机器上启动一个隧道

在本地机器上监听 :50000 并转发到 2.2.2.2:50000 :

`shadowtunnel -E -f 2.2.2.2:50000 -l :50000`

4.在 chrome 中设置 http 代理配置

设置本地 chrome 的http代理配置如下：

ip: 127.0.0.1
port: 50000

5.完成

## TCP over UDP

1.http 代理

假设有一个 vps，它的 IP 是 2.2.2.2

首先在 2.2.2.2 启动一个 http 代理
然后下载 http 代理程序，使用 root 权限在 vps 上执行下面的命令：

`wget https://github.com/snail007/goproxy/releases/download/v4.9/proxy-linux-amd64.tar.gz && tar zxfv proxy-linux-amd64.tar.gz && rm proxy-linux-amd64.tar.gz && mv proxy /usr/bin/ && proxy http -p 127.0.0.1:38080 &`

2.在 vps 启动一个隧道

下载 shadowtunnel 程序，使用 root 权限在 vps 上执行下面的命令：

`wget https://github.com/bhzhu203/shadowtunnel/releases/download/v1.1/shadowtunnel-linux-amd64.tar.gz && tar zxfv shadowtunnel-linux-amd64.tar.gz && rm shadowtunnel-linux-amd64.tar.gz && mv shadowtunnel /usr/bin/`

在 vps 上监听UDP :50000 并转发到TCP 127.0.0.1:38080 ：

`shadowtunnel -u -e -f 127.0.0.1:38080 -l :50000 -p your-password`

3.在本地机器上启动一个隧道

在本地机器上监听TCP :50000 并转发到UDP 2.2.2.2:50000 :

`shadowtunnel -U -E -f 2.2.2.2:50000 -l :50000 -p your-password`

4.在 chrome 中设置 http 代理配置

设置本地 chrome 的http代理配置如下：

ip: 127.0.0.1
port: 50000

5.完成

## Deamon & Forever & Log

-daemon:

使用参数-daemon可以让shadowtunnel脱离当前命令行,后台运行.

-forever:

使用参数-forever可以让shadowtunnel以创建并监控子进程的方式运行,

如果发生异常退出,会重启子进程,保证服务永远在线.

-log

使用参数-log可以设置日志输出到文件,而不是在命令行输出.

-nolog

使用参数-nolog可以从彻底关闭日志输出,节省CPU占用.

一般是-daemon -forever -log /tmp/st.log 三个参数联合使用,这样出问题时,也可以通过看日志发现问题原因.

实例:

`shadowtunnel -u -e -f 127.0.0.1:38080 -l :50000 -p your-password -daemon -forever -log /tmp/st.log`

## DNS服务

shadowtunnel可以在提供本地DNS查询服务,同时具有缓存功能,可以提高解析速度.

在本地启动端口转发的同时启动一个DNS服务,需要有上级配合.

-dns 参数可以设置本地DNS服务监听的IP和端口,比如:0.0.0.0:5353

-dns-server 参数可以设置最终用来解析域名的DNS服务器,要求是服务器必须支持TCP方式的DNS查询,默认是:8.8.8.8:53.

本地实例:

`shadowtunnel -E -f 2.2.2.2:50000 -l :50000  -p your-password -dns :5353 -dns-server 8.8.8.8:53`

上级实例:

如果上级是链式,那么链条中需要执行DNS代理的上级需要加上-dns-proxy参数.

`shadowtunnel -e -f 127.0.0.1:38080 -l :50000 -p your-password -dns-proxy`

## DNS缓存

-ttl 参数可以设置DNS查询结果缓存时间,单位秒,如果是0,使用查询结果的ttl.

-cache 参数设置DNS缓存文件位置,防止程序重启缓存消失,降低性能.

## 负载均衡

支持上级负载均衡,多个上级重复-f参数即可.

`shadowtunnel -E -f 2.2.2.2:50000 -f 3.3.3.3:50000 -l :50000`

### 设置重试间隔和超时时间

`shadowtunnel -E -f 2.2.2.2:50000 -f 3.3.3.3:50000 -l :50000 -lb-method leastconn -lb-retrytime 300 -lb-timeout 300`

### 设置权重  

`shadowtunnel -E -f 2.2.2.2:50000@2 -f 3.3.3.3:50000@1 -l :50000 -lb-method weight -lb-retrytime 300 -lb-timeout 300`

### 使用目标地址选择上级

`shadowtunnel -E -f 2.2.2.2:50000@2 -f 3.3.3.3:50000@1 -l :50000 -lb-method hash -lb-hashtarget -lb-retrytime 300 -lb-timeout 300`

## DNS HOSTS文件

-dns-hosts参数可以设置DNS解析的时候使用的hosts文件,内容和系统的hosts文件内容格式相同.

`shadowtunnel -f 2.2.2.2:50000 -dns :5353 -dns-hosts /etc/hosts`