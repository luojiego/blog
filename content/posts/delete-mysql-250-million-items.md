---
title: 如何删除 MySQL 表中 2.5 亿条数据？
date: 2021-05-25 21:57:32
tags:
- MySQL
- Go
---

# 为什么会有 2.5 亿条数据

运营需求，需要提供查询用户登录的记录。

我一思索，这简单呀，就设计了如下的数据表。

```sql
desc login_record;
+-------------+------------------+------+-----+-------------------+-------+
| Field       | Type             | Null | Key | Default           | Extra |
+-------------+------------------+------+-----+-------------------+-------+
| user_id     | int(10) unsigned | NO   | MUL | NULL              |       |
| ip          | varchar(40)      | YES  |     |                   |       |
| device_mark | varchar(500)     | NO   |     |                   |       |
| time        | timestamp        | NO   | MUL | CURRENT_TIMESTAMP |       |
+-------------+------------------+------+-----+-------------------+-------+
```
<!--more-->
由于 `user_id` 和 `time` 上都有索引，所以查询的速度很快。

但是某一次我闲着没事，执行了个 `SELECT COUNT(*) FROM login_record` 发现很久都没有返回，我觉得事情可能比我想的要严重，这个表根据我的推测，可能数据过亿条了。

![表中的总条目](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/counts-more-than-250-millions.png)

# 例行维护

在知道这个表比较大之后，我的应对方案是将该表从 RDS 迁移到本地的数据库中。

![RDS 的配置](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/2021-05-25-rds-config.png)



本地的数据库使用的 EC2 机器比较渣，2 核 8 GB。

反正停服之后搞，也不会影响用户。我们预计的停服时间是六个小时，停服一开始，在确定没有数据库写入之后，我的小伙伴就开始用 mysqldump 工具将 RDS 中的数据往本地导入了，因为在同个内网，我们猜测应该会比较快，顺便我们也看了一下 RDS 的监控。

![14:17 分执行了 mysqldump 之后，RDS IOPS 监控](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/2021-05-25-rds-monitor.png)



显然，**RDS 的 IOPS 瞬间从 0 上升到了 2000**，但是就下降了。但是 mysqldump 一直在执行，直到 15:30，我们实在没有办法等了，就停止掉了 mysqldump。此时查看本地数据库，发现**才导入了 九千万记录**。

此时我们意识到，可能我们直接扔掉这个表可能会比较合适。

那就先用本地的刚才导入的数据表试一下 drop，在我们执行了 drop 一小时之后，正常结束。

这下我打消了在 RDS 上执行 drop 的想法，因为已经离停服开始过去了两个半小时，而且我们还有好多功能要进行测试。

但是好消息是，我可以先删除掉 RDS 记录上的一部分数据，因为这个不会影响测试人员的正常工作。

![删除 1.1 亿条数据，花费了这么长时间](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/Untitled%203.png)

但是即使我删除掉 1.1 亿条数据，也花费了两个小时。

接着，我们放弃了立刻要在本地停服中删除掉这个数据表的想法，可以通过脚本慢慢删除这个数据表中的数据，毕竟这个表不会再写入。

# 如何删除

## 用 LIMIT ?

看看以下有趣的现象

```sql
explain select user_id from login_record limit 10;
+------+-------------+--------------+-------+---------------+---------+---------+------+----------+-------------+
| id   | select_type | table        | type  | possible_keys | key     | key_len | ref  | rows     | Extra       |
+------+-------------+--------------+-------+---------------+---------+---------+------+----------+-------------+
|    1 | SIMPLE      | login_record | index | NULL          | user_id | 4       | NULL | 12261955 | Using index |
+------+-------------+--------------+-------+---------------+---------+---------+------+----------+-------------+
```

```sql
explain select user_id,time from login_record limit 10;
+------+-------------+--------------+------+---------------+------+---------+------+----------+-------+
| id   | select_type | table        | type | possible_keys | key  | key_len | ref  | rows     | Extra |
+------+-------------+--------------+------+---------------+------+---------+------+----------+-------+
|    1 | SIMPLE      | login_record | ALL  | NULL          | NULL | NULL    | NULL | 12262319 |       |
+------+-------------+--------------+------+---------------+------+---------+------+----------+-------+
```

虽然 user_id 和 time 上都有索引，**查询 user_id 或者 time 都会使用索引**，但是如果**一起查，不好意思，不能使用索引**。

这个仔细一想，很合理，因为该表 user_id 和 time 都是普通索引，如果两个要一起查，显然不能使用索引，所以要么根据 user_id 删除，要么根据时间删除。

## 用时间删除可能比较合理

线上总注册用户超过三百万，虽然可以使用 user_id 不停的删除也是个办法。

但是我还是觉得使用时间来删除比较合适，于是我写个程序，按小时来删除数据。每次删除完之后，还执行睡眠一秒。

```go
		// 每次删除一个小时的日志
    for i := startTime; i < endTime; {
        start := time.Unix(int64(i), 0).Format(layout)                                                                                              
        i += 1800
        end := time.Unix(int64(i), 0).Format(layout)
				count, err := deleteRecord(start, end)
        if err != nil {
            panic(err)
        }
        if count == 0 {
            fmt.Println("end time: ", i)
            break
        }
        fmt.Printf("[%s-%s) delete count: %d\n", start, end, count)

        time.Sleep(1 * time.Second)
    }
```



![使用脚本删除时，IOPS 的变化](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/Untitled%204.png)



![使用脚本删除时，CPU 和内存使用率的变化](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/Untitled%205.png)

# 总结

在执行了大概两个多小时，删除了 1.5 亿条中的 90% 的日志。剩余的日志，我使用脚本导入到了本地数据库，让运营有数据可查。

删除完 RDS 数据表中的数据之后，占用的存储空间不会下降，需要 drop 掉表之后，空间才能释放，我的需求也是不再使用这张表，所以就使用了 drop。



![drop 掉表之后，磁盘空间的释放](https://go-daily.oss-cn-chengdu.aliyuncs.com/img/Untitled%206.png)
