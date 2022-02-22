---
title: "webhosting_ldb"
---

```toc

```

# Info

샴페인

```
lasvegas 66.232.138.6
ldb800 66.232.138.28
ldb801 66.232.138.54
ldb802 66.232.138.61
```

# History

211103: 기존 중첩된 processor load 알람 disabled 처리 후 하기와 같이 알람 임계치 수정

```
{ldb802.siteprotect.co.kr:system.cpu.load[percpu,avg1].min(15m)}>10
```

수정 이유: CPU 1 / MEM 2 인 스펙이며, MYISAM Engine 사용으로 UPDATE 문 실행 시, 테이블 Lock 발생 -> 기본 CPU Load 12 이상 증가

210823: mysql 백업으로 인해 03:15 ~ 04:10 알람 예외 처리 보류
