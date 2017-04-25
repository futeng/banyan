# iptables

## iptables 开启与关闭

```
# 当前生效，重启后失效
service iptables start  
service iptables stop 

# 配置随系统启动
chkconfig iptables on  
chkconfig iptables off  
```