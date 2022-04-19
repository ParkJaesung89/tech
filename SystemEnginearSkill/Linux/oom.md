<div class=”hostway-style-01”>

# 제목 : OOM Killer 에 대하여

* 본 글은 Kernel 5.4.0-104-generic 을 기반으로 작성하였습니다.

## OOM(Out Of Memory) 이란 ? 

Linux의 swap 메모리와 물리 메모리를 모두 사용 중 일 때, 프로세스가 추가적인 물리 메모리 할당을 요청 하여도  할당이 불가능 한 상태를 의미한다.  
주로 Over Commit 상태일 때 해당 현상이 동반된다.


## OOM(Out Of Memory) Killer 란?

Linux 의 메모리 부족으로 Out Of Memory가 발생 하였을 때, OOM Score 를 기준으로 점수가 높은 프로세스를 Kill 하여 메모리 확보를 하는 Linux Kernel 기능이다.  
OOM Killer 로 인한 프로세스 강제 종료 시 /var/log/ 내부의 시스템 로그에서 확인 할 수 있다.  

```bash
# oom_killer 로 인한 프로세스 종료 Log
$ cat /var/log/syslog | grep oom
 Mar  7 19:14:00 zabbix-node01 kernel: [1132818.054201] ib_log_writer invoked oom-killer: gfp_mask=0x100cca(GFP_HIGHUSER_MOVABLE), order=0, oom_score_adj=0
```

OOM Score 점수 산정 및 OOM Killer 구동의 직접적인 영향을 주는 요인을 알아보자.  

```
1. 프로세스에 할당 된 메모리 양 + 프로세스로 부터 fork() 된 자식 프로세스의 메모리 양 
2. 프로세스의 동작 중 인 시간
3. 프로세스가 하드웨어에 직접 접근하거나, root계정 (super user) 에 의해 실행 된 프로세스
4. 프로세스의 nice 값이 1 이상일 경우 Score 점수 2배 증가
5. /proc/[PID]/oom_score_adj 값 (유저가 직접 변경 가능)
6. /proc/[PID]/oom_adj 값
```

프로세스의 OOM Score를 확인하는 방법은 다음과 같다.

```bash
# oom_score 확인 방법
$ cat /proc/890081/oom_score
1048
# 890081은 PID 이며, 현재 OOM Score는 1048이다.
```

## oom_adj / oom_score_adj 에 대한 부연 설명

종료되면 안되는 주요 프로세스가 OOM Score 가 높아 OOM Killer에 의한 종료가 예상 될 때에는, oom_adj /oom_score_adj 값의 조절을 통하여, OOM Score 조절이 가능하다.  
oom_adj 의 범위는 -17 ~ 15 이며, 최저값인 -17로 지정 시 OOM Killer Disable과 같은 의미를 가진다.  
oom_score_adj의 범위는 -1000 ~ 1000 이며, 현재 OOM Score 에서  oom_score_adj 값을 가감한다.(oom_scoer_adj 의 -1000은 oom_adj 의 -17과 같다.)  
oom_adj / oom_score_adj 의 값을 변경 시 변경 한 값을 참조하여 oom_score의 재 산정이 이루어진다.
프로세스의 OOM Score 확인 및 oom_score_adj 를 통한 OOM Score 조절은 아래 예를 참고하자.

```bash
# 현재 oom_score 확인
$ cat /proc/890081/oom_score
1048
# 890081은 PID 이며, 현재 OOM Score는 1048이다.

# oom_score_adj 를 통한 OOM Score 조절
$ echo -1000 > /proc/890081/oom_score_adj

$ cat /proc/890081/oom_score_adj
-1000
# oom_score_adj 값에 가장 낮은 값인 -1000 으로 설정

# 설정 후 oom_score / oom_adj 확인
$ cat /proc/890081/oom_score
0
$ cat /proc/890081/oom_adj
-17
# oom_score_adj 설정 후 확인 결과, oom_adj 는 -17 (OOM Killer Disable)
# oom_score는 1048 -> 0 으로 감소한 것을 확인할 수 있다.
```

추가적으로, overcommit에 대한 설정을 할 수 있다.
over commit의 설정 방법은 다음과 같다.

```bash
# over commit 설정
$ cat /etc/sysctl.conf | grep overcommit_memory
vm.overcommit_memory = 1 # 0~2 의 설정값을 가진다.
# 0 = Heuristic overcommit. Default 설정 값이며, 휴리스틱하게(유동적이게) 커널 자체적으로 over commit 비율을 조정한다.
# 1 = 항상 over commit을 허용한다. OOM Killer가 동작하지 않으나, 프로세스의 메모리할당에 실패 할 수 있다.
# 2 = vm.overcommit_ratio 의 값을 참조하여 over commit 비율을 조정한다. 

# 사용 가능한 물리 메모리의 백분율
$ cat /etc/sysctl.conf | grep overcommit_ratio
vm.overcommit_ratio = 90
# 해당 설정은 vm.overcommit_memory가 2일때 의미를 가진다.
# 90%의 물리메모리 사용 + swap 메모리 사용을 초과하게 되면 OOM Killer 가 동작한다.

# 설정 적용
$ cat systemctl -w
```
그렇다면, 현재 commit 되고있는 메모리와 허용가능한 over commit 메모리양은 어떻게 확인할까?  
답은 /proc/meminfo 과 sysstat 패키지로 조회할 수 있다.
하기 예를 참고하자.

```bash
# commit 정보 확인
$ cat /proc/meminfo | grep Commit
CommitLimit:    30229156 kB 
# vm.overcommit_memory 의 설정이 2 일 때 vm.overcommit_ratio 의 설정을 기반으로 commit 가능 한 메모리 양
Committed_AS:   64267776 kB
# 현재 commit 되고 있는 메모리 양

# sysstat 패키지를 이용한 현재 commit 양 확인
$ apt install -y sysstat 
# sysstat 패키지 설치

# sar 명령어를 이용한 메모리 상태 조회
$ sar -r 1 
Linux 5.4.0-104-generic (zabbix-node02)         04/14/2022      _x86_64_        (16 CPU)

09:25:17 AM kbmemfree   kbavail kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
09:25:18 AM    208884   2446760  20804112     85.73    990436   1441416  64262056    196.79  19538260   3304352      1508
# %commit 부분이 현재 commit된 메모리 양을 백분율로 표기해주는 부분이다.
# 현재 100% 이상 commit 되고 있으므로, over commit 상태이다.
# sar 명령어는 시스템 자원(CPU, Memory, I/O) 사용량을 수집, 레포트 하는 명령어다.
# -r 옵션은 Memory 사용량을 조회하는 옵션이며, 뒤의 1은 1초 마다 수집한다는 의미이다.
```

## Linux Memory Commit / Memory Over Commit 이란?

Memory Commit 이란  
프로세스가 커널에게 메모리 공간을 요청 하여 가상 메모리 영역 주소를 할당 받은 상태

이해를 위해 아래 예를 참고 하자.  

```
A 프로세스가 커널에게 메모리 할당을 요청 하였다고 가정 했을 때, 
커널은 이에 해당하는 가상 메모리 주소를 A 프로세스에게 할당한다.  
결과적으로 A프로세스는 커널에게 메모리 영역을 할당 받았다고 생각하지만, 실제론 물리 메모리에 할당 받지 않은 상태다.  
이를 Memory Commit (메모리 커밋) 이라고 한다.  
```

프로세스 의 메모리 주소 할당을 바로 물리 메모리 주소로 할당 하면 생길 수 있는 문제점은 다음과 같다.

```
1. A 프로세스가 메모리를 할당받고 사용하지 않을 수 있다.
2. 이미 사용중인 메모리 주소값으로 접근 할 수 있는 위험성이 있다. 
 - 예로, 이미 커널이 사용 중 인 물리 메모리 주소가 반환될 경우 매우 위험
3. 메모리 단편화(Fragmentation)가 발생한다.
 - 메모리 단편화 : RAM 에서 메모리 공간이 작은 공간으로 나뉘어 사용가능한 메모리가 존재하지만 할당이 불가능한 상태
 - 메모리 단편화 발생 사유 및 해결방법에 대한 자세한 내용은 다음에 다루도록 한다.
```

Memory Over Commit이란  
말 그대로, 리눅스의 실제 물리 Memory보다 Over(초과) 하여 Commit 된 상태를 의미한다.  
결론적으로 over commit이 발생한다고 하더라도, 시스템에 문제가 생기진 않는다.  
하지만, over commit의 양이 지속적으로 증가한다면 oom killer로 중요 프로세스가 갑자기 종료되는 일이 없도록   
시스템의 설정과 프로세스가 요구하는 메모리를 잘 파악하고 메모리 누수가 없는지 점검이 필요하다.  




</div>