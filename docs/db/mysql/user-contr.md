---
category: MySQL
---
# 6 用户权限和访问控制

## 1 用户

### 1.1 创建用户并设置密码

```mysql
create user aa@localhost identified by '123';//本地用户
create user aa@'%' identified by '123'; //其他远程用户
```

### 1.2 重命名用户

```mysql
rename user 'test3'@'%' to 'test1'@'%';
```

### 1.3 删除用户

```mysql
drop user 'testUser'@'%';
```

## 2 权限

### 2.1 授予权限

```mysql
grant 权限 on 库.表 to 用户@主机 identified by '密码'；grant 权限 on 库.表 to 用户@主机 identified by '密码'；
grant all on *.* to 'testUser'@'%' identified by '123';
```

#### 2.1.1 查询、插入、更新、删除的权限

```mysql
grant select on testdb.* to 'testUser'@'%';  
grant insert on testdb.* to 'testUser'@'%';  #其中*第通配符，表示所有
grant update on testdb.* to 'testUser'@'%';
grant delete on testdb.* to 'testUser'@'%';

#总结成一条命令
grant select,insert,update,delete on testdb.* to 'testUser'@'%';
```

#### 2.1.2 创建和删除表、索引、视图、存储过程的权限

```mysql
grant create on testdb.* to 'testUser'@'%';  #其中*第通配符，表示所有
grant alter on testdb.* to 'testUser'@'%';
grant drop on testdb.* to 'testUser'@'%';

#总结成一条命令
grant create,alter,drop on testdb.* to 'testUser'@'%';

#外键权限
grant reference on testdb.* to 'testUser'@'%';

#索引权限
grant index on testdb.* to 'testUser'@'%';

#视图权限
grant create view on testdb.* to 'testUser'@'%';
grant show view on testdb.* to 'testUser'@'%';

#存储过程权限
grant create routine on testdb.* to 'testUser'@'%';
grant alter routine on testdb.* to 'testUser'@'%';
grant execute on testdb.* to 'testUser'@'%';
```

#### 2.1.3 指定用户管理数据库的权限

```mysql
#仅管理testdb数据库
grant all privileges on testdb.* to 'testUser'@'%';

#管理所有数据库
grant all privileges on *.* to 'testUser'@'%';
#其中privileges关键字可省略
```

### 2.2 权限的作用层次

#### 2.2.1 作用在整个MySQL服务器上

```mysql
grant all privileges on *.* to 'testUser'@'%';
```

#### 2.2.2 作用在单个数据库上

```mysql
grant all privileges on testdb.* to 'testUser'@'%';
```

#### 2.2.3 作用在单个数据表上

```mysql
grant all privileges on testdb.testTable to 'testUser'@'%';
```

#### 2.2.4 作用在单个数据表的若干个列上

```mysql
grant select(id, name, home, phone) on testdb.testTable to 'testUser'@'%'; #select可以改其他，字段根据实际修改
```

#### 2.2.5 作用在存储过程、函数上

```mysql
grant execute on procedure testdb.tsetfunc to 'testUser'@'%';
grant execute on function testdb.tsetfunc to 'testUser'@'%';
```

### 2.3 权限刷新

```mysql
flush privileges;
```

### 2.4 查看权限

```mysql
#查看当前用户的权限
show grants;

#查看mysql中其他用户的权限
show grants for 'testUser'@'%';
```

### 2.5 移除权限

```mysql
revoke 权限 on 库.表 from 用户@主机;
revoke all on *.* from 'testUser'@'%';
```

### 2.6 mysql授权表

mysql授权表共有5个表：user、db、host、tables_priv和columns_priv。

#### 2.6.1 user表

user表列出可以连接服务器的用户及其口令，并且它指定他们有哪种全局（超级用户）权限。在user表启用的任何权限均是全局权限，并适用于所有数据库。

#### 2.6.2 db表

db表列出数据库，用户有权限访问它们。在这里指定的权限适用于一个数据库中的所有表。

#### 2.6.2 host表

host表与db表结合使用在一个较好层次上控制特定主机对数据库的访问权限，这可能比单独使用db好些。这个表不受GRANT和REVOKE语句的影响，所以，你可能发觉你根本不是用它。

#### 2.6.3 tables_priv表

tables_priv表指定表级权限，在这里指定的一个权限适用于一个表的所有列。

#### 2.6.4 columns_priv表

columns_priv表指定列级权限。这里指定的权限适用于一个表的特定列。

### 2.7 注意事项

grant, revoke 用户权限后，该用户只有重新连接 MySQL 数据库，权限才能生效。

如果想让授权的用户，也可以将这些权限 grant 给其他用户，需要选项 `grant option`

```mysql
grant all on testdb.* to 'testUser'@'%' with grant option;
```

## 3 密码

### 3.1 修改密码

####  3.1.1 更新mysql.user表

```mysql
# mysql5.7之前
update user set password=password('123456') where user='root';
# mysql5.7之后
update user set authentication_string=password('123456') where user='root';
flush privileges;
```

#### 3.1.2 用set password命令

**语法：**set password for ‘用户名'@'登录地址'=password(‘密码')

```mysql
set password for 'root'@'localhost'=password('123456');
```

#### 3.1.3 mysqladmin

**语法：**mysqladmin -u用户名 -p旧的密码 password 新密码

```mysql
mysqladmin -uroot -p123456 password 1234abcd
```

### 3.2 忘记密码

#### 3.2.1 跳过授权

`vim /etc/my.cnf`

```
[mysqld]
skip-grant-tables
```

#### 3.2.2 重启服务

```
service mysqld restart
```

#### 3.2.3 修改密码

此时在终端用mysql命令登录时不需要用户密码，然后按照修改密码的第一种方式将密码修改即可。

#### 3.2.4 还原登录权限跳过检查配置

将my.cnf中mysqld节点的skip-grant-tables配置删除，然后重新启动服务即可。

> 参考链接：
>
> https://blog.csdn.net/a791693310/article/details/81083864
>
> https://www.jb51.net/article/87979.htm


