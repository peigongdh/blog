## shadowsocks

```
/usr/local/bin/ss-server -v -c /etc/shadowsocks-libev/config.json -f /var/run/shadowsocks-libev.pid
```

supervisorctl

[program:shadowsocks]
user=nobody
directory=/usr/local/shadowsocks
command=/usr/local/bin/ss-server -v -c /etc/shadowsocks-libev/config.json
process_name=%(program_name)s
autostart=true
redirect_stderr=true
stdout_logfile=/var/log/shadowsocks/server.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=0

[program:kcptun]
user=kcptun
directory=/usr/local/kcptun
command=/usr/local/kcptun/server_linux_amd64 -c "/usr/local/kcptun/server-config.json"
process_name=%(program_name)s
autostart=true
redirect_stderr=true
stdout_logfile=/var/log/kcptun/server.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=0

## kcptun操作

```
zhangpei supervisorctl
kcptun                           RUNNING   pid 2459, uptime 655 days, 12:23:06
supervisor> stop kcptun
kcptun: stopped
supervisor> start kcptun
kcptun: started
supervisor> status
kcptun                           RUNNING   pid 8690, uptime 0:00:04
supervisor> status
kcptun                           RUNNING   pid 8690, uptime 0:00:11
supervisor> quit
```

配置参数说明：
https://www.wangjunfeng.com.cn/2019/11/11/kcp-tun-config/

使用脚本：https://github.com/kuoruan/shell-scripts