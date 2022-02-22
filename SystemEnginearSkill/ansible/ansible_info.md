---
title: "ansible_info"
---

```toc

```


# ansible 개념

## 앤서블이란?
Ansible (앤서블)은 여러 개의 서버를 효율적으로 관리할 수 있게 해주는 환경 구성 자동화 도구입니다.

## 구조

[  제어노드  ]  - - - - - - - - - - [ 매니지드 노드 ]
      |
      |
      |--------------------
      |                   |
      |                   |
--[인벤토리]------    --[플레이북]------
|                |   |                |
|  매니지드노드1  |   | 1. db 설치     |
|  매니지드노드2  |   | 2. db 생성     |
|  매니지드노드3  |   | 3. 테이블 생성 |
|  매니지드노드4  |   |      .         |
|       .        |   |      .         |
|       .        |   |      .         |
|________________|   |________________|

### 제어 노드(Control node)
앤서블을 실행하는 노드.
'ansible' 이나 'ansible-playbook' 명령을 이용하여 제어 노드에서 관리 노드들을 관리.



### 매니지드 노드(Managed node)
앤서블로 관리되는 서버들을 매니지드 노드라고 합니다. 매니지드 노드에는 앤서블을 설치하지 않습니다.
<!-- 설치해도 무방함 -->



### 인벤토리(Inventory)
매니지드 노드 목록을 작성한 파일을 '인벤토리' 라고 지칭.
인벤토리 파일은 호스트 파일과 같이 작성하여 관리(hosts와 같으며 앤서블용 hosts를 생성해서 사용 가능).
인벤토리는 각 매니지드 노드에 대한 IP 주소, 호스트 정보, 변수와 같은 정보를 지정할 수 있습니다.

- 정적 인벤토리 : 직접 hosts에 서버 ip나 명칭을 작성
- 동적 인벤토리 : 동적(클라우드 환경같은 오토스케일링을 통해 서버들이 늘었다 줄었다 할 경우)인 환경에서 인벤토리 설정
                 직접 텍스트 파일에 호스트들을 넣었다 뺐다하기 쉽지 않기 때문에 인벤토리 플러그인을 사용.
            " ansible-doc -t inventory -l " 명령을 통해 인벤토리 플러그인 종류들을 확인 가능.
            *참고* - https://it-hangil.tistory.com/88

#### 설정 방법
ini, yaml 파일로 설정 가능.

예) webservers에 foo, bar 서버 목록, dbservers에 one, two, three 서버 목록 추가 시 ini, yaml 파일 설정방법

1. hosts.ini
```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```

2. hosts.yaml
```
all:
 hosts:
  mail.example.com:
   host_var: "local_var"            // 호스트 변수 사용(단일 호스트 아래에 작성)
  210.122.45.173:                   // ip로 작성됨.
 chiledren:
  webservers:
   hosts:
    foo.example.com:
    bar.example.com:
  dbservers:
   hosts:
    one.example.com:
    two.example.com:
    three.example.com:
   vars:                            // 그룹 변수 사용(그룹 안에 작성)
    db_id: "admin"
    db_passwd: "sjaksahffk."
   
  helloservers:
   hosts:
    hello-server[1:10]:             // hello-server1 ~ hello-server10 을 [1:10] 으로 짧게 작성 가능

 vars:                              // 전역변수로 사용(all 안에 작성)
  string_var: "A"
  number_var: 1
  boolean_var: "yes"
  list_var:
   - a
   - b
   - c
  dict_var:
   key_a: "val_a"
   key_b: "val_b"
   key_c: "val_c"
```

#### 인벤토리 사용방법
#ansible-playbook -i inventory.yaml             <!-- 'i' 옵션을 추가하여 원하는 인벤토리를 지정할 수 있음.-->



### 모듈(Module)
앤서블이 실행하는 코드 단위.
각 모듈은 데이터베이스 처리, 사용자 관리, 네트워크 장치 관리 등 다양한 용도로 사용.
단일 모듈을 호출하거나 플레이북에서 여러 모듈을 호출 할 수 있음.



### 태스크(Task)
앤서블의 작업 단위.
애드훅(ad-hoc)명령을 사용하여 단일 작업을 한 번 실행할 수 있습니다.



### ****플레이북 (Playbook)****
순서가 지정된 태스크 목록이 저장되어 지정된 작업을 해당 순서로 반복적으로 실행.
플레이 북에는 변수와 작업이 포함될 수 있다.
플레이 북은 YAML로 작성.
<!-- 어떻게 작업을 할 것인지에 대한 목록 -->

