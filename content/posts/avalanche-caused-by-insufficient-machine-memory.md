---
title: 内存不足 Redis 获取数据出错引发的雪崩分析
date: 2022-03-06 22:26:27
tags:
- Redis
- Code
---
## 异常回顾

28 日凌晨机器警报拉响，某 Redis 服务出现“MISCONF Redis is configured to save RDB snapshots……”，接着活动服开始连续崩溃，大概十分钟之后，活动服正常拉起。

经过分析，发现机器由于内存不足，Redis 服务无法正常 dump，导致写入失败引发的连环错误。但是看服务的日志时，我们发现一些特别诡异的问题，从 Redis 中获取出来的数据数据格式和请求的根本的不匹配。熟悉 C++ 的小伙伴深知，这大概率是内存出错了。但是仔细看了看，毫无头绪，主要由于服务**使用的是单线程的模型，请求数据时，返回的数据格式就是错误的**。

HSET 的时候，还会调用 EXPIRE 设置过期时间，调用了两次 `redisAppendCommand` ，**但却只调用了一次** **`redisGetReply`** 命令，**也即只获取了 hset 的返回，但是并未获取 expire 的返回**，**这会导致后续的命令再调用** **`redisGetReply`** **时获取到是 expire 的返回，后续的命令就会连续错位。**
<!--more-->
## 环境信息

Redis v 3.12

hiredis: v0.11.0

## 错误分析

UpdateHash 的封装如下：

```cpp
int RedisManager::UpdateHash(const std::string& key_name, const FieldInfo& field, int expire_time /* = kMaxExpireTime*/) {
    if (RedisSuccess != checkConnect()) {
        log_error("connect redis faild");
        return RedisServerInvaild;
    }

    // 注意此处的 %b
    redisAppendCommand(_connect, "hset %s %s %b", key_name.c_str(), field.name.c_str(), field.value.c_str(), field.value.size());
    redisAppendCommand(_connect, "expire %s %d", key_name.c_str(), expire_time);

    redisReply* reply = NULL;
    redisGetReply(_connect, (void **)&reply);

    if (!reply) {
        log_error("hset key:%s field name:%s, error:%d:%s", key_name.c_str(), field.name.c_str(), _connect->err, _connect->errstr);
        return RedisSystemError;
    }

    if (reply->type != REDIS_REPLY_INTEGER) {
        log_error("hset key:%s field:%s failed, type:%d err:%s", key_name.c_str(), field.name.c_str(), reply->type, reply->str);
        freeReplyObject(reply);
        return RedisSystemError;
    }

    freeReplyObject(reply);

    redisGetReply(_connect, (void **)&reply);
    if (reply) {
        // 这里的逻辑写的依然有问题
        freeReplyObject(reply);
    } else {
        log_debug("expire set failed ret type:%d value:%lld", reply->type, reply->integer);
    }                                                                                                                                                              
    return RedisSuccess;
}
```

错误原因：注意第 7 行和第 8 行，**此处添加了两条指令 hset 和 expire**，但是由于机器内存不足，dump 不成功，redisGetReply 会请求 hset 的返回出错，错误信息为: “MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error.”，**UpdateHash 从 14 行错误返回。**

GetHash 的封装如下：

```cpp
int RedisManager::GetHash(const std::string& key_name, FieldInfo& field) {
    if (RedisSuccess != checkConnect()) {
        log_error("connect redis faild");
        return RedisServerInvaild;
    }

    snprintf(_command, sizeof(_command), "hget %s %s", key_name.c_str(), field.name.c_str());
    redisReply* reply = reinterpret_cast<redisReply*>(redisCommand(_connect, _command)); 
    if (!reply) {
        log_error("hget key:%s field name:%s, error:%d:%s", key_name.c_str(), 
                field.name.c_str(), _connect->err, _connect->errstr);
        return RedisSystemError;
    }

    if (REDIS_REPLY_NIL == reply->type) {
        log_error("hget key:%s field:%s not get data", key_name.c_str(), field.name.c_str());
        freeReplyObject(reply);
        return RedisKeyNotExist;
    } else if (reply->type != REDIS_REPLY_STRING) {
        log_error("hget key:%s error:%d:%s", key_name.c_str(), reply->type, reply->str);
        freeReplyObject(reply);
        return RedisSystemErro
    }

    field.value.clear();
    field.value.assign(reply->str, reply->len);

    freeReplyObject(reply);
    return RedisSuccess;
}
```

hiredis 底层的主要逻辑如下：

```c
int redisGetReply(redisContext *c, void **reply) {
    int wdone = 0;
    void *aux = NULL;

    /* Try to read pending replies */
    if (redisGetReplyFromReader(c,&aux) == REDIS_ERR)
        return REDIS_ERR;

    /* For the blocking context, flush output buffer and read reply */
    if (aux == NULL && c->flags & REDIS_BLOCK) {
        /* Write until done */
        do {
            if (redisBufferWrite(c,&wdone) == REDIS_ERR)
                return REDIS_ERR;
        } while (!wdone);

        /* Read until there is a reply */
        do {
            if (redisBufferRead(c) == REDIS_ERR)
                return REDIS_ERR;
            if (redisGetReplyFromReader(c,&aux) == REDIS_ERR)
                return REDIS_ERR;
        } while (aux == NULL);
    }

    /* Set reply object */
    if (reply != NULL) *reply = aux;
    return REDIS_OK;
}
```

GetHash 中第 8 行 `redisReply* reply = reinterpret_cast<redisReply*>(redisCommand(_connect, _command));` 实际会调用 **`redisGetReply`** ，我们注意看第 6 行，由于 UpdateHash 的请求中还有一个 **`expire`** 的指令结果没有获取，此处会设置 aux 的值，第 10 行的逻辑不会进入。显然 HGet 请求的结果实际上是 expire 的返回结果，在 GetHash 函数中会走到第 20 行，从错误逻辑分支退出。

```cpp
class RedisManager {
// ....
private:
	redisContext* _connect;	
};
```

```cpp
typedef struct redisContext {
    int err; /* Error flags, 0 when there is no error */
    char errstr[128]; /* String representation of error when applicable */
    int fd;
    int flags;
    char *obuf; /* Write buffer */
    redisReader *reader; /* Protocol reader */
} redisContext;
```

但是错误远未结束，因为后续的请求依然会使用 _connect 这个对象，_connect 是在和 Redis 建立链接时创建出来的。obuf 是 write buffer，刚才我们的 GetHash 由于直接获取了 expire 的结果，会导致 obuf 的内容并未发送（write）。这个错误将会延续到下一个 Redis 的请求中，下一次的请求会继续往 obuf 中追加内容，再调用 **`redisGetReply`** 时，会一次性将 obuf 的内容发送出去，然后清空 obuf。但是不幸的是，既然此次返回成功，也是上一个 GetHash 的返回，并非现在请求的结果。后果就是后续的请求几乎全部错位，**严重时可能会导致大量的用户数据出错**。

## 如何修复？

UpdateHash 不再使用 redisAppendCommand，而是直接使用 redisCommand，避免请求了两条指令却只获取一次返回。

更进一步，如何将 hset 和 expire 做成原子性？向网友求助，给的解决方案是 lua，但由于我们使用的是 hiredis，不支持，暂时只能先修复现有封装中的错误，看后续是否有合适的时机解决这一问题。

## 学到了什么

### Redis dump 会压缩吗？

```bash
127.0.0.1:6379> config get rdbcompression
1) "rdbcompression"
2) "yes"
```

**rdbcompression** 是是否开启压缩的标识，默认是 yes。我在构建复现环境的时候就被坑了，启动了一个 16mb 的 docker 的 Redis 实例，写入了大概 10mb 的空间，但是 dump 是可以成功的。而且 dump 的文件竟然不足 1mb，最后解决的办法是通过 uuid 随机写入来构建重现环境的。

```bash
#!/bin/bash
for i in {1..3000}
do
    echo "" > /tmp/test.txt
    for j in {1..100}
    do
        cat /proc/sys/kernel/random/uuid >> /tmp/test.txt
    done
    s=$(cat /tmp/test.txt)
    echo $s
    redis-cli -p 6379 -n 15 set test5_$i "$s"
done
```

### 可以设置 stop-writes-on-bgsave-error 为 no 吗？

默认为 yes 是有原因的，如果不深入了解，把这个配置设置为 no 的话，将会导致在内存不足的时候，dump 不会成功。若因为机器内存不足，Redis 实例因为 OOM 原因被系统杀掉，显然会丢失大量的数据。要开 dump，显然是不希望数据丢失，除非特别理解将该参数的意思，请务必不要将其设置为 no。

### 我对 Redis 只能说是了解

从毕业到现在已经 9 年多了，几乎一直都在使用 Redis，但是显然我没有深入研究过 Redis，到现在我才意识，我只是熟悉它的几种数据结构，事务，订阅及一些高级应用，只在很早的时候了解过。现在基本上忘过了，看来是时候好好去研究总结一下了。

## 总结

**源码之下，了无秘密**。再离奇的问题，**总归有一个非常简单的原因**。出了问题，百分之九十九的情况都是我们的代码写的有问题。静下心来，深入到代码中，构建复现环境，通过 GDB 等方式了解源码，不断提升自己阅读源码的功底。利用合适的工具，快速的解决现有问题。
