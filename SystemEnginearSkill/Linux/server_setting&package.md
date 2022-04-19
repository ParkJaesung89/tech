curl net-tools epel-release wget tcping sysstat rsync iotop lsof htop chrony(rdate 실행 시 서버쪽 시간을 바로 적용. 시간 서버 – 클라이언트 60초의 오차가 있을 때 60초를 즉시 건너 뛴다.
그래서 시간이 빠른 장비의 시간 보정을 rdate로 하면 (장비 관점에서)시간이 과거로 돌아가는 문제가 있음 -> DB의 경우 과거 시간으로 데이터가 쌓이면 DB 테이블이 깨질 수 있음.
ntpd(chrony)는 오차 시간을 빠르게 혹은 느리게 시간을 돌려서 맞춰(slewing) rdate와 같은 문제 발생하지 않음.
시간 오차가 너무 클 때는 NTP 구현체도 step 방식으로 시간 보정.)


<user계정 생성>
useradd -m jaesung
passwd jaesung
su - jaesung

<sudo권한 설정>
vi /etc/sudoers
jaesung ALL=(ALL:ALL)	NOPASSWD:ALL
wd!

<root접속 막기>
sudo sed -i 's/#PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd.config
service sshd restart

<패키지 설치>
sudo yum install -y epel-release
sudo yum install -y  net-tools wget tcping sysstat htop iotop lsof chrony rsync
su - root

<zabbix 설치>
wget http://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-agent-5.0.7-1.el7.x86_64.rpm
rpm -Uvh zabbix-agent-5.0.7-1.el7.x86_64.rpm
cat /etc/hostname
hostnamectl set-hostname Jaesung_Server
hostname
sudo cp /etc/zabbix/zabbix_agentd.conf /etc/zabbix/zabbix_agentd.conf.ori
sudo sed -i 's/^Server=127.0.0.1/Server=61.100.13.181, 61.100.13.182, 61.100.13.183/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/^ServerActive=127.0.0.1/ServerActive=61.100.13.181, 61.100.13.182, 61.100.13.183/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/# HostnameItem/HostnameItem/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/# HostMetadataItem=/HostMetadataItem=system.uname/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/LogFileSize=0/LogFileSize=10/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/Hostname=Zabbix server/Hostname=Jaesung_Server/g' /etc/zabbix/zabbix_agentd.conf
cat /etc/zabbix/zabbix_agentd.conf
systemctl start zabbix-agent.service && systemctl enable zabbix-agent.service

<방화벽설정>
firewall-cmd --permanent  --remove-service=ssh
firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="211.115.223.215/32" port protocol="tcp" port="22" accept'
firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="211.115.221.192/27" port protocol="tcp" port="8400-8450" accept'
vi /etc/firewalld/zones/public.xml
firewall-cmd --reload

<컴볼트 설치>
파일은 파일질라로 넣음..
sudo echo -e "##commvault## \n211.115.221.196 Comm-Master-A \n211.115.221.197 Comm-Master-B \n211.115.221.198 Comm-MA-Combine \n203.248.23.233 Jaesung_server" >> /etc/hosts
cat /etc/hosts
cd /home/jaesung/
ls
mv custom_pkg_V11SP16.tar /root/
cd /root/
tar -xvf custom_pkg_V11SP16.tar
cd pkg/
tcping 211.115.221.196 8400
vi default.xml
./silent_install -p default
netstat -tlnp


sudo cp /etc/zabbix/zabbix_agentd.conf /etc/zabbix/zabbix_agentd.conf.ori
sudo sed -i 's/^Server=127.0.0.1/Server=61.100.13.181, 61.100.13.182, 61.100.13.183/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/^ServerActive=127.0.0.1/ServerActive=61.100.13.181, 61.100.13.182, 61.100.13.183/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/# HostnameItem/HostnameItem/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/# HostMetadataItem=/HostMetadataItem=system.uname/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/LogFileSize=0/LogFileSize=10/g' /etc/zabbix/zabbix_agentd.conf
sudo sed -i 's/Hostname=Zabbix server/Hostname=Jaesung_Server/g' /etc/zabbix/zabbix_agentd.conf


<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description> Commvault Backup Access</description>
  <service name="dhcpv6-client"/>
  <rule family="ipv4">
    <source address="211.115.223.215"/>
    <port protocol="tcp" port="22"/>
    <accept/>
  </rule>
  <rule family="ipv4">
    <source address="211.115.221.192/27"/>
    <port protocol="tcp" port="8400-8450"/>
    <accept/>
  </rule>
</zone>

firewall-cmd --permanent --zone=dmz --add-rich-rule='rule family="ipv4" source address="211.115.223.215/32" port protocol="tcp" port="22" accept'
firewall-cmd --permanent --zone=dmz --add-rich-rule='rule family="ipv4" source address="211.115.221.129/27" port protocol="tcp" port="22" accept'
