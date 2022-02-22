---
title: "webhosting_lsh"
---

```toc

```

# Info

샴페인

```bash
hartford 66.232.138.12
dover 66.232.138.14
topeka 66.232.138.15
newyork 66.232.138.10
boston 66.232.138.11
atlanta 66.232.138.13
lsh800 66.232.138.16
lsh801 66.232.138.17
lsh802 66.232.138.18
lsh803 66.232.138.19
lsh804 66.232.138.20
lsh805 66.232.138.21
lsh806 66.232.138.22
lsh807 66.232.138.23
lsh808 66.232.138.24
lsh809 66.232.138.25
lsh810 66.232.138.26
lsh811 66.232.138.27
lsh812 66.232.138.42
lsh813 66.232.138.43
lsh814 66.232.138.44
lsh815 66.232.138.45
lsh816 66.232.138.46
parking 66.232.138.55
iwant   61.100.13.105
```

# WebHosting Connection

## Denver1

웹호스팅 서버 접속 방법

```bash
ssh 10.30.0.21
## Denver1 서버 접속

su9 ssh madison
## root 권한 획득 목적

ssh 10.30.0.21
ssh [hostname]
```

## Host Machine

```bash
66.232.137.15 / administrator / H0stW@y2016@)!^

4F_B15_A1
```

# Init Script

최초 OS 부팅 후, 실행되는 내역

서버 특이사항으로 자동 재부팅 시, 하기 rc.local 정상 실행되었는지 확인 권장

```bash
/etc/rc.d/rc.local
```

# Schedule

## Cron

아래 3가지 모두 확인 필요

```bash
crontab -e

vi /etc/crontab

vi /etc/cron.d/
```

# Firewall

## Rule Add

향후 UTM 장비로 교체한다고 하지만, 아직 lfw 서버가 사용중이다.

특이사항 발생 시, lfw802 혹은 lfw805 로 접속한다.

웹호스팅 서버에 특이사항 발생 시, 상단 방화벽(=lfw)에 정책 등록도 권장한다. (급하면 국제망 차단 필요)

```bash
cd /etc/rc.d/
vi badip.conf
## 기내용 참고하여 차단하고 싶은 IP 혹은 네트워크 대역 추가

./rc.fw
## badip.conf 정책 iptables 에 등록
```

# Apache

## File

아파치 주요 파일에 대해 기재한다.

```bash
vi /home/www/conf/httpd.conf
## 아파치 메인 설정 파일

ls /home/www/
## 웹호스팅 DocumentRoot PATH
```

## Service Restart

```bash
/usr/local/bin/httpd start
## 혹시 모르니 ps aux | httpd 로 httpd PATH 확인
```

# Zabbix

## Service Restart

```bash
killall zabbix_agentd

/sbin/zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf
```

# TroubleShoot

## Kernel Error

`8253 count too high! resetting..` 과 같이 커널 특이사항이 발견되는 경우 콘솔 접속 후, 서버 재시작

## Network Error

80 포트 정상 접속되는지 확인

```bash
telnet lsh800 80

netstat -anp | grep ESTABLISHED
## 자신의 IP 가 접속 중인지 내역 확인
```

Drop 된 패킷이 있는지 확인. 가상 머신이므로 해결이 어려운 경우 서버 재시작

```bash
ifconfig
```

## Apache Error

비정상 패킷 인입되는 경우 해당 IP 차단 혹은 국제망 차단

```bash
ps auxww | grep httpd | wc -l
## 프로세스 개수가 과하게 많은 경우 (=150개 이상)

netstat -anpt | grep 80
```

## Email Alarm Noti

웹호스팅에서 정체를 알 수 없는 알람이 메일을 통해 수신되는 경우가 있다.

해당 메일을 송신하는 프로그램은 news (=Encrypted perl script) 이다. 해당 프로세스는 cron 을 통해 1분마다 실행된다. (`* * * * * /usr/spectro/news2`)

해당 프로그램의 설정은 /root/.news.rc 을 사용한다.

# History

210819: SSH root 로그인 변경 및 국제망 차단

```
PermitRootLogin yes
->
PermitRootLogin without-password
```
