## 登录dba账户
conn sys/fulong@orcl as sysdba;  

## 切换用户
conn user/password

## 查询用户
select * from all_users;
## 修改锁定账号
alter user beijing account unlock;  
## 修改用户密码
alter user beijing identified by fulong;  