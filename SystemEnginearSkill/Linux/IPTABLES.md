, 10.30.103.181, 10.30.103.182, 10.30.103.183
, 61.100.13.181, 61.100.13.182, 61.100.13.183


































































































/sbin/zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf


66.232.138.13
66.232.138.52
66.232.138.58
66.232.138.51
66.232.138.60
10.30.9.67
10.30.9.22


$IPTABLES -A INPUT -p tcp -s 61.100.13.181 --dport 10050 -m state --state NEW,ESTABLISHED -j ACCEPT
$IPTABLES -A INPUT -p tcp -s 61.100.13.182 --dport 10050 -m state --state NEW,ESTABLISHED -j ACCEPT
$IPTABLES -A INPUT -p tcp -s 61.100.13.183 --dport 10050 -m state --state NEW,ESTABLISHED -j ACCEPT



-A INPUT -s 61.100.13.181/32 -p tcp -m tcp --dport 10050 -m state --state NEW,ESTABLISHED -j ACCEPT
-A INPUT -s 61.100.13.182/32 -p tcp -m tcp --dport 10050 -m state --state NEW,ESTABLISHED -j ACCEPT
-A INPUT -s 61.100.13.183/32 -p tcp -m tcp --dport 10050 -m state --state NEW,ESTABLISHED -j ACCEPT




iptables -nL --line-numbers
iptables -I INPUT 21 -s 61.100.13.181/32 -p tcp -m tcp --dport 10050 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -I INPUT 21 -s 61.100.13.182/32 -p tcp -m tcp --dport 10050 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -I INPUT 21 -s 61.100.13.183/32 -p tcp -m tcp --dport 10050 -m state --state NEW,ESTABLISHED -j ACCEPT


ipchains -A TCPIY -s 61.100.13.181 -d $ALL 10050:10051 -p TCP -j ACCEPT
IPCHAINS -A TCPIY -s 61.100.13.182 -d $ALL 10050:10051 -p TCP -j ACCEPT
IPCHAINS -A TCPIY -s 61.100.13.183 -d $ALL 10050:10051 -p TCP -j ACCEPT

ipchains -A input -i eth1 -s 192.168.1.0/24 -d 0/0 -j ACCEPT


!wlsfh#201







# iptables 관리
vi /etc/sysconfig/iptables에 방화벽 룰 설정 파일존재
설정 적용 후 해당 설정 바로 적용하려면 service iptables restart
현재 설정을 저장해두려면 service iptables save
save된 파일은 /etc/sysconfig/iptables.save에 저장됨
이를 관리하기위해 iptables.save파일을 따로 복사해서 날짜별로 변경할때마다 관리하는게 좋음.