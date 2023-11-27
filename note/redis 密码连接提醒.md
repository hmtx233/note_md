#2023-09-22 #redis
# Redis如何解决"Warning: Using a password with .....may not be safe"
### 连接Redis的时候，加上参数--no-auth-warning
``` shell
# redis-cli -h 127.0.0.1 -p 6379 -a "*********" --no-auth-warning
```

### 使用auth password

```shell
$ redis-cli -h host -p 6379  
$ auth password
```