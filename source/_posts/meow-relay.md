title: 使用Redsocks + MEOW + ss-server实现代理中转
date: 2015-02-22 02:22:31
tags: Anti-GFW
---

1. 安装所需依赖
--
```shell
apt-get install redsocks
```
将`/etc/redsocks.conf`替换为以下内容，之后重启redsocks服务
```
base {
  log_debug = off;
  log_info = on;
  daemon = on;
  user = redsocks;
  group = redsocks;
  redirector = iptables;
}
redsocks {
  local_ip = 127.0.0.1;
  local_port = 82;
  ip = 127.0.0.1;
  port = 4411;
  type = http-relay; 
}
redsocks {
  local_ip = 127.0.0.1;
  local_port = 445;
  ip = 127.0.0.1;
  port = 4411;
  type = http-connect;
}
```
在[这里](http://dl.chenyufei.info/shadowsocks/latest/)下载最新版本的Shadowsocks服务器，并解压，重命名为ss-server，写好config.json

使用`curl -L git.io/meowproxy | bash`安装MEOW，参照[README](https://github.com/renzhn/MEOW#配置)写好配置文件

使用`adduser`新建一个叫meow的用户，把刚才下载的MEOW可执行文件和~/.meow复制到该用户的~里面

将以下代码保存为relay_setup.sh
```
iptables -t nat -N SS

iptables -t nat -A PREROUTING -j SS

iptables -t nat -I SS -m owner --uid-owner meow -j RETURN
iptables -t nat -I SS -m owner --uid-owner redsocks -j RETURN
 
iptables -t nat -A SS -p tcp --dport 80 -j DNAT --to-destination 127.0.0.1:82
iptables -t nat -A SS -p tcp --dport 443 -j DNAT --to-destination 127.0.0.1:445
```
以root运行上述文件

使用`su -c "~/MEOW" -s /bin/bash meow`以meow用户运行代理服务器
直接使用`./ss-server`运行shadowsocks服务器，配置完成
