---
title: "zabbix_agent_install"
---

```toc

```

# Zabbix_agent install guide

## 레포지터리 다운방식

1. OS 버전별로 실행

### CentOS6
```
#rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/6/x86_64/zabbix-release-4.0-2.el6.noarch.rpm

#yum install -y zabbix-agent
```


### CentOS7
```
#rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm

#yum install -y zabbix-agent
```


### CentOS8
```
#rpm -Uvh https://repo.zabbix.com/zabbix/4.0/rhel/8/x86_64/zabbix-release-4.0-2.el8.noarch.rpm

#yum install -y zabbix-agent
```


### Ubuntu 16
```
#wget https://repo.zabbix.com/zabbix/4.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_4.0-2+xenial_all.deb 

#sudo dpkg -i zabbix-release_4.0-2+xenial_all.deb

#sudo apt-get udpate

#sudo apt-get install -y zabbix-agent
```


### Ubuntu 18
```
#wget https://repo.zabbix.com/zabbix/4.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_4.0-3+bionic_all.deb 

#sudo dpkg -i zabbix-release_4.0-3+bionic_all.deb

#sudo apt-get udpate

#sudo apt-get install -y zabbix-agent
```


2. Zabbix_agent 설정 변경
```
#sudo cp /etc/zabbix/zabbix_agentd.conf /etc/zabbix/zabbix_agentd.conf.ori

#sudo sed -i 's/^Server=127.0.0.1/Server=zabbix.hostway.co.kr/g' /etc/zabbix/zabbix_agentd.conf

#sudo sed -i 's/^ServerActive=127.0.0.1/ServerActive=zabbix.hostway.co.kr/g' /etc/zabbix/zabbix_agentd.conf

#sudo sed -i 's/# HostnameItem/HostnameItem/g' /etc/zabbix/zabbix_agentd.conf

#sudo sed -i 's/# HostMetadataItem=/HostMetadataItem=system.uname/g' /etc/zabbix/zabbix_agentd.conf

#sudo sed -i 's/LogFileSize=0/LogFileSize=10/g' /etc/zabbix/zabbix_agentd.conf

#sudo vi /etc/zabbix/zabbix_agentd.conf
    Hostname=<서버의 hostname 확인하여 넣기>    //앞 주석 제거
```

<!--
zabbix_agentd.conf 안에서 hostname 설정 우선순위 경우 2가지

1) Hostname=<hostname>
   HostnameItem=system.hostname
   -> 이 경우 zabbix server에서 host 등록시 hostname 이름으로 맞춤

2) #Hostname=<hostname>
   HostnameItem=system.hostname
   -> 이 경우 zabbix server에서 host 등록시 os 시스템의 호스트이름으로 맞춤

-->

3. zabbix server에서 agent 서버 추가











---
# zabbix_proxy 설정

1.	앞서 설명한 Zabbix repository 추가 후, 하기 진행
A.	Ubuntu
i.	apt install mysql-server zabbix-proxy-mysql -y
ii.	systemctl enable mysql
iii.	systemctl start mysql
iv.	mysql -u root -p 
v.	create database zabbix_proxy character set utf8 collate utf8_bin;
vi.	grant all privileges on zabbix_proxy.* to zabbix@localhost identified by 'password';
vii.	flush privileges;
viii.	exit;
ix.	zcat /usr/share/doc/zabbix-proxy-mysql/schema.sql.gz | mysql -u zabbix -p zabbix_proxy
x.	egrep "Server=|HostnameItem=|DBHost=|DBName=|DBUser=|DBPassword=" /etc/zabbix/zabbix_proxy.conf | grep -v ^#
Server=zabbix.hostway.co.kr
HostnameItem=system.hostname
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=password
xi.	systemctl enable zabbix-proxy
xii.	systemctl start zabbix-proxy
B.	CentOS
i.	yum -y install mariadb-server zabbix-proxy-mysql -y
ii.	systemctl enable mariadb
iii.	systemctl start mariadb
iv.	mysql -u root -p
v.	create database zabbix_proxy character set utf8 collate utf8_bin;
vi.	grant all privileges on zabbix_proxy.* to zabbix@localhost identified by 'password';
vii.	flush privileges;
viii.	exit;
ix.	zcat /usr/share/doc/zabbix-proxy-mysql-4.0.13/schema.sql.gz | mysql -u zabbix -p zabbix_proxy
x.	egrep "Server=|HostnameItem=|DBHost=|DBName=|DBUser=|DBPassword=" /etc/zabbix/zabbix_proxy.conf | grep -v ^#
Server=zabbix.hostway.co.kr
HostnameItem=system.hostname
DBName=zabbix_proxy
DBUser=zabbix
DBPassword=password
xi.	systemctl enable zabbix-proxy
xii.	systemctl start zabbix-proxy
2.	Zabbix Dashboard 로 이동
A.	Administrator -> Proxies




