#2023-11-14 #oracle
# 登录dba账户
```sql
conn sys/fulong@orcl as sysdba;  
```

# 切换用户
```sql
conn user/password
```

# 查询用户
```sql
select * from all_users;
```

# 修改锁定账号
```sql
alter user beijing account unlock;  
```
# 修改用户密码
```sql
alter user beijing identified by fulong;  
```