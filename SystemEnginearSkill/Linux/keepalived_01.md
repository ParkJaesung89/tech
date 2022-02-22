---
title: "tech_keepalived_01"
---

```toc

```

# Install

```bash
yum install -y keepalived
systemctl enable --now keepalived
```

# Config: Master

```bash
vi /etc/keepalived/keepalived.conf

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server localhost
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state MASTER # MASTER or BACKUP
    interface enp0s3
    virtual_router_id 100 # master와 slave가 같은 가상라우터 id 값이 필요
    priority 250 # 값이 높을 수록 failover 시, master 우선순위
    advert_int 3 # VRRP 패킷 송신 초단위 간격, slave와 되도록 같은 값 추천
    authentication {
        auth_type PASS
        auth_pass password
    }
    virtual_ipaddress {
        203.248.23.161/25 dev enp0s3
        # vip 주소와 디바이스 정보
    }
    notify_master /usr/local/bin/to_master.sh
    notify_backup /usr/local/bin/to_backup.sh
    notify_fault /usr/local/bin/to_fault.sh
    # 스크립트 적용
}
```

# Config: Backup

```bash
vi /etc/keepalived/keepalived.conf

global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server localhost
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_2 { # vrrp 인터페이스 이름 마스터와 다르게 변경할 것.
    state BACKUP
    interface enp0s3
    virtual_router_id 100
    priority 200 # Master 보다 낮게 설정
    advert_int 3
    authentication {
        auth_type PASS
        auth_pass password
    }
    virtual_ipaddress {
        203.248.23.161/25 dev enp0s3
    }
    notify_master /usr/local/bin/to_master.sh
    notify_backup /usr/local/bin/to_backup.sh
    notify_fault /usr/local/bin/to_fault.sh
}
```

# Apply

```bash
systemctl restart keepalived
```

# Check

```bash
ip addr
```
