---
title: "tech_drbd_01"
---

```toc

```

# OS

```bash
# Server All
yum install -y ntp
systemctl enable --now ntp

vi /etc/ntp.conf

server time.bora.net
## 추가 / 다른 서버들 주석처리

systemctl restart ntp

vi /etc/selinux/config

SELINUX=disabled

hostnamectl set-hostname <Hostname>

vi /etc/hosts

203.248.23.162 drbd1
203.248.23.163 drbd2

fdisk /dev/sdb
mkfs.xfs /dev/sdb1

modprobe drbd

sudo reboot
```

# drbd Download 08

```bash
# Server All
yum install -y epel-release

sudo rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-5.el7.elrepo.noarch.rpm

yum install -y kmod-drbd84 drbd84-utils
```

# drbd Download 09

```bash
# Server All
rpm -Uvh https://mirror.rackspace.com/elrepo/elrepo/el7/x86_64/RPMS/kmod-drbd90-9.0.22-2.el7_8.elrepo.x86_64.rpm
## drbd

rpm -Uvh https://mirror.rackspace.com/elrepo/elrepo/el7/x86_64/RPMS/drbd90-utils-9.12.2-1.el7.elrepo.x86_64.rpm
## drbd-utils
```

# drbd Config

```bash
vi /etc/drbd.d/data.res

resource data { #resource 의 이름
  startup {
    wfc-timeout 30;
    outdated-wfc-timeout 20;
    degr-wfc-timeout 30;
  }

  net {
    cram-hmac-alg sha1;
    shared-secret sync_disk;
  }
  on drbd1 { # primary hostname
    device /dev/drbd0; # drbd 논리 블록 디바이스
    disk /dev/sdb1; # 실제 연결될 디스크
    address 203.248.23.162:7789; # primary 주소, 포트
    meta-disk internal;
  }
  on drbd2 { # secondary hostname
    device /dev/drbd0;
    disk /dev/sdb1;
    address 203.248.23.163:7789; # secondary 주소, 포트
    meta-disk internal;
  }
}
## 오타 및 IP 확인 (IP 잘못 기재 시 "BAD LUCK, equal hashes" 메세지 출력)
```

# drbd PreSet

```bash
# Server All
drbdadm create-md data
## metadata 생성
## 추가로 yes 물어보면 yes 입력

systemctl enable --now drbd
## drbd 시작

cat /proc/drbd
# 둘다 secondary로 나옴
## cs:Connected ro:Secondary/Secondary ds:UpToDate/UpToDate #primary
## cs:Connected ro:Secondary/Secondary ds:UpToDate/UpToDate #secondary
# 주의사항
## drbd 9버전은 /proc/drbd0 에 연동 정보가 남지 않아 "drbdadm status" 명령어로 확인 진행

drbdadm status
```

# drbd PostSet

```bash
# Master Node
drbdadm primary --force data
## Master 서버 drbd를 Primary 로 전환
```

```bash
# Server All
cat /proc/drbd
# primary가 되면 secondary와 동기화 시작
# sync 진행을 통해 100%가 되면 완전 동기화 됨
## cs:Connected ro:Primary/Secondary ds:UpToDate/UpToDate  #master
## cs:Connected ro:Secondary/Primary ds:UpToDate/UpToDate  #slave

mkdir /drbd	# Mount 포인트 생성
```

```bash
# Master Node
mount /dev/drbd0 /drbd 	# drbd Mount (Master 서버)
## 참고 : secondary는 Mount 불가 , Primary만 Mount 가능

df -Th
```

# drbd Test Role Change

Primary / Secondary 역할을 변경해보자.

```bash
# Master Node
umount /drbd

systemctl stop drbd
```

```bash
# Slave Node
drbdadm primary data
## Master 서버 중지 후 Slave 서버의 drbd를 Primary로 전환

mount /dev/drbd0 /drbd
```
