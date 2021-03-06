---
title: Redis 注意知识点整理
date: 2015-06-01
tags: [Redis]
---

Redis 默认支持16个数据库，可以通过配置参数databases来修改这一数字，客户端与Redis建立链接后会自动选择0号数据库，可以select 命令更换数据库 `select 1`

Redis 不支持自定义数据库的名字，以编号命名，也不支持为每个数据库设置不同的访问密码。多个数据库之间并不是完全隔离，`FLUSHALL`命令可以清空一个Redis实例中所有数据库中的数据，Redis中的数据库更像是一种命名空间，而不适合存储不同应用的数据。不同的应用应该使用不同的Redis实例存储。由于Redis非常轻量级，一个空的Redis实例占用内存1mb空间。

`keys` 命令需要遍历Redis中的所有键，当键的数量较多时会影响性能，不建议在生成环境中使用。

`DEL` 命令的参数不支持通配符号，可以结合Linux的管道和xargs命令实现删除所有复合规则的键。`redis-cli keys "user:*" | xargs redis-cli del` 或者`redis del redis-cli keys "user:*"`

一个字符串类型的键允许存储的最大容量为512MB，字符串类型是其他四中基本类型的基础 

Redis 的数据类型同样不支持数据类型嵌套

列表类型的内部是使用双向链表，搭配使用lpush和lpop或者rpop和rpush可以当做栈来使用。如果当成队列则搭配使用lpush和rpop或者rpush和lpop

Redis 保证一个事物中的所有命令要么执行，要么不执行，如果在发送exec命令前，客户端断了线，则Redis会清空事物队列。事物中的所有命令都不会执行。而一旦客户端发送了exec命令，所有的命令都会执行。即使客户端断线也没关系。因为Redis中记录了所有要执行的命令

Redis 事物在2.6.5版本之后 如果多个命令中有**语法错误**，则会忽略所有的命令，正确的命令也不会执行。在2.6.5版本之前会执行正确的命令，如果是**运行错误**比如使用不同类型的命令操作其他类型。这种错误在实际执行之前Redis是无法发现的。所以出现错误后，其他命令会正常执行

Redis 没有事物回滚的功能，需要自己手动收拾摊子

Redis 使用Watch 命令防止竞态条件

Watch 命令的作用只是当被监控的键值被修改后阻止之后的一个事物执行，而不能保证其他客户端不修改这个键值，所以需要在exec失败后重新执行整个函数

 

