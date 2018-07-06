# redis-lua-scripts


Redis在`2.6`版本之后可以使用`lua`脚本
Redis在`2.8`版本之后可以使用`scan`命令
Redis在`3.2`版本之后可以使用`redis.replicate_commands()`命令


快速填充数据
```
eval "for i=0,9999 do redis.call('set','prefix:'..i, i) end" 0
```

不安全删除Redis数据(Redis会阻塞，可以用在数据量少的情况下,禁止在生产环境执行)
```
eval "redis.call('del', unpack(redis.call('keys', 'prefix:*')))" 0
```

统计某前缀的数据
```
eval "local cursor = '0' local count = 0 repeat local result = redis.call('SCAN', cursor, 'MATCH', 'prefix:*', 'COUNT', 500) cursor = result[1] count = count + #result[2] until cursor == '0' return count" 0
```


安全快速删除Redis数据(`3.2`版本以上)
```
eval "redis.replicate_commands() local cursor = '0' local delete_count = 0 repeat local result = redis.call(         'scan', cursor, 'match', 'prefix:*', 'count', 100 ) cursor = result[1] local list = result[2] delete_count = delete_count + #list redis.call('del', unpack(list)) until (cursor == '0') return delete_count" 0
```

