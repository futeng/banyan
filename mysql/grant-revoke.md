# Grant/Revoke

## 记忆

```
grant all privileges on DB.TABLE  to 'user'@'host' identified by 'password';

flush privileges;

# grant all privileges on DB.*  to 'user'@'%' ;
```


1. **授予（Grant）** 允许什么样的**操作** 对 **数据库.表** 给什么**用户**（来自哪台机器），充分展示了英语句式相对于汉语的倒置结构；
2. 支持授权时创建该用户；
2. revoke 语法类似，to 换成 from；

