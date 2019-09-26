### 下载redis[WINDOWS](https://github.com/dmajkic/redis/downloads)[LINUX链接](http://www.redis.cn/download.html)
### 教程[链接](https://www.redis.net.cn/tutorial/3502.html)
> 本文档只做windows，下载后解压32/64位，改名redis，进入，启动服务端`redis-server.exe redis.conf`，不能关闭窗口，否则停止服务
> 重新打开一个cmd，进入redis文件夹，启动客户端`redis-cli.exe  -h 127.0.0.1 -p 6379`
> 添加一个值 `set [keyname] [keyvalue]`
> 查看值`get [keyname]`

### 修改配置文件
- `CONFIG GET CONFIG_SETTING_NAME`查看配置
- `CONFIG GET *` 查看所有配置
- `CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE` 赋值新值
> 数据类型
 - string `set [keyname] [keyvalue]`-->`get [keyname]`
   - `set key value` 赋值
   - `get key` 取值
   - `getrange key start end `返回 key 中字符串值的子字符
   - `getset key value` 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。
   - `getbit key offset`对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
   - `mget [key1] [key2]`获取所有(一个或多个)给定 key 的值
   - `mset key1 value1 key2 value2 `同时设置一个或多个 key-value 对。
   - `incr key` 将 key 中储存的数字值增一。
   - `decr key` key 中储存的数字值减一。
 - hash:键值对集合Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。 `hmset hashkey:hashvalue col1 col1value col2 col2value ` 例如 `hmset user:1 name weixiao arg 22` -->`hgetall hashkey:hashvalue` 例如`hgetall user:1`
 - List:列表
   - 添加`lpush [listName] [value]`
   - 查询`lrange [listName] [从末尾开始位置，从0开始] [查询结束的位置，从0开始]`
 - Set:集合 集合内元素的唯一性，第二次插入相同元素将被忽略。
   - 添加 `sadd key member`
   - 查看 `smembers key`
 - zset:(sorted set：有序集合)Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。zset的成员是唯一的,但分数(score)却可以重复。
   - 添加`zadd key score member`
   - 查询 `zrangebyscore key [score开始位置] [score结束位置]` 
 -  HyperLogLog HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。
    -  添加 `pfadd key value`
    -  查询数量 `pfcount key`
>
>语法
 - `redis-cli -h host -p port -a password` 启动客户端语法
 - `del key` 删除
 - `dump key`序列化
 - `exists key`判断是否存在
 - `expire key seconds` 设置过期时间
 - `expireat key timestamp `EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。
 - `pexpire key milliseconds` 设置 key 的过期时间亿以毫秒计。
 - `pxpireat  key timestamp `设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
 - `type key` 查看类型
 - `persist key` 移除 key 的过期时间，key 将持久保持
 - `pttl key` 以毫秒为单位返回 key 的剩余的过期时间。
 - `ttl key`以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。
 - `randomkey`从当前数据库中随机返回一个 key 
 - `move key db`将当前数据库的 key 移动到给定的数据库 db 当中。
 - `key pattern` 查找所有符合给定模式( pattern)的 key 。
 - `rename key newkey`修改 key 的名称
 - `renamenx key newkey`仅当 newkey 不存在时，将 key 改名为 newkey 
 - `multi .... exec` 事务执行,multi开始事务，exec执行事务
 - `ping` 查看是否运行
 - `ping` 查看是否运行
 - `quit`退出连接
 - `select index` 切换到指定的数据库
 - `echo message` 打印字符串
 - `info` 取 Redis 服务器的各种信息和统计数值
 - `time` 获取系统服务器时间
 - `config set requirepass passworld` 设置密码
 - `config get requirepass` 查询密码
 - `auth password`验证密码是否正确
 - `redis-cli -p 6379 -a passworld` 登录时候输入密码