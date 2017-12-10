#MYSQL主从复制 常见问题和解决方法 

Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'系列一：

主库添加log-bin-index 参数后，从库报这个错误：Got fatal error 1236 from master when reading data from binary log: 'Could not find first log file name in binary log index file'

Got fatal error 1236 from master when reading data from binary log: 'could not find next log'

可以

```
stop slave;

reset slave;

start slave;
```
