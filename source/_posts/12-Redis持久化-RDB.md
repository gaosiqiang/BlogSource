---
title: 12.Redis持久化——RDB
date: 2020-02-21 11:37:18
tags:
    - Redis
    - 缓存
categories:
    - 中间件
    - Redis
---

## 12.1 RDB机制

RDB其实就是把数据以快照的形式保存在磁盘上。

RDB持久化是指在指定的时间间隔内将内存中的数据集快照写入磁盘。也是`默认的持久化方式`，这种方式是就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为dump.rdb。



## 12.2 触发方式

- 手动触发：执行BGSAVE命令
- 自动触发：配置SAVE选项，在指定时间内发生指定次数的key修改，自动进行后台RDB SAVE



```c
#file: src/rdb.c
void saveCommand(redisClient *c) {

    // BGSAVE 已经在执行中，不能再执行 SAVE
    // 否则将产生竞争条件
    if (server.rdb_child_pid != -1) {
        addReplyError(c,"Background save already in progress");
        return;
    }

    // 执行 
    if (rdbSave(server.rdb_filename) == REDIS_OK) {
        addReply(c,shared.ok);
    } else {
        addReply(c,shared.err);
    }
}
void bgsaveCommand(redisClient *c) {

    // 不能重复执行 BGSAVE
    if (server.rdb_child_pid != -1) {
        addReplyError(c,"Background save already in progress");

    // 不能在 BGREWRITEAOF 正在运行时执行
    } else if (server.aof_child_pid != -1) {
        addReplyError(c,"Can't BGSAVE while AOF log rewriting is in progress");

    // 执行 BGSAVE
    } else if (rdbSaveBackground(server.rdb_filename) == REDIS_OK) {
        addReplyStatus(c,"Background saving started");

    } else {
        addReply(c,shared.err);
    }
}

int rdbSaveBackground(char *filename) {
    pid_t childpid;
    long long start;
......
    start = ustime();

    if ((childpid = fork()) == 0) {
        int retval;

        /* Child */
        
        // 关闭网络连接 fd
        closeListeningSockets(0);

        // 设置进程的标题，方便识别
        redisSetProcTitle("redis-rdb-bgsave");

        // 执行保存操作
        retval = rdbSave(filename);
......

        // 向父进程发送信号
        exitFromChild((retval == REDIS_OK) ? 0 : 1);

    } else {

......
    }

    return REDIS_OK; /* unreached */
}


/* Save the DB on disk. Return REDIS_ERR on error, REDIS_OK on success 
 *
 * 将数据库保存到磁盘上。
 *
 * 保存成功返回 REDIS_OK ，出错/失败返回 REDIS_ERR 。
 */
int rdbSave(char *filename) {
    dictIterator *di = NULL;
    dictEntry *de;
    char tmpfile[256];
    char magic[10];
    int j;
    long long now = mstime();
    FILE *fp;
    rio rdb;
    uint64_t cksum;

    // 创建临时文件
    snprintf(tmpfile,256,"temp-%d.rdb", (int) getpid());
    fp = fopen(tmpfile,"w");
    if (!fp) {
        redisLog(REDIS_WARNING, "Failed opening .rdb for saving: %s",
            strerror(errno));
        return REDIS_ERR;
    }

    // 初始化 I/O
    rioInitWithFile(&rdb,fp);

    // 设置校验和函数
    if (server.rdb_checksum)
        rdb.update_cksum = rioGenericUpdateChecksum;

    // 写入 RDB 版本号
    snprintf(magic,sizeof(magic),"REDIS%04d",REDIS_RDB_VERSION);
    if (rdbWriteRaw(&rdb,magic,9) == -1) goto werr;

    // 遍历所有数据库
    for (j = 0; j < server.dbnum; j++) {

        // 指向数据库
        redisDb *db = server.db+j;

        // 指向数据库键空间
        dict *d = db->dict;

        // 跳过空数据库
        if (dictSize(d) == 0) continue;

        // 创建键空间迭代器
        di = dictGetSafeIterator(d);
        if (!di) {
            fclose(fp);
            return REDIS_ERR;
        }

        /* Write the SELECT DB opcode 
         *
         * 写入 DB 选择器
         */
        if (rdbSaveType(&rdb,REDIS_RDB_OPCODE_SELECTDB) == -1) goto werr;
        if (rdbSaveLen(&rdb,j) == -1) goto werr;

        /* Iterate this DB writing every entry 
         *
         * 遍历数据库，并写入每个键值对的数据
         */
        while((de = dictNext(di)) != NULL) {
            sds keystr = dictGetKey(de);
            robj key, *o = dictGetVal(de);
            long long expire;
            
            // 根据 keystr ，在栈中创建一个 key 对象
            initStaticStringObject(key,keystr);

            // 获取键的过期时间
            expire = getExpire(db,&key);

            // 保存键值对数据
            if (rdbSaveKeyValuePair(&rdb,&key,o,expire,now) == -1) goto werr;
        }
        dictReleaseIterator(di);
    }
    di = NULL; /* So that we don't release it again on error. */

    /* EOF opcode 
     *
     * 写入 EOF 代码
     */
    if (rdbSaveType(&rdb,REDIS_RDB_OPCODE_EOF) == -1) goto werr;

    /* 
     * CRC64 校验和。
     */
    cksum = rdb.cksum;
    memrev64ifbe(&cksum);
    rioWrite(&rdb,&cksum,8);

    /* Make sure data will not remain on the OS's output buffers */
    // 冲洗缓存，确保数据已写入磁盘
    if (fflush(fp) == EOF) goto werr;
    if (fsync(fileno(fp)) == -1) goto werr;
    if (fclose(fp) == EOF) goto werr;

    /* 
     * 使用 RENAME ，原子性地对临时文件进行改名，覆盖原来的 RDB 文件。
     */
    if (rename(tmpfile,filename) == -1) {
        redisLog(REDIS_WARNING,"Error moving temp DB file on the final destination: %s", strerror(errno));
        unlink(tmpfile);
        return REDIS_ERR;
    }

    // 写入完成，打印日志
    redisLog(REDIS_NOTICE,"DB saved on disk");

    // 清零数据库脏状态
    server.dirty = 0;

    // 记录最后一次完成 SAVE 的时间
    server.lastsave = time(NULL);

    // 记录最后一次执行 SAVE 的状态
    server.lastbgsave_status = REDIS_OK;

    return REDIS_OK;

werr:
    // 关闭文件
    fclose(fp);
    // 删除文件
    unlink(tmpfile);

    redisLog(REDIS_WARNING,"Write error saving DB on disk: %s", strerror(errno));

    if (di) dictReleaseIterator(di);

    return REDIS_ERR;
}

```

![](http://cache410.oss-cn-beijing.aliyuncs.com/rdbduring.png)

## 12.3 配置

```ini
#file: redis.conf
############SNAPSHOTTING#######
......
#当900秒执行1个写命令时，启用快照备份
save 900 1
#当300秒执行10个写命令时，启用快照备份
save 300 10
#当60秒内执行10000个写命令时，启用快照备份
save 60 10000
......
#Redis 执行 save 命令的时候，将禁止写入命令
#在默认情况下，如果 Redis 执行 bgsave 失败后，Redis 将停止接受写操作，这样以一种强硬的方式让用户知道数据不能正确的持久化到磁盘，否则就会没人注意到灾难的发生，如果后台保存进程重新启动工作了，Redis 也将自动允许写操作。然而如果安装了靠谱的监控，可能不希望 Redis 这样做，那么你可以将其修改为 no。
stop-writes-on-bgsave-error yes
......
#这个命令意思是是否对 rbd 文件进行检验，如果是将对 rdb 文件检验。从 dbfilename 的配置可以知道，rdb 文件实际是 Redis 持久化的数据文件。
rdbcompression yes
......
dbfilename dump.rdb

```

## 12.4 文件类型

RDB保存的文件是一个二进制文件，可以通过redis自带的工具`redis-check-dump`来查看里面的内容