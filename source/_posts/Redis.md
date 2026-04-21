---
title: Redis Overview
date: 2022-04-21 14:09:00
category: Redis
tags:
    - cache
    - redis
---

An overview of redis.

## Data types

Use cases of different data structure:
1. **String**: User info
2. **Set**: Intrests label; Common friends. 
3. **List**: Message queue
4. **Hash**: Objects, like user data.
5. **ZSet**: Ranking


## Persistance

There are two core persistance mechanism, **RDB** and **AOF**.

### RDB

RDB is Redis Dataabse. It take snapshot of data in memory and save it to `dump.rdb` at specific intervals. The output file is very small since it only contains the final state.

But a significant drawback is that it might lose data if service shutdown before next snapshot.

### AOF

Stands for **Append-only file**, that means each **Write Command** (Like SET, DEL) will be appended to the `appendonly.aof`. When redis restart, commands in it will be replay to recover data.

You can decide to *always* write command to file immediately, or do it *everysec*, or take *no* action and let os decide, by set the `appendfsync`.

Thanks to this mechanism, it hardly never lose data. But it also makes the AOF file much bigger than RDB file, and the service restart is slower due to the replay.

### Hybrid

From redis 4.0, **Hybrid Persistence** was introduced.

The magic happens at **AOF Rewrite**, that create a new AOF file, 'compress' data in memory to RDB format and add it to the head of the new file. Then all following AOF command will also write to the new file. 

It can be trigger by `BGREWRITEAOF` command or the AOF proportion is too big(controlled by auto-aof-rewrite-percentage and auto-aof-rewrite-min-size).



