---
category: MySQL
---
# 14 MySQL bin-log日志清理

## 自动清理

### 永久生效

需要重启mysql才能生效

修改`my.cnf`文件

添加下面一行

```
expire_logs_days = 7
```

### 临时生效

进入mysql，执行，下面的语句

```
show variables like '%expire_logs_days%';
set global expire_logs_days = 7;
```

## 手动清理

进入mysql，查看binlog日志

```
show binary logs;
```

**删除某个日志文件之前的所有日志文件**

```
purge binary logs to 'mysql-bin.000035';
```

**清理2019-09-09 13:00:00前binlog日志**

```
PURGE MASTER LOGS BEFORE '2019-09-09 13:00:00';
```

**清除3天前的bin日志**

```
PURGE MASTER LOGS BEFORE DATE_SUB(NOW( ), INTERVAL 3 DAY); 
```

> 注意，不要轻易手动去删除binlog，会导致binlog.index和真实存在的binlog不匹配，而导致expire_logs_day失效