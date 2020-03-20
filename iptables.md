# iptables 安全配置
```bash
apt install ipset -y
ipset create IPSET_LIST hash:ip hashsize 4096 maxelem 10000000 timeout 3600000
ipset create WHITE_LIST  hash:ip

ipset flush IPSET_LIST 
ipset flush WHITE_LIST 
iptables -F
iptables -A INPUT -m set --match-set WHITE_LIST src -j ACCEPT  
iptables -A INPUT -m set --match-set IPSET_LIST src -j DROP
iptables -A INPUT -m state --state NEW -m recent --name BAD_CC_ACCESS --update -p tcp -m tcp --dport 22086 --seconds 1 --hitcount 50 -j SET --add-set IPSET_LIST src
iptables -A INPUT -m recent --name BAD_CC_ACCESS --set -j ACCEPT 
iptables -A OUTPUT -o eth0 -m hashlimit --hashlimit-above 1000kb/s --hashlimit-mode dstip --hashlimit-name out -j DROP
```

> 添加下一物理层节点、监控、日志收集等节点ip 至 白名单 WHITE_LIST
> 如 ipset add WHITE_LIST  34.92.58.164


# nginx 相关配置
```
# 主配置文件 http 标签中添加内容：
limit_req_zone $binary_remote_addr zone=one:100m rate=30r/m;


# 其他配置文件server标签中添加以下内容：
limit_req zone=one burst=5 nodelay;
```