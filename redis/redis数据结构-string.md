---
title: redis数据结构-string
date: 2018-12-14
categories: redis
tags: 学习笔记
description: 
---

string类型是redis最基础的数据结构。其他几种数据结构都是在string基础上构建的。string类型的value可以是字符串，数字，二进制，value最大不能超过512M。

### 常用命令

1. 设置值: `set key value [ex] [px] [nx|xx]`
- ex 为key设置秒级过期时间

- px 为key设置毫秒级过期时间

- nx 当key不存在时，进行设置 

- xx 当可以存在时，进行设置，相当于更新操作

```redis
示例
set name hello

# 设置过期时间为1秒
set name hello EX 1

# 设置过期时间为1000毫秒 
set name hello PX 1000


```

`setnx key value`的作用等同于`set key value nx`。`setnx`命令可以作为分布式锁的一种实现方案。有关redis实现分布式锁: http://redis.io/topics/distlock

`setex key value`等同于`set key value ex 1`



2. 获取值: `get key`

```redis
get name
```

3. 批量设置值: `mset key value [key value ...]`

```redis
# 一次设置3个key(name, age, sex)的值
mset name hello age 16 sex boy
```

4. 批量获取值: `mget key [key ...]`

```redis
# 一次获取3个key的值
mget name, age, sex
```

5. `incr`命令用于对值做自增，返回结果分为3种情况
   
   - value 不是整数，返回错误
   
   - value 为整数，返回自增后的结果
   
   - key 不存在，按照值为0自增，返回1。

```redis

```

Redis还提供了`decr(递减)`、`incrby(自增指定数字)`、`decrby`、`incrbyfloat`

更多string相关命令: https://redis.io/commands

### 内部编码

string类型的内部编码有3种：

- int：8个字节的长整型；

- embstr：小于等于44个字节的字符串；

- raw：大于44个字节的字符串。

Redis会根据当前值的类型和长度决定使用哪种内部编码实现。

### 使用场景

1. 用作缓存，由于Redis具有支撑高并发的特性，所以缓存通常能起到加速读写和降低后端压力的作用。

2. 计数：Redis可以实现快速计数、查询缓存的功能，同时数据可以异步落地到其他数据源。

3. 共享Session：可以使用Redis将用户的Session进行集中管理，在这种模式下只要保证Redis是高可用和扩展性的，每次用户更新或者查询登录信息都直接从Redis中获取。

除了上述3种场景，string还有很多的使用场景，我们可以充分发挥自己的想象力。
