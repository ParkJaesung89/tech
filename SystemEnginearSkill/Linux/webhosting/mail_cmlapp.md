---
title: "webhosting_mail_cmlapp"
---

```toc

```

# Info

```bash
cmlapp801 66.232.138.51
cmlapp802 66.232.138.60
```

# Intro

http://sitemail.hostway.co.kr 의 수신용 메일 서버 제공

standard mail service 에서는 sendmail 을 이용한다.

advanced mail service 에서는 postfix 를 이용한다.

cmlapp801 -> cmlmct801 -> cmlapp801

# cmlapp801

## History

210819: Zabbix IO Wait Alarm Disable (백업)

```bash
Problem: {cmlapp801.siteprotect.co.kr:system.cpu.util[,iowait].min(5m)}>50 and {cmlapp801.siteprotect.co.kr:system.cpu.util[,iowait].time()}<071500 and {cmlapp801.siteprotect.co.kr:system.cpu.util[,iowait].time()}>075000

Recovery: {cmlapp801.siteprotect.co.kr:system.cpu.util[,iowait].max(10m)}<50 and {cmlapp801.siteprotect.co.kr:system.cpu.util[,iowait].time()}<071500 and {cmlapp801.siteprotect.co.kr:system.cpu.util[,iowait].time()}>075000
```

# cmlapp802

## History

210819: Zabbix IO Wait Alarm Disable (백업)

수요일 마다 06:15 ~ 07:10 알람 발생 안하도록 설정

수요일 06:15 ~ 07:10 를 제외하고 알람 발생 하도록 설정

```bash
{cmlapp802.siteprotect.co.kr:system.cpu.util[,iowait].min(5m)}>20 and {cmlapp802.siteprotect.co.kr:system.cpu.util[,iowait].dayofweek()}<>3 and {cmlapp802.siteprotect.co.kr:system.cpu.util[,iowait].time()}<061500 and {cmlapp802.siteprotect.co.kr:system.cpu.util[,iowait].time()}>071000

{cmlapp802.siteprotect.co.kr:system.cpu.util[,iowait].max(10m)}<20 and {cmlapp802.siteprotect.co.kr:system.cpu.util[,iowait].dayofweek()}<>3 and {cmlapp802.siteprotect.co.kr:system.cpu.util[,iowait].time()}<061500 and {cmlapp802.siteprotect.co.kr:system.cpu.util[,iowait].time()}>071000
```
