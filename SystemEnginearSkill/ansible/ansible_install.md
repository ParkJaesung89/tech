---
title: "ansible_install"
---

```toc

```


# install ansible 방법

## Python
```
$ pip install ansible

-> 버전지정
$ pip install ansible==2.10.7
```

## Centos
```
$ yum install -y ansible 
```

## ubuntu
```
$ apt install ansible
```

## MacOS
```
$ brew install ansible
```


# ansible 구성 작업
## 사전 환경 구성

-ansible 버전 확인 <!-- 주요 경로들 확인 가능 -->
```
$ ansible --version
[
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/ansible/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr  9 2019, 14:30:50) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
]
```

### ssh 키 교환
1. 제어노드와 매니지드 노드에 ssh 키 생성 후 키 복사하여 저장.
```
$ ssh-keygen
$ ssh-copy-id <user>@<Node IP or Hostname>      예) ssh-copy-id root@master , ssh-copy-id ansible@210.122.45.173
```

2. ssh 접속하여 비밀번호 없이 접속 가능한지 확인

### hosts 설정
/etc/ansible/hosts.ini 에서
```
[all]
210.122.45.173
```
#### ansible ping 테스트
$ ansible all ping -i hosts.ini
$ ansible all ping -i hosts.ini -u ansible
<!-- 매니지드 노드의 ansible 계정으로 핑테스트 -->

-------------------------------------------------------

