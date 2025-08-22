## Nginx升级加固SSL/TLS协议信息泄露漏洞(CVE-2016-2183)和HTTP服务器的缺省banner漏洞

> :warning: 注意：要根据实际情况进行修改，这只是大体的思路

```
cd /tmp
wget  https://www.openssl.org/source/openssl-1.1.0k.tar.gz
tar zxvf openssl-1.1.0k.tar.gz -C /usr/local

```

```
# 打开nginx源文件下的/usr/local/src/nginx-1.9.9/auto/lib/openssl/conf文件：
vi /root/nginx-1.14.2/auto/lib/openssl/conf
# 找到以下代码,差不多三四十行
CORE_INCS="$CORE_INCS $OPENSSL/.openssl/include"
CORE_DEPS="$CORE_DEPS $OPENSSL/.openssl/include/openssl/ssl.h"
CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libssl.a"
CORE_LIBS="$CORE_LIBS $OPENSSL/.openssl/lib/libcrypto.a"
CORE_LIBS="$CORE_LIBS $NGX_LIBDL"
# 修改成以下代码
CORE_INCS="$CORE_INCS $OPENSSL/include"
CORE_DEPS="$CORE_DEPS $OPENSSL/include/openssl/ssl.h"
CORE_LIBS="$CORE_LIBS $OPENSSL/lib/libssl.a"
CORE_LIBS="$CORE_LIBS $OPENSSL/lib/libcrypto.a"
CORE_LIBS="$CORE_LIBS $NGX_LIBDL"
```

```
cd /usr/local/openssl-1.1.0k/
mkdir lib
cp libssl.a libcrypto.a lib

./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-openssl=/usr/local/openssl-1.1.0k
make -j 8
cd /usr/local/nginx/sbin/
cp nginx nginx.bak
cd /opt/nginx-1.16.1
cp -f objs/nginx /usr/local/nginx/sbin/

ps -ef|grep nginx
kill -USR2 `master 进程号`
#关闭旧的woker进程，kill -WINCH旧的master进程号
kill -WINCH `master 进程号`
#关闭旧的master进程
kill -QUIT `master 进程号`

```

### Nginx缺省banner修改

```
[root@Test ~]# vim nginx-1.19.1/src/http/ngx_http_header_filter_module.c 

需要修改：

static u_char ngx_http_server_string[] = "Server: nginx" CRLF;
static u_char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;
static u_char ngx_http_server_build_string[] = "Server: " NGINX_VER_BUILD CRLF;

修改成：

static u_char ngx_http_server_string[] = "Server: unknow" CRLF;
static u_char ngx_http_server_full_string[] = "Server: unknow"  CRLF;
static u_char ngx_http_server_build_string[] = "Server: unknow"  CRLF;

```

然后重新编译，热升级nginx