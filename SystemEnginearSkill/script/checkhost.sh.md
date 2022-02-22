---
title: "checkhost.sh"
---

```toc

```

#!/bin/bash

OK_COUNT=0
FAIL_COUNT=0
DISCRETER="++++++++++++++++++++++++++++++++++++++++++++++++++++++"

echo $DISCRETER
echo "[wise]"
command=`ps -ef |  grep wise | grep -v grep |  wc -l`
if [ $command -eq 1 ]; then OK_COUNT=$((OK_COUNT+1)); else FAIL_COUNT=$((FAIL_COUNT+1)); fi
echo $command
echo $DISCRETER

echo $DISCRETER
echo "[mysql]"
command=`systemctl is-active mariadb`
if [ $command = 'active' ]; then OK_COUNT=$((OK_COUNT+1)); else FAIL_COUNT=$((FAIL_COUNT+1)); fi
echo $command
echo $DISCRETER

echo $DISCRETER
echo "[cloudstack-management]"
command=`systemctl is-active cloudstack-management`
if [ $command = 'active' ]; then OK_COUNT=$((OK_COUNT+1)); else FAIL_COUNT=$((FAIL_COUNT+1)); fi
echo $command
echo $DISCRETER

echo $DISCRETER
echo "[free]"
free -m
echo "(criteria: swap 372)"
echo $DISCRETER

echo $DISCRETER
echo "[df]"
df -Th
echo "(criteria: 13%)"
echo $DISCRETER

echo $DISCRETER
echo "[ntp]"
ntpq -p
echo $DISCRETER

echo $DISCRETER
echo "[sar]"
sar -s 07:00:00 -e 10:00:00
echo "(criteria: 12%)"
echo $DISCRETER

echo $DISCRETER
echo "[dmesg]"
dmesg -T | tail -n 10
echo $DISCRETER

echo $DISCRETER
echo "[/var/log/messages]"
cat /var/log/messages | tail -n 20
echo $DISCRETER

echo $DISCRETER
echo "[/var/log/cloudstack/management-server.log]"
cat /var/log/cloudstack/management/management-server.log | tail -n 20
echo $DISCRETER

echo $DISCRETER
echo "[netstat]"
netstat -anp | grep ESTABLISHED | wc -l
echo "(criteria: 50)"
echo $DISCRETER

echo $DISCRETER
echo "[wc logs]"
du -sh /Edge/portal/wc/logs/
du -sh /Edge/portal/wcadmin/logs/
echo "(criteria: 13 / 7.0)"
echo $DISCRETER

echo $DISCRETER
echo "[uptime]"
uptime
echo $DISCRETER

echo $DISCRETER
echo TOTAL: $((OK_COUNT+FAIL_COUNT))
echo OK_COUNT: $OK_COUNT
echo FAIL_COUNT: $FAIL_COUNT
echo $DISCRETER