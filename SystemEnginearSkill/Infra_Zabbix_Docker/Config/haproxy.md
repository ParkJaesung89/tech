# haproxy config
## haproxy.cfg
```bash
cat /etc/haproxy/haproxy.cfg | grep -v "#" | grep -v ^$
global
    log 127.0.0.1 local2 info
    log 127.0.0.1 local2 notice
    chroot      /etc/haproxy
    pidfile     /run/haproxy.pid
    maxconn     4000
    user        root
    daemon
    stats socket /run/haproxy-master.sock level admin
defaults
    log     global
    mode    http
    option  httplog clf
    option  dontlognull
    option  forwardfor
    option  http-server-close
    timeout http-request    10s
    timeout client          20s
    timeout server          30s
    timeout connect         4s
    timeout http-keep-alive 10s
frontend web-in
    bind *:80
    maxconn 4000
    capture request header Host len 128
    capture request header User-Agent len 64
    capture request header Referrer len 64
    log-format "%ci\ [%trl]\ %HM\ \"%HU\"\ \"%HV\"\ %ST\ %B\ %hr\ %s\ %b\ %TR/%Tw/%Tc/%Tr/%Ta\ %ac/%fc/%bc/%sc/%rc"
    default_backend             web-back
backend web-back
    balance     roundrobin
    option  httpchk HEAD /_health HTTP/1.1\r\nHost:\ localhost
    errorfile 400 /etc/haproxy/errors/400.http
    errorfile 403 /etc/haproxy/errors/403.http
    errorfile 408 /etc/haproxy/errors/408.http
    errorfile 500 /etc/haproxy/errors/500.http
    errorfile 502 /etc/haproxy/errors/502.http
    errorfile 503 /etc/haproxy/errors/503.http
    errorfile 504 /etc/haproxy/errors/504.http
    http-request set-header X-Forwarded-Port %[dst_port]
    server  zabbix-node01   10.30.97.56:80  check fall 3 rise 2
    server  zabbix-node02   10.30.97.58:80  check fall 3 rise 2
    server  zabbix-node03   10.30.97.58:80  check fall 3 rise 2

```

# haproxy log
## rsyslog.conf
```bash
sed -i 's/#$ModLoad imudp/$ModLoad imudp/g' /etc/rsyslog.conf
sed -i 's/#$UDPServerRun 514/$UDPServerRun 514/g' /etc/rsyslog.conf
sed -i 's/#$ModLoad imtcp/$ModLoad imtcp/g' /etc/rsyslog.conf
sed -i 's/#$InputTCPServerRun 514/$InputTCPServerRun 514/g' /etc/rsyslog.conf

```

## haproxy.conf
```bash
cat <<EOF > /etc/rsyslog.d/haproxy.conf
local2.info    /home/haproxy/log/haproxy_info.log
local2.notice    /home/haproxy/log/haproxy_notice.log
& stop
EOF
```

## logrotate
```bash
cat << EOF > /etc/logrotate.d/haproxy
/var/log/haproxy/*.log {
    daily
    rotate 90
    create 0644 nobody nobody
    missingok
    notifempty
    compress
    sharedscripts
    postrotate
        /bin/systemctl restart rsyslog.service > /dev/null 2>/dev/null || true
    endscript
}
EOF

```

