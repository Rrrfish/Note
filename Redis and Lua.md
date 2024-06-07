# `Redis` and `Lua`

在学习`Redis`的分布式锁的时候，了解到一个轻量级的脚本语言`Lua`，因为`Redis`执行脚本时保证了原子性，进而使用`Lua`可以保证锁的原子性。



## 使用`Redis`+`Lua`的几种方法

### 命令行交互式

#### `Redis`语句

```shell
SET lock_key unique_value NX PX 10000
```

这个`Redis`语句中，尝试设置名为`lock_key`的锁：

- `NX`表示，如果键不存在则插入`lock_key unique_value`，否则不插入。这保证了锁的互斥性。
- `PX 10000`避免了持有锁的程序异常而导致的无法释放锁。
- `unique_value`表示持有锁的进程的唯一标识，只有`unique_value`对应的进程才可以解锁（删除锁）。

#### `Lua`脚本

```lua
if redis.call("get", KEY[1]) == ARGV[1] then
    return redis.call("del", KEY[1])
else
    return 0
end
```

- 使用 `redis.call("get", KEYS[1])` 获取 `lock_key` 的当前值。

- 检查这个值是否等于传入的 `unique_value` (`ARGV[1]`)。

- 如果相等，使用 `redis.call("del", KEYS[1])` 删除这个键，释放锁。

- 如果不相等，返回 0，表示操作未执行（锁未被释放）。



#### 执行`Lua`脚本

在`Redis`客户端中输入如下命令，执行`Lua`脚本

```shell
EVAL "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end" 1 lock_key unique_value
```

这里的 `EVAL` 命令用于执行 `Lua` 脚本，`1` 表示脚本中用到一个键（`lock_key`），后面跟着的是键名和参数。



以下是实践内容：
```shell
127.0.0.1:6379> SET lock_key unique_value NX PX 100000
OK
127.0.0.1:6379> gET lock_key
"unique_value"
127.0.0.1:6379> EVAL "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end" 1 lock_key wrong_value
(integer) 0
127.0.0.1:6379> EVAL "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end" 1 lock_key unique_value
(integer) 1
```

可以观察到，使用`wrong_value`无法解锁，而`unique_value`可以解锁。















