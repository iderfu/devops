---
category: MySQL
---
# 13 MySQL常见问题

## 主库异常，从库手动切换为主库方案

1.登录从服务器，确认从服务器已经完成所有同步操作：

```
mysql> stop slave io_thread  
mysql> show processlist 
直到看到状态都为：xxx has read all relay log 表示更新都执行完毕
```

2.停止从服务器slave服务：

```
mysql> stop slave
```

3.将从服务器切换为主服务器：

```
mysql> reset master 
```

完成切换

4.授权内网其他机器有写入等权限(如果没有权限的话)

```
mysql> SELECT Host,User FROM mysql.user;
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.1.%' IDENTIFIED BY '123456'  WITH GRANT OPTION;
mysql> FLUSH PRIVILEGES;
```

5.修改其他机器hosts或应用内连接

```
# vi /etc/hosts
192.168.1.106 db-001
```

Mysql常见的几个错误问题及解决方法：

<!--more-->

## mysql DNS反解：skip-name-resolve

错误日志有类似警告：

1. 120119 16:26:04 [Warning] IP address '192.168.1.10' could not be resolved: Name or service not known

2. 120119 16:26:04 [Warning] IP address '192.168.1.14' could not be resolved: Name or service not known

3. 120119 16:26:04 [Warning] IP address '192.168.1.17' could not be resolved: Name or service not known

通过show processlist发现大量类似如下的连接：


1. |592|unauthenticated user|192.168.1.10:35320|NULL|Connect| |login|NULL|

2. |593|unauthenticated user|192.168.1.14:35321|NULL|Connect| |login|NULL|

3. |594|unauthenticated user|192.168.1.17:35322|NULL|Connect| |login|NULL|

 skip-name-resolve 参数的作用：不再进行反解析（ip不反解成域名），这样可以加快数据库的反应时间。 修改配置文件添加并需要重启：

```
[mysqld] 
skip-name-resolve
```

## 一键安装mysql脚本
```shell
#!/bin/bash


# Notes: install mysql5.6 on centos
#
mysql_install_dir=/opt/mysql                 #程序目录
mysql_data_dir=/opt/mysql/data                           #数据目录
mysql_6_version=5.6.36                                    #更改文件名
dbrootpwd=1qazxsw2                                              #mysql密码

Mem=`free -m | awk '/Mem:/{print $2}'`
Swap=`free -m | awk '/Swap:/{print $2}'`

Install_MySQL()
{
yum -y install make gcc-c++ cmake bison-devel  ncurses-devel autoconf
wget http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-${mysql_6_version}.tar.gz

id -u mysql >/dev/null 2>&1
[ $? -ne 0 ] && useradd -M -s /sbin/nologin mysql

mkdir -p $mysql_data_dir;chown mysql.mysql -R $mysql_data_dir
tar zxf mysql-${mysql_6_version}.tar.gz
cd mysql-$mysql_6_version
make clean
[ ! -d "$mysql_install_dir" ] && mkdir -p $mysql_install_dir
cmake . -DCMAKE_INSTALL_PREFIX=$mysql_install_dir \
-DMYSQL_DATADIR=$mysql_data_dir \
-DSYSCONFDIR=/etc \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DENABLE_DTRACE=0 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DWITH_EMBEDDED_SERVER=1 \

make -j `grep processor /proc/cpuinfo | wc -l`
make install

if [ -d "$mysql_install_dir/support-files" ];then
    echo "${CSUCCESS}MySQL install successfully! ${CEND}"
    cd ..
    rm -rf mysql-$mysql_6_version
else
    rm -rf $mysql_install_dir
    echo "${CFAILURE}MySQL install failed, Please contact the author! ${CEND}"
    kill -9 $$
fi

/bin/cp $mysql_install_dir/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig mysqld on
cd ..

# my.cf
[ -d "/etc/mysql" ] && /bin/mv /etc/mysql{,_bk}
cat > /etc/my.cnf << EOF
[client]
port = 3306
socket = /tmp/mysql.sock
default-character-set = utf8

[mysqld]
port = 3306
socket = /tmp/mysql.sock

basedir = $mysql_install_dir
datadir = $mysql_data_dir
pid-file = $mysql_data_dir/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1

init-connect = 'SET NAMES utf8mb4'
character-set-server = utf8mb4

skip-name-resolve
skip-external-locking
#skip-networking
back_log = 300

max_connections = 1000
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 128
max_allowed_packet = 4M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 16M

read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 8M
key_buffer_size = 4M
thread_cache_size = 8
query_cache_type = 1
query_cache_size = 8M
query_cache_limit = 2M
ft_min_word_len = 4
log_bin = mysql-bin
binlog_format = mixed
expire_logs_days = 10
log_error = $mysql_data_dir/mysql-error.log
slow_query_log = 1
long_query_time = 1
#slow_query_log_file = $mysql_data_dir/mysql-slow.log
performance_schema = 0
explicit_defaults_for_timestamp

#lower_case_table_names = 1
default_storage_engine = InnoDB
#default-storage-engine = MyISAM
innodb_file_per_table = 1
innodb_open_files = 500
innodb_buffer_pool_size = 64M
innodb_write_io_threads = 4
innodb_read_io_threads = 4
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 32M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120

bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 8M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1

interactive_timeout = 28800
wait_timeout = 28800

[mysqldump]
quick
max_allowed_packet = 16M

[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M

EOF

if [ $Mem -gt 1500 -a $Mem -le 2500 ];then
    sed -i 's@^thread_cache_size.*@thread_cache_size = 16@' /etc/my.cnf
    sed -i 's@^query_cache_size.*@query_cache_size = 16M@' /etc/my.cnf
    sed -i 's@^myisam_sort_buffer_size.*@myisam_sort_buffer_size = 16M@' /etc/my.cnf
    sed -i 's@^key_buffer_size.*@key_buffer_size = 16M@' /etc/my.cnf
    sed -i 's@^innodb_buffer_pool_size.*@innodb_buffer_pool_size = 128M@' /etc/my.cnf
    sed -i 's@^tmp_table_size.*@tmp_table_size = 32M@' /etc/my.cnf
    sed -i 's@^table_open_cache.*@table_open_cache = 256@' /etc/my.cnf
elif [ $Mem -gt 2500 -a $Mem -le 3500 ];then
    sed -i 's@^thread_cache_size.*@thread_cache_size = 32@' /etc/my.cnf
    sed -i 's@^query_cache_size.*@query_cache_size = 32M@' /etc/my.cnf
    sed -i 's@^myisam_sort_buffer_size.*@myisam_sort_buffer_size = 32M@' /etc/my.cnf
    sed -i 's@^key_buffer_size.*@key_buffer_size = 64M@' /etc/my.cnf
    sed -i 's@^innodb_buffer_pool_size.*@innodb_buffer_pool_size = 512M@' /etc/my.cnf
    sed -i 's@^tmp_table_size.*@tmp_table_size = 64M@' /etc/my.cnf
    sed -i 's@^table_open_cache.*@table_open_cache = 512@' /etc/my.cnf
elif [ $Mem -gt 3500 ];then
    sed -i 's@^thread_cache_size.*@thread_cache_size = 64@' /etc/my.cnf
    sed -i 's@^query_cache_size.*@query_cache_size = 64M@' /etc/my.cnf
    sed -i 's@^myisam_sort_buffer_size.*@myisam_sort_buffer_size = 64M@' /etc/my.cnf
    sed -i 's@^key_buffer_size.*@key_buffer_size = 256M@' /etc/my.cnf
    sed -i 's@^innodb_buffer_pool_size.*@innodb_buffer_pool_size = 1024M@' /etc/my.cnf
    sed -i 's@^tmp_table_size.*@tmp_table_size = 128M@' /etc/my.cnf
    sed -i 's@^table_open_cache.*@table_open_cache = 1024@' /etc/my.cnf
fi

$mysql_install_dir/scripts/mysql_install_db --user=mysql --basedir=$mysql_install_dir --datadir=$mysql_data_dir

chown mysql.mysql -R $mysql_data_dir
service mysqld start
[ -z "`grep ^'export PATH=' /etc/profile`" ] && echo "export PATH=$mysql_install_dir/bin:\$PATH" >> /etc/profile
[ -n "`grep ^'export PATH=' /etc/profile`" -a -z "`grep $mysql_install_dir /etc/profile`" ] && sed -i "s@^export PATH=\(.*\)@export PATH=$mysql_install_dir/bin:\1@" /etc/profile

. /etc/profile

$mysql_install_dir/bin/mysql -e "grant all privileges on *.* to root@'127.0.0.1' identified by \"$dbrootpwd\" with grant option;"
$mysql_install_dir/bin/mysql -e "grant all privileges on *.* to root@'localhost' identified by \"$dbrootpwd\" with grant option;"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "delete from mysql.user where Password='';"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "delete from mysql.db where User='';"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "delete from mysql.proxies_priv where Host!='localhost';"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "drop database test;"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "reset master;"
rm -rf /etc/ld.so.conf.d/{mysql,mariadb,percona}*.conf
echo "$mysql_install_dir/lib" > mysql.conf
/sbin/ldconfig
service mysqld stop
}

Install_MySQL
```

