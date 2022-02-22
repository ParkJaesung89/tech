# Info

## Teacher

```
최국현
tang@linux.com
gochoi@redhat.com
```

## Main

https://github.com/tangt64/duststack-k8s-auto

https://github.com/tangt64/lab-sources-kubernetes

https://github.com/tangt64/community-lab-kubernetes-administrator-1

https://github.com/tangt64/diagram-kubernetes

https://github.com/tangt64/tangt64

## Openshift

https://github.com/tangt64/duststack-ocp-upi

## Etcd

https://github.com/tangt64/PoC-etcd-utility/settings

## ETC

`IAM`

https://www.keycloak.org/

https://www.keycloak.org/getting-started/getting-started-openshift

https://cloud.redhat.com/blog/adding-authentication-to-your-kubernetes-web-applications-with-keycloak

`CentOS 8 Stream`

```bash
https://ftp.kaist.ac.kr/CentOS/8-stream/isos/x86_64/

CentOS-Stream-8-x86_64-latest-boot.iso: Network Install (설치 전 외부 네트워크 연결을 통한 Repo 연동)
```

`Linux Container`

```bash
https://linuxcontainers.org/

LXD, LXC, LXCFS 관리 프로젝트
```

`skopeo`

https://github.com/containers/skopeo

`keycloak`

```
https://www.keycloak.org/
https://cloud.redhat.com/blog/adding-authentication-to-your-kubernetes-web-applications-with-keycloak
https://www.keycloak.org/getting-started/getting-started-openshift
```

`ovn-kubernetes`

```bash
https://github.com/ovn-org/ovn-kubernetes

## 기존 k8s 에서 ovs + [iptables 혹은 nftables] 를 사용하여 OS 에 영향을 받았다.
## 향후 iptables 혹은 nftables 가 아닌 ovn+ovs 조합으로 변경될 수도 있다.
```

`satellite`

`foreman`

https://theforeman.org/

`quay`

```
https://quay.io/
https://about.gitlab.com/devops-tools/quay-vs-gitlab/
```

`atom os`

https://atomdocs.io/#test-a-container

https://rook.io/

https://www.gluster.org/

# Info-Other

```
https://kubernetes.io/ko/docs/reference/kubectl/cheatsheet/
https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
https://kubernetes.io/ko/docs/concepts/scheduling-eviction/taint-and-toleration/
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#drain
https://kubernetes.io/ko/docs/concepts/policy/pod-security-policy/
https://kubernetes.io/ko/docs/concepts/storage/storage-classes/
https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/
https://github.com/tangt64/duststack-osp-auto
```

# Data

```bash
/home/archive/container_k8s/k8s_duststack_auto/

/home/archive/container_k8s/k8s_lab-sources-kubernetes/

/home/archive/container_k8s/k8s_diagram-kubernetes/
```

# Set_OS-VTDX_Check

윈도우 - task manager - Performance - CPU - Virtualisation: Enabled

시작 - 시스템 정보

```bash
Hyper-V - VM 모니터 모드 확장	예
Hyper-V - 두 번째 수준 주소 변환 확장	예
Hyper-V - 펌웨어에 가상화 사용	예
Hyper-V - Data Execution Protection	예
# VTDX 가 활성화 안되는 경우 하기 방법도 시도
## bcdedit /set hypervisorlaunchtype off
```

# Set_OS-HyperV_Set

제어판 - 프로그램 - Windows 기능 켜기/끄기 - Hyper V 체크

```bash
Virtualbox 의 Acceleration 을 위해 Hyper V 기능 활성화
```

# Set_Virtualbox-VM

## VM_Master

```bash
Name: master
Type: Linux
Version: Red Hat (64bit)
Spec: CPU: 2 Core / RAM: 4GB / Hard Disk: 10GB
Setting:
  System - Processor - 2 코어
  System - Processor - Enable PAE/NX
  System - Acceleration - Para - Hyper-V

  Network - Adapter 1 - Bridge
  Network - Adapter 2 - Bridge
```

## VM_Node-1

```bash
Name: node1
Type: Linux
Version: Red Hat (64bit)
Spec: CPU: 2 Core / RAM: 4GB / Hard Disk: 10GB
Setting:
  System - Processor - 2 코어
  System - Processor - Enable PAE/NX
  System - Acceleration - Para - Hyper-V

  Network - Adapter 1 - Bridge
  Network - Adapter 2 - Bridge
```

## VM_Node-2

```bash
Name: node2
Type: Linux
Version: Red Hat (64bit)
Spec: CPU: 2 Core / RAM: 4GB / Hard Disk: 10GB
Setting:
  System - Processor - 2 코어
  System - Processor - Enable PAE/NX
  System - Acceleration - Para - Hyper-V

  Network - Adapter 1 - Bridge
  Network - Adapter 2 - Bridge
```

# Set_Virtualbox-Guest_OS

CentOS 07 설치

```bash
CentOS 07 ISO 최초 설치화면 - Tab

vmlinuz initrd=initrd.img ... quiet net.ifnames=0
## Legacy 방식의 네트워크 디바이스명 설정 (eth)
# biosdevname=0
## Dell 장비에 적용

root / redhat

eth0: DEFROUTE=no
eth1: DEFROUTE=yes

sudo hostnamectl set-hostname master
sudo hostnamectl set-hostname node1
sudo hostnamectl set-hostname node2

sudo vi /etc/hosts

203.248.23.173 master
203.248.23.175 node1
203.248.23.177 node2

systemctl disable --now postfix
```

Package

```bash
# Master Node
sudo yum install -y centos-release-ansible-29

sudo yum install -y net-tools git ansible nano vim
```

# Set_Virtualbox-K8S_Deploy

## Process

```bash
# Master Node
git clone https://github.com/tangt64/duststack-k8s-auto.git

ssh-keygen -t rsa -N ''

ssh-copy-id master
ssh-copy-id node1
ssh-copy-id node2

ssh master
ssh node1
ssh node2
## fingerprint

cd duststack-k8s-auto/

vi inventory/kubernetes

[k8s_master]
203.248.23.173 hostname=master nodename=master

[k8s_node]
203.248.23.175 hostname=node1 nodename=node1
203.248.23.177 hostname=node2 nodename=node2

#[k8s_utility]
#192.168.122.200 hostname=utility nodename=utility

ansible all -m ping -i inventory/kubernetes

ansible-playbook playbooks/k8s-install.yaml -i inventory/kubernetes
```

## Test

```bash
kubectl run test-pod --image=nginx

kubectl get pod -o wide

kubectl run -it debugger --rm --image=wayles54/debugger-alpine -- /bin/sh

curl 192.168.166.132

delete pod --all -n default
```

# Set_Libvirt-K8S_Deploy

libvirtd, GuestOS, 그리고 K8S 를 한 번에 배포한다.

```bash
systemctl disable --now postfix

sudo yum install -y centos-release-ansible-29

sudo yum install -y net-tools git ansible python-lxml

ssh-keygen -t rsa -N ''

ssh-copy-id 127.0.0.1

git clone https://github.com/tangt64/duststack-k8s-auto.git

ansible-playbook -i inventory/classroom playbooks/lab-provisioning.yaml
```

# Set_K8S_Kubeadm-Intro

프로덕션에서는 아래와 같이 네트워크를 구분한다.

```
1: 외부망
2: 내부망
3: 스토리지 망
4. 관리 네트워크 망 (대규모 서비스, kubectl 도 많은 사용자가 수행 시, 부하가 꽤 크다)
5. 서비스 망
```

K8S 클러스터 외에 아래와 같은 Node 도 별도로 필요하다.

```
LB
NTP
Utility (RPM, Image, dhcpd 등)
Storage (GlusterFS, Ceph, NFS 등)
```

# Set_K8S_Kubeadm-Set

## Intro

K8S 클러스터를 구성한다.

## PreRequisite

DNS 서버가 꼭 필요하다. 없다면 /etc/hosts에서 A 레코드 매칭

## Init

`All Node`

```bash
sudo systemctl disable --now firewalld

swapoff -a
## 스왑 임시 해제
## 스왑 영구 해제는 /etc/fstab 주석처리

swapon -s
## 스왑상태 확인 가능
```

`Master Node`

```bash
kubeadm init

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get nodes

## --kubernetes-version string: Choose a specific Kubernetes version for the control plane. (default "stable-1")
## --image-repository string
```

`Worker Node`

```bash
kubeadm join 203.248.23.173:6443 --token w33a6y.isnh0vk1h6r2zsu9 \
        --discovery-token-ca-cert-hash sha256:c99319442b5054b4c1b0f20b2e36295178f7331bb2f51446bb94453bf9bae84a
```

## Token

K8S init 완료 후, token 재확인이 불가하다. 신규 token 을 추가할 수 있다.

```bash
kubeadm token create --print-join-command

kubeadm token list
```

# Set_K8S-Kubeadm_Teardown

## Intro

K8S 를 서비스를 삭제한다.

## Process

`All Node`

```bash
sudo systemctl stop kubelet
kubeadm reset -f
```

`Master`

```bash
sudo rm -rf ~/.kube
sudo rm -rf /root/.kube
sudo rm -rf /var/lib/etcd
```

# Set_K8S-Upgrade

## Intro

K8S 버전을 업데이트 한다.

## Process

```bash
sudo yum install -y kubeadm-1.22.2
sudo yum install -y kubelet-1.22.2

kubeadm upgrade plan

kubeadm upgrade apply v1.22.2

systemctl restart kubelet

kubectl get nodes

kubeadm upgrade node
## 워커노드 업그레이드
```

# Set_K8S-Kubectl

```bash
sudo yum install -y bash-completion

kubectl completion bash >> ~/.bash_profile
source /usr/share/bash-completion/bash_completion

source ~/.bash_profile

complete -r -p
```

# Set_Editor-Vim

yaml 파일에 대해 indentation 을 기본값이 아닌 2칸으로 수정한다.

```bash
sudo yum install -y vim

vi ~/.vimrc

syntax on
au! BufNewFile,BufReadPost *.u{yaml,yml} set filetype=yaml foldmethod=indent
autocmd FileType yaml setlocal ts=2 sts=2 sw=2 expandtab

vi ~/.bashrc

export EDITOR=vim
alias vi=vim

. ~/.bashrc
```

# OS-Package_Source_Binary

`소스 패키지`

```bash
소스 패키지는 보통 코드가 압축되어 있다. 보통 사용되는 패키지 압축 형식은 "tar.gz" 이며, 소스 패키지를 Tarball이라고도 한다.

Tarball 은 Linux 시스템 용 패키징 도구로, 소스 패키지를 패키징 및 압축 할 수 있으며, 결과로 압축 된 압축 파일을 Tarball 파일이라고도 한다.

소스 패키지는 소프트웨어의 공식 웹 사이트에서 다운로드해야하며 보통 아래 내용이 포함되어 있다.
(소스 코드 파일 / 구성 및 탐지 절차 / 소프트웨어 설치 지침 및 소프트웨어 지침)
```

`바이너리 패키지`

바이너리 패키지는 소스 패키지가 성공적으로 컴파일 된 후 생성된 패키지이다.

현재 두 가지 주요 주류 바이너리 패키지 관리 시스템이 있다.
(RPM 패키지 관리 시스템 / DPKG 패키지 관리 시스템)

# OS-Package_Repository

```bash
EPEL: 레드햇에서 관리

rpmfusion: 커뮤니티에서 관리
```

# OS-Security_nftables

```bash
iptables: CentOS/RHEL 7까지만 지원 -> nftables로 지원

nftables:
  iptables는 자동화가 까다로움
  nftables는 자동화 및 좀 더 유지보수가 쉽고 성능이 기존 iptables보다 월등히 개선됨
  connection_tracker.ko 메모리 및 CPU 사용량이 높음
  iptables2nftables 현재는 호환성을 유지하고 있음
```

# OS-Network

## bridge link

Layer 1

## bridge fdb

The FDB (forwarding database) table is used by a Layer 2 device (switch/bridge) to store the MAC addresses that have been learned and which ports that MAC address was learned on. The MAC addresses are learned through transparent bridging on switches and dedicated bridges.

# OS-Namespace

## lsns

```bash
kubectl run test-pod --image=ubuntu --command -- sleep 1000

kubectl exec test-pod -- ps
```

```bash
# 네임스페이스 영역 조회 1
lsns
## NS: namespace ID
## TYPE: 자원 타입 (mnt: 파일시스템 / utf: 시스템 시간 / ipc: 프로세스 커뮤니케이션 영역, OS <---> APP+LIB)
### ipc mnt net pid user uts ;; namespace에서 사용하는 추상화 자원
## NPROCS: 컨테이너 내부 PID
## PID: Host 맵핑 PID
## USER: 컨테이너 실행 유저
## COMMAND

# 네임스페이스 영역 조회 2
ps aux | grep sleep
## nginx: master process nginx -g daemon off;

ls -al /proc/<pid>/ns/
```

## nsenter

네임스페이스 영역 접근

# OS-CGroup

```bash
mount
# cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuacct,cpu)
# cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
# cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
# cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)
# cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
# cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_prio,net_cls)
# cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
# cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
# cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)
# cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
```

## systemd-cgls

systemd-cgls recursively shows the contents of the selected Linux control group hierarchy in a tree.

```bash
systemd-cgls
```

## systemd-cgtop

systemd-cgtop shows the top control groups of the local Linux control group hierarchy, ordered by their CPU, memory, or disk I/O load.

```bash
systemd-cgtop
```

# Hypervisor-Intro

링 구조로 구동 된다.

가상머신은 이 모든 부분을 소프트웨어로 재구성

# Container-Intro

소프트웨어 파티셔닝: 컨테이너는 시스템 콜 및 라이브러리 콜을 호스트 컴퓨터와 공유 (namespace + cgroup)

O/S의 별도 커널 정보는 없다. lib/glibc와 애플리케이션 라이브러리 및 데이터

```
                    LOW                                                                    HIGH

                 .----------------------------------------- podman
                /                                                  ------------ docker
               /                                                                       ----------- kubernetes
              /           runtime       .--- pod
                           -----       /
conmon ---  RUNC    ---    cri-o       ---
                                       \
                                        '--- container

                           containerd
```

컨테이너는 하이퍼바이저 타입 1 의 구동방식과 유사하다.

```
         .----> Virtualization PaaS
        /        - hardware high
PaaS --->        - booting(BIOS)
        \                                       .--- 메인프레임(AIX) ---> Partition(하드웨어 분할)
         `----> Container PaaS  <--------------'
                 - hardware low
                 - none-booting(rootless)

                 cri-o/runc**
                 ----------
                 \
                  \
                   '---> runc(running in kernel)
                          /
                         /
                        /
                       v
                Type-1: Baremetal system
                (kvm) ---> kernel inside
```

하나의 OS 에 여러 Container Engine 구동이 가능하다.

# Container-Term

Open Source Container Initiative: industry standards around container formats and runtimes.

Docker-shim: Kubernetes v1.24 에서 제거 예정

```bash
https://kubernetes.io/blog/2020/12/02/dockershim-faq/
```

Containerd: containerd is available as a daemon for Linux and Windows. It manages the complete container lifecycle of its host system, from image transfer and storage to container execution and supervision to low-level storage to network attachments and beyond.

```bash
https://containerd.io/
```

# Container_Runtime-Intro

OCI 준수 컨테이너 런타임: Docker, CRI-O, LXC, LXD 등

Runtime 은 실제 동작하는 로직에 관여하지는 않는다. (스펙==API 역할을 하지 않는다.)

도커는 자체적으로 여러 구성요소가 있지만, OCI 의 경우 컴포넌트 별로 기능이 구분된다.

# Container_Runtime-Diff

## Docker-CRIO

`Docker vs CRIO`

contained 환경을 지원하는 CRI-O, docker 는 OCI 를 지원하지만, 약간의 차이가 있다.

```bash
kubelet -> docker-shim -> docker -> containerd -> containers
## K8S 에서 docker 호환성을 위해 임시로 docker-shim 을 개발하였지만, 유지보수에 어려움이 있어서 향후 제거 예정

kubelet -> cri-containerd -> containerd -> containers
```

## Docker_CRIO-LXC_LXD

`Docker/CRIO vs LXC/LXD`

Docker, CRIO: rootless container runtime

```bash
장점: bootup fast
단점: super ultra limited access to the hardware device
```

LXC/LXD: rootful container runtime

```bash
장점: bootup slow
단점: limited access to the hardware device, much better than Docker/CRIO
```

# Container_Runtime-Structure

RunC: 라이프사이클 관리자

# Container_CRIO-Intro

/etc/crio/crio.conf 에서 kubectl exec 을 가능하게 하는 보안 관련 설정을 할 수 있다.

# Container_CRIO-Command

```bash
crictl pods
## pod 출력

crictl pods --label run=nginx

crictl images

crictl ps -a
## 종료된 컨테이너도 확인

crictl exec 94e19913634d1 ls
## 컨테이너 내부 명령어 실행

crictl logs 94e19913634d1

crictl inspect 94e19913634d1
```

# Container_Containerd-Intro

우분투 계열에서 주로 containerd 를 사용하려 하지만 프로덕션에서는 CRI-O 를 많이 사용한다.

containerd 도 CRI-O 와 같이 컨테이너 경량화를 목적으로 한다.

# Container_Containerd-Set

```bash
sudo yum install -y containerd

systemctl enable --now containerd

systemctl status containerd
```

# Container_Docker-Intro

https://www.docker.com/increase-rate-limit

# K8S_Kubeproxy-Intro

```
iptables-save
        -------------
        \
         \
          '--- 컨테이너에서 사용하는 네트워크 정보를 구성 및 재구성 해주는 역할
               S/D NAT기반으로 구성이 되어 있음
```

```bash
ps aux | grep kube-proxy
```

# K8S_CoreDNS-Intro

https://coredns.io/

https://kubernetes.io/ko/docs/concepts/services-networking/dns-pod-service/

```
CoreDNS는 bind와 같이 일반적인 DNS 레코드 서비스 대몬.
Go언어로 작성이 되었있음
크기가 bind보다 훨씬 작음(상대적으로 버그가 없다)
설정이 매우 간편함
```

# K8S_CoreDNS-Config

```bash
kubectl get service -A
kubectl describe svc/kube-dns --namespace kube-system
```

# K8S_Etcd-Intro

https://etcd.io/

# K8S_Kubelet-Intro

kubelet: API 통신을 위한 일종의 proxy서버

```bash
sudo systemctl status kubelet
## 과거: 모든 서비스가 host 에서 동작
## 현재: kubelet 제외하고 나머지는 컨테이너 기반으로 동작 (=kubelet 만 systemd 로 구동)
```

# K8S_Kubelet-Check

kubelet 확인하기

```bash
systemctl -t service
## systemctl -t socket -> 소켓 나열

systemctl status kubelet
## 유일하게 O/S에 설치 되어있는 쿠버네티스 서비스

ss -antp | grep LISTEN
## netstat -ntlpu 와 유사

ss -antp | grep ESTAB
## netstat -anp 와 유사
```

# K8S_Component-Command

K8S 전체 상태 확인

```bash
kubectl get componentstatuses
# 1.20.2 버전에서 kube-scheduler 가 기본 Unhealthy 한 이슈가 있다.
## cd /etc/kubernetes/manifests/kube-controller-manager.yaml
## cd /etc/kubernetes/manifests/kube-scheduler.yaml
## #- --port=0
## systemctl restart kubelet
## https://github.com/kubernetes/kubernetes/issues/93342

kubectl get nodes
```

# K8S_Namespace-Intro

namespace == project

```bash
namespace: back, 시스템 영역에서 보통 이렇게 부름
project: front, 사용자 영역에서 보통 이렇게 부름
```

# K8S_Namespace-Command

```bash
kubectl get namespaces

kubectl create namespace first-namespace
```

# K8S_Namespace-Question

쿠버네티스에서 네임스페이스를 "basic”라는 이름으로 생성한다.

```bash
kubectl create ns basic
```

생성된 "basic” 네임스페이스를 기본 네임스페이스로 설정한다.

```bash
kubectl config set-context --current --namespace=basic
```

올바르게 생성이 되면 kubectl get pods 그리고 kubectl config current-context 명령어로 올바르게 전환이 되었는지 확인한다.

```bash
kubectl config get-contexts
```

# K8S_Kubectl-Config

```bash
kubectl config get-contexts

kubectl config set-context first-namespace
## 기본 네임스페이스를 first-namespace 로 지정 (에러 발생)

kubectl config get-contexts
## 위의 명령어 실행 시, 기존 cluster 와 유저에 namespace 를 추가하는 것이 아닌, 임의로 context 를 생성한다.
## CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
##           first-namespace
## *         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin   basic

kubectl config set-context --current --namespace=first-namespace
## 기본 네임스페이스를 first-namespace 로 지정 (정상)

kubectl config get-contexts
```

# K8S_Kubectl-Explain

```bash
kubectl explain pod
```

# K8S_Kubectl-Get

```bash
kubectl get pods -w

kubectl get pods -o -A
```

# K8S_Kubectl-Get-Question

pod, deploymnt, replicaset, replica 에서 구성이 되어있는 자원 목록을 확인한다.
해당 내용을 각각 오브젝트 이름으로 .txt 파일을 만들어서 내용을 저장한다.

```bash
kubectl get pods -o wide

kubectl get deployments.apps -o wide

kubectl get replicasets.apps -o wide

kubectl get replicationcontrollers -o wide
```

-o yaml 으로 내용으로 전환 후 저장한다.

```bash
kubectl get pods -o yaml

kubectl get deployments.apps -o yaml

kubectl get replicasets.apps -o yaml

kubectl get replicationcontrollers -o yaml
```

# K8S_Kubectl-Describe

```bash
kubectl describe deployments.apps/my-httpd

kubectl describe -f nginx-demo-create.yaml
```

# K8S_Kubectl-Describe-Question

```bash
kubectl describe service kubernetes -n default
```

# K8S_Kubectl-Create

```bash
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
EOF

kubectl get deployments.apps
kubectl get replicasets.apps
```

# K8S_Kubectl-Copy-Question

```bash
기존에 구성이 되어있는 my-httpd 에 index.html 파일을 복사한다.
해당 index.html 파일은 “Hello Apache”라는 문자열을 가지고 있어야 한다.
```

```bash
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-httpd
spec:
  selector:
    matchLabels:
      run: my-httpd
  replicas: 1
  template:
    metadata:
      labels:
        run: my-httpd
    spec:
      containers:
      - name: my-httpd
        image: httpd
        ports:
        - containerPort: 80
EOF

cat << EOF | tee index.html
helllllo
EOF

kubectl cp index.html my-httpd-6796cbbc4c-h5cj4:/usr/local/apache2/htdocs/
## pod name

kubectl port-forward my-httpd-6796cbbc4c-h5cj4 80:80
## pod name

curl 127.0.0.1
```

# K8S_Kubectl-Exec

```
kubectl exec   --
               -: 표준 출력
               --: 표준 입출력
bash
 \
  \
   '- /usr/bin/httpd  <------> [container image]  <---  args['/bin/ls','-al']
   |                                sh, bash
   |                                                        --
    '. /bin/ls -al    <------> [container image] ---> kubectl redirect ---> [a.txt]
                      stdin/out     runtime
```

```bash
kubectl run curl --image=radial/busyboxplus:curl -i --tty

kubectl run centos --image=centos -i --tty
## exec 에서는 접근 불가 (런타임에서 접근 방지, crictl 도 exec -it 불가)
## 컨테이너 런타임 보안 설정 때문

kubectl run ubuntu --image=ubuntu -i --tty

kubectl attach curl -c curl -it
## attach 는 접근 가능
```

# K8S_Kubectl-Exec-Question

```
앞에서 만든 my-httpd 의 index.html 파일을 확인 및 내부 내용을 확인한다
```

```bash
kubectl exec my-httpd-6796cbbc4c-h5cj4 -- ls
```

# K8S_Kubectl-Attach

```bash
kubectl run curl --image=radial/busyboxplus:curl --command -- sleep 1000

kubectl attach curl -c curl -it
## attach 는 접근 가능하지만 sleep 상태이기 때문에 조치 불가
## 정상적으로 콘솔에 접근하기 위해서는 run 시점부터 -it 옵션이 필요하다.
```

# K8S_Kubectl-Edit

```bash
kubectl create deployment my-httpd --image=httpd

kubectl edit deployment my-httpd
## vi editor

EDITOR=nano kubectl edit deployment my-httpd

echo "KUBE_EDITOR=nano" >> ~/.bash_profile
```

# K8S_Kubectl-Edit-Question

```
deployment 에 구성된 httpd 및 nginx 의 replica 의 개수를 10 개로 변경 후 상태를 확인
```

```bash
kubectl create deployment my-httpd --image=httpd

EDITOR=nano kubectl edit deployment my-httpd
## replicas: 10
```

# K8S_Kubectl-Delete

```bash
kubectl delete pod -l run=my-nginx

kubectl delete service my-nginx

kubectl delete pods --all

kubectl delete all --all

kubectl get pods
```

# K8S_Kubectl-CP-Intro

```bash
kubectl cp -> SHA, Hash 암호화 -> /var/lib/containers 파일 write

/var/lib/containers/storage/overlay/backingFsBlockDev 을 통해 Layer 에 변경사항 기재

Container Image 는 오버레이 파일 시스템을 사용한다. (OVFS, OFS, Overlay FileSystem)

qcow: Copy On Write
```

# K8S_Kubectl-Diff

```bash
cat << EOF | tee test.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
EOF

kubectl create -f test.yml

kubectl edit -f test.yml
## replicas: 2

kubectl diff -f test.yml
```

# K8S_Kubectl-Diff-Question

my-httpd 의 replicas 개수를 100 개로 변경 후 시스템에 등록된 my-httpd 변경사항을 확인하여 올바르게 되었는지 확인한다.

```bash
cat << EOF | tee test.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
EOF

kubectl create -f test.yml

kubectl edit -f test.yml
## replicas: 100

kubectl diff -f test.yml
```

# K8S_Kubectl-Debug

container shell 로 접근이 가능한 명령어로 보인다. 보안 상의 이유로 컨테이너에 접근하는 기능은 없어지는 추세이므로 참조만 하자.

```bash
kubectl debug -it debugger --image=wayles54/debugger-alpine --target=debugger
## Targeting container "debugger". If you don't see processes from this container it may be because the container runtime doesn't support this feature.
## Shell 접근 없이 Pod 가 생성된다.
```

# K8S_Kubectl-Log

```bash
kubectl run nginx --image=nginx

kubectl logs -f nginx
## tail -f 와 유사함
```

# K8S_Kubectl-Apply-Question

```
my-nginx 에 레이블을 system=window 라고 추가한다.
replica 개수를 50 개로 변경한다.
시스템에 어떠한 변경이 발생하는지 확인한다.
```

```bash
cat << EOF | tee test.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
    system: window
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
EOF

kubectl create -f test.yml

vi test.yml
## replicas: 2

kubectl apply -f test.yml

vi test.yml
## replicas: 3

kubectl replace -f test.yml
```

# K8S_Kubectl-Replace

## 01

apply 와 replace 의 경우 상황따라 다르므로 replace 와 apply 가 결과는 유사하지만 내부 과정이 미세하게 다르다는 정도로 이해하자.

https://kubernetes.io/ko/docs/tasks/manage-kubernetes-objects/_print/

`apply`

```bash
cat << EOF | tee test.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
EOF

kubectl create -f test.yml

vi test.yml
## replicas: 2

kubectl apply -f test.yml
```

`replace`

```bash
cat << EOF | tee test.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
EOF

kubectl create -f test.yml

vi test.yml
## replicas: 2

kubectl replace -f test.yml
```

## 02

apply 는 기존 오브젝트의 수정된 Field 만 Update 한다면, replace 는 작성된 yaml 파일 기준으로 재생성한다.

`apply`

```bash
cat << EOF | tee test.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
EOF

kubectl apply -f test.yml

kubectl edit deployments.apps test
## svc: prod 라벨 추가

vi test.yml
## type: web 라벨 추가

kubectl apply -f test.yml

kubectl describe deployments.apps test
## Labels:         app=test
##                 svc=prod
##                 type=web
## 수정된 라벨 모두 내역 존재
```

`replace`

```bash
cat << EOF | tee test.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
  name: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test
  template:
    metadata:
      labels:
        app: test
    spec:
      containers:
      - image: nginx
        name: nginx
EOF

kubectl create -f test.yml

kubectl edit deployments.apps test
## svc: prod 라벨 추가

vi test.yml
## type: web 라벨 추가

kubectl replace -f test.yml

kubectl describe deployments.apps test

## Labels:           app=test
##                   type=web
## kubectl edit 로 추가한 내용이 사라진다.
```

# K8S_Pod-Pause

Pod == Pause Container

Pod 는 컨테이너 및 다른 객체들을 격리 및 제어하는 역할 (pause.go라는 프로그램이 POD관리 및 구성)

`pause container 확인`

```bash
ps -ef | grep pause
## pause 컨테이너 확인
```

`pause container namespace`

```bash
ps -ef | grep pause
lsns | grep <PAUSE_PID>
# 4026532118 mnt        1 17993 root    /pause
# 4026532182 uts        2 17993 root    /pause
# 4026532183 ipc        2 17993 root    /pause
# 4026532184 pid        1 17993 root    /pause
```

`kubelet pause container 옵션`

```bash
ps aux | grep kubelet | grep pod
## --pod-infra-container-image=k8s.gcr.io/pause:3.5

systemctl cat kubelet --no-pager

vi /etc/sysconfig/kubelet

vi /var/lib/kubelet/config.yaml
```

# K8S_Pod-Pod_Container

Pod 와 Container 를 구분하여 출력해보자.

`Docker`

```bash
docker container ls
## pod, container 구분없이 모두 출력
```

`Cri-O`

```bash
crictl pods
## Pod 목록 출력

crictl ps
## Container 목록 출력
```

`PS`

```bash
ps aux | grep -i "_pod_"
## _pod_ 가 포함된 프로세스는 pod 라고 보면 된다.
```

# K8S_Pod-Check

```bash
kubectl get pods --all-namespaces
## 전체 네임스페이스 Pod 조회

kubectl get pods --all-namespaces --show-labels --sortby=.metadata.name

kubectl describe pod/nginx-ingress-rshwt --namespace=nginx-ingress
```

# K8S_Pod-Create

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: httpd
  labels:
    name: test
    version: rev1
spec:
  containers:
  - image: centos/httpd
    name: httpd
    ports:
    - containerPort: 80
      name: http
      protocol: TCP
EOF

kubectl get pods -w

kubectl describe pod/httpd

kubectl get pods httpd -o jsonpath --template {.status.podIP}

kubectl logs pods/httpd

cat << EOF | tee index.html
hello
EOF

kubectl cp --container httpd index.html httpd:/var/www/html/index.html

kubectl exec httpd -- cat /var/www/html/index.html

kubectl port-forward httpd 80:80

curl 127.0.0.1
```

# K8S_Pod-Create-Question

```bash
nginx 이미지 사용

레이블 version: prod 적용
```

```bash
sudo yum install -y podman

podman search nginx
## docker.io/library/nginx

podman pull docker.io/library/nginx

cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
    version: prod
spec:
  containers:
  - image: docker.io/library/nginx:latest
    name: nginx
    ports:
    - containerPort: 80
      name: http
      protocol: TCP
EOF

cat << EOF | tee index.html
hello
EOF

kubectl cp index.html nginx:/usr/share/nginx/html

kubectl port-forward nginx 80:80

curl 127.0.0.1
```

# K8S_Replicaset-Create

```
https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicaset/
https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicationcontroller/
https://cloud.redhat.com/blog/kubernetes-replicas-appreciated-workhorses
```

# K8S_Deployment-Create

```bash
cat << EOF | tee deployment.yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
EOF

kubectl create -f deployment.yml

kubectl get pod -o wide --show-labels
```

# K8S_Deployment-Create-Question

```
• 기존에 작성하였던 nginx 의 이미지를 apache 이미지로 변경한다.
• port 번호는 기존 포트번호 80/TCP 로 그대로 사용한다.
• 복제자 개수는 3 개에서 10 개로 변경한다.
• 레이블의 정보에 nginx 라는 값을 전부 httpd 로 변경한다.
```

```bash
podman search httpd
## docker.io/library/httpd

cat << EOF | tee deployment.yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  labels:
    app: httpd
spec:
  replicas: 10
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
        - name: httpd
          image: docker.io/library/httpd
          ports:
            - containerPort: 80
EOF

kubectl create -f deployment.yml

kubectl get pod -o wide --show-labels

kubectl get pod -o wide --show-labels | grep httpd
```

```
1. 기존에 사용하였던 yaml 파일을 사용해서 다음처럼 서비스를 구성하세요.
2. vsftpd 이미지를 사용해서 yaml 파일을 작성합니다.
3. 이미지 파일은 아래의 주소에서 받기가 가능합니다.
4. https://hub.docker.com/r/fauria/vsftpd/
5. 받은 이미지의 repllica 개수는 1 개로 합니다.
```

```bash
podman search vsftpd
## docker.io/library/httpd

cat << EOF | tee deployment.yml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vsftpd
  labels:
    app: vsftpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vsftpd
  template:
    metadata:
      labels:
        app: vsftpd
    spec:
      containers:
        - name: vsftpd
          image: docker.io/fauria/vsftpd
          ports:
            - containerPort: 21
EOF

kubectl create -f deployment.yml

kubectl get deployments.apps

kubectl get pod -o wide --show-labels
```

# K8S_Deployment-Set

```bash
kubectl create deployment my-httpd --image=httpd

kubectl describe deployment my-httpd

kubectl set image deployment/my-httpd httpd=httpd:alpine
## container name

kubectl describe deployment/my-httpd
```

# K8S_Deployment-Set-Question

my-nginx 의 버전을 mainline-alpine 으로 변경한다

```bash
kubectl create deployment my-nginx --image=nginx

kubectl set image deployment/my-nginx nginx=docker.io/library/nginx:mainline-alpine

kubectl describe deployment/my-nginx
```

# K8S_Deployment-RollingUpdate

```bash
cat << EOF | tee test.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
EOF

kubectl apply -f test.yml

kubectl rollout status deployment/nginx-deployment
## 배포 상태 확인

kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1

kubectl rollout status deployment/nginx-deployment
## 배포 상태 확인

kubectl rollout history deployment.v1.apps/nginx-deployment
## 배포 내역 확인

kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2
## 배포 내역 세부 확인

kubectl rollout undo deployment.v1.apps/nginx-deployment
## 이전 revision 롤백. 즉, revision=1
```

# K8S_Deployment-Scale

```bash
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10

kubectl scale deployment.v1.apps/nginx-deployment --replicas=2

kubectl get pod
```

# K8S_Service-Intro

POD Network: POD 에서 사용하는 네트워크. 일반적으로 POD 간 통신을 위한 네트워크

Service: 서비스는 POD에서 외부로 구성한 애플리케이션을 노출이 필요한 경우 사용하는 네트워크

```
{ 클러스터, ClusterIP, Kubernetes(default) }
{ 외부 도메인, ExternalName, 말 그대로 도메인 서비스, bind, coredns }
{ 외부 아이피, ExternalIP, 아이피 주소 기반으로 접근(v) }
{ 노드포트, NodePort, 포트번호 기반으로 접근(v) }
```

서비스가 생성된 경우 트래픽 흐름

```
iptables-save | grep <service_ip>
iptables-save | grep <pod_ip>

                                 L3            L2
                           { conntrack }    { bridge }
                              DNAT,MASQ
                         .--- iptables --- br_netfilter --- SVC
                        /
APP ---  container --- IP --- SEP
                        \
                         \
                          \
                           \          .--- RR ---.
                            \        /            \       External
APP ---  container --- namespace --- IP --- Endpoint --- ClusterIP(x)  ---  USER
                        veth@if                          LoadBalance

userspace --- DRV --- kernelspace
     \                   /
      '---   irq     ---'

veth: virtual tap device @ namespace dev ifX
         userspace     --->    kernelspace

kubectl describe svc kubernetes -n default

     --------------- POD ---------------	          				           .--------------- SVC ---------------.
                                                                      /                                     \
                                                                     /                                       \
       .--- namespace network  ---.                       .---  iptables   ---.                               \
      /                            \                     /                     \                               \
	APP ---- container     --->      POD   ---> IP --- veth ---  internal --- veth ---     SEP    ---        SVC     ---
	                           net: 10.244.               \        [router]       /   [Service Endpoint]    [service]
     \                              /                    \                     /
      '----------------------------'                      '---    bridge   ---'
```

```bash
iptables-save | grep <pod_ip>
## -A KUBE-SEP-DFKXVFOIF4YUHUTP -s 172.16.104.39/32 -m comment --comment "basic/my-nginx" -j KUBE-MARK-MASQ
## -A KUBE-SEP-DFKXVFOIF4YUHUTP -p tcp -m comment --comment "basic/my-nginx" -m tcp -j DNAT --to-destination 172.16.104.39:80
## KUBE-SEP-DFKXVFOIF4YUHUTP / KUBE-MARK-MASQ: 체인 이름
## 전체 노드에서 상기 정책 모두 확인 가능
## Service 생성마다 각각 정책을 확인할 수 있다.

iptables -nvL -t nat | grep -A10 -i KUBE-SEP-QD5DRZ5NRMQ5KSHV

Pod Outbound: Chain KUBE-SEP-6JKNXMU5GG5KGGHN -> Chain KUBE-MARK-MASQ
Pod Inbound: Chain KUBE-SEP-6JKNXMU5GG5KGGHN
```

# K8S_Service-Expose

## Imperative

```bash
cat << EOF | kubectl create -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
  labels:
    app: httpd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
        - name: httpd
          image: docker.io/library/httpd
          ports:
            - containerPort: 80
EOF

kubectl expose deployment/httpd

kubectl get services

kubectl run -it debugger --rm --image=wayles54/debugger-alpine -- /bin/sh
```

## Declarative

```bash
cat << EOF | kubectl create -f -
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
  labels:
    run: httpd
spec:
  ports:
    - port: 80
      protocol: TCP
  selector:
    app: httpd
EOF

kubectl get services

kubectl run -it debugger --rm --image=wayles54/debugger-alpine -- /bin/sh
```

# K8S_Service-Expose-Question

my-ngninx-second 라는 이름으로 80/TCP, my-ngnix 서비스를 ClusterIP 로 구성한다.

```bash
kubectl create deployment my-nginx --image=nginx

kubectl expose deployment my-nginx --port=80

kubectl get service

kubectl run -it debugger --rm --image=wayles54/debugger-alpine -- /bin/sh

curl 10.104.143.166
```

# K8S_Service-ExternalName

```bash
cat << EOF | tee svc_external.yml
apiVersion: v1
kind: Service
metadata:
  name: myapp-extname
spec:
  type: ExternalName
  externalName: www.google.com

EOF

kubectl apply -f svc_external.yml

kubectl run -it debugger --rm --image=wayles54/debugger-alpine -- /bin/sh

curl myapp-extname.svc.basic.cluster.local

curl -v myapp-extname
```

# K8S_Label-Create

```bash
kubectl run my-pod --image=nginx

kubectl label pods my-pod type=test

kubectl get pod --show-labels

kubectl label pods my-pod run=test --overwrite
## 기존 Key 값 수정
```

# K8S_Label-Create-Question

my-httpd 에 type=product 라는 레이블을 추가

```bash
kubectl run my-httpd --image=httpd

kubectl label pods my-httpd type=product
## Label 추가

kubectl describe pod my-httpd
```

# K8S_RBAC-Create

## Certificate

```bash
kubectl config get-contexts

sudo useradd satellite
sudo useradd openshift

cd ~satellite/

openssl genrsa -out satellite.key 2048

openssl req -new -key satellite.key -out satellite.csr -subj "/CN=satellite"
## 해당 명령어도 가능은 하다.

openssl req -new -key satellite.key -out satellite.csr -subj "/CN=satellite/O=$(groups)"
## 정석

openssl x509 -req -in satellite.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out satellite.crt -days 500

mkdir .certs
mv satellite.* .certs/

kubectl config set-credentials satellite --client-certificate=/home/satellite/.certs/satellite.crt --client-key=/home/satellite/.certs/satellite.key
## .kube/config user 항목 추가

kubectl config set-context satellite-context-1 --cluster=kubernetes --user=satellite
## .kube/config context 항목 추가

kubectl config set-context satellite-context-2 --cluster=kubernetes --user=satellite --namespace=satellite

kubectl config set-context satellite-context-3 --cluster=kubernetes --user=satellite --namespace=project-httpd-dev

kubectl config set-context satellite-context-4 --cluster=kubernetes --user=satellite --namespace=project-httpd-prod

kubectl create ns satellite
kubectl create ns project-httpd-dev
```

`참조: 향후 리눅스 유저 shell 을 kubectl 로 지정하여 k8s 사용자가 kubectl 만 사용가능하도록 커스터마이징도 가능`

```bash
chsh /usr/bin/kubectl

ssh satellite@1.1.1.1 get pods
```

## Rolebinding

```bash
kubectl config get-contexts

kubectl config use-context satellite-context-1
kubectl config get-contexts

kubectl get pod
## Error from server (Forbidden): pods is forbidden: User "satellite" cannot list resource "pods" in API group "" in the namespace "default"
## 현재 satellite 유저의 권한이 없다.

kubectl config use-context kubernetes-admin@kubernetes

강사님은 Role 과 Rolebinding 의 경우 namespace 를 명시하지 않아도(=default 에 정의) 괜찮다고 했지만, default 네임스페이스에 정의된 Role 과 Rolebinding 은 다른 네임스페이스에 권한 부여가 불가하다.

cat << EOF | tee rbac_01.yml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: normal-user-satellite
  namespace: satellite
rules:
  - apiGroups: ["*"]
    resources: ["pods"]
    verbs: ["get", "list"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: test-rolebinding-1
  namespace: satellite
subjects:
  - kind: User
    name: satellite
    apiGroup: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: normal-user-satellite

---

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: normal-user-project-httpd-dev
  namespace: project-httpd-dev
rules:
  - apiGroups: ["*"]
    resources: ["pods"]
    verbs: ["get", "list"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: test-rolebinding-2
  namespace: project-httpd-dev
subjects:
  - kind: User
    name: satellite
    apiGroup: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: normal-user-project-httpd-dev

EOF

kubectl apply -f rbac_01.yml

kubectl api-versions

kubectl config get-contexts

kubectl config use-context satellite-context-2
kubectl config get-contexts

kubectl get pods
## 조회 성공

kubectl get nodes
## Error from server (Forbidden): nodes is forbidden: User "satellite" cannot list resource "nodes" in API group "" at the cluster scope
## nodes 는 resources 에 명시가 안되었기 때문에 권한 에러 발생

kubectl config use-context kubernetes-admin@kubernetes
kubectl config get-contexts
```

## Cluster Rolebinding

roleRef 의 view, edit 와 같은 Keyword 의 경우 사전 정의된 Role 이므로 ClusterRole 에서만 사용 가능하다. (=Role 에서는 사용 불가)

```bash
kubectl config use-context kubernetes-admin@kubernetes
kubectl config get-contexts

kubectl create ns project-httpd-dev
kubectl create ns project-httpd-prod

cat << EOF | tee rbac_02.yml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: satellite-view
  namespace: project-httpd-dev
subjects:
- kind: User
  name: satellite
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: satellite-view
  namespace: project-httpd-prod
subjects:
- kind: User
  name: satellite
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: satellite-edit
  namespace: project-httpd-prod
subjects:
- kind: User
  name: satellite
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: edit
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f rbac_02.yml

kubectl config get-contexts

kubectl config use-context satellite-context-3
kubectl config get-contexts

kubectl auth can-i list pods
kubectl auth can-i create pods
kubectl auth can-i delete pods
kubectl auth can-i update pods
## 유저 satellite 의 네임스페이스 project-httpd-dev 에 대한 권한은 get 이므로 list pods 만 가능하다.

kubectl config use-context satellite-context-4
kubectl config get-contexts

kubectl auth can-i list pods
kubectl auth can-i create pods
kubectl auth can-i delete pods
kubectl auth can-i update pods
## 유저 satellite 의 네임스페이스 project-httpd-prod 에 대한 권한은 get, edit 이므로 모두 가능하다.
## 다만, edit 만 기재하여도 list 가 가능하므로, edit 만 명시하면 모든 권한이 활성화된다.

kubectl run httpd --image=httpd --dry-run=client -o yaml | kubectl apply -f -
## RUNNING
```

# K8S_RBAC-Question

아래 프로젝트 및 사용자 그리고 RBAC 를 구성해라.

```bash
Namespace: project-httpd-dev
satellite: edit

Namespace: project-httpd-prod
satellite: view
openshift: edit

위에서 project-httpd-dev 와 satellite 사용자를 제외한 project-httpd-prod 와 openshift 사용자를 생성하지 않았다.
앞에서 학습한 내용을 가지고 project-httpd-prod 생성 후, 사용자 openshift 및 satellite 를 올바른 Role 및 ClusterRole 기반으로 생성한다.
```

프로젝트 project-httpd-prod 를 생성

```bash
kubectl create ns project-httpd-prod

kubectl config use-context kubernetes-admin@kubernetes
kubectl config get-contexts
```

openshift 시스템 사용자 계정을 생성한다

```bash
sudo useradd openshift
sudo passwd openshift
```

openshift 시스템 계정에 X.509 기반으로 인증서 생성한다

```bash
cd ~openshift/

openssl genrsa -out openshift.key 2048

openssl req -new -key openshift.key -out openshift.csr -subj "/CN=openshift"
## 해당 명령어도 가능은 하다.

openssl req -new -key openshift.key -out openshift.csr -subj "/CN=openshift/O=$(groups)"
## 정석

openssl x509 -req -in openshift.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out openshift.crt -days 500

mkdir .certs
mv openshift.* .certs/

kubectl config set-credentials openshift --client-certificate=/home/openshift/.certs/openshift.crt --client-key=/home/openshift/.certs/openshift.key
## .kube/config user 항목 추가
```

생성된 사용자에게 .kube 라는 디렉터리를 생성 후, config 파일을 구성한다. 생성이 완료가 되면, config 파일에 인증서를 구성한다

```bash
kubectl config set-context openshift-context-1 --cluster=kubernetes --user=openshift --namespace=project-httpd-prod
## .kube/config context 항목 추가
```

Role, ClusterRole 에 edit 권한을 할당한다.

```bash
cat << EOF | tee rbac_question.yml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: openshift-edit
  namespace: project-httpd-prod
subjects:
- kind: User
  name: openshift
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: edit
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f rbac_question.yml

kubectl config get-contexts

kubectl config use-context openshift-context-1
kubectl config get-contexts

kubectl auth can-i list pods
kubectl auth can-i create pods
kubectl auth can-i delete pods
kubectl auth can-i update pods

kubectl config use-context kubernetes-admin@kubernetes
kubectl config get-contexts
kubectl delete -f rbac_question.yml
```

crictl ps 와 같은 정보는 아래 경로에서 정보를 가져온다.

cat /var/lib/containers/cache/blob-info-cache-v1.boltdb

# K8S_HPA

```bash
cat << EOF | tee index.php
<?php
  \$x = 0.0001;
  for (\$i = 0; \$i <= 1000000; \$i++) {
    \$x += sqrt(\$x);
  }
  echo "OK!";
?>
EOF

cat << EOF | tee Dockerfile
FROM php:5-apache
COPY index.php /var/www/html/index.php
RUN chmod a+rx index.php
EOF

podman build -t docker.io/wayles54/test-app -f Dockerfile .

buildah login docker.io

buildah push docker.io/wayles54/test-app

kubectl run test-app --image=docker.io/wayles54/test-app

kubectl get pod -o wide

kubectl run -it debugger --rm --image=wayles54/debugger-alpine -- /bin/sh

ps aux | grep apache

cat << EOF | tee apache-autoscale.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache-autoscale
spec:
  replicas: 1
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
        - name: php-apache
          image: docker.io/wayles54/test-app
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: 500m
            requests:
              cpu: 200m

---

apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
    - port: 80
  selector:
    run: php-apache
EOF

kubectl apply -f apache-autoscale.yaml

kubectl autoscale deployment.v1.apps/php-apache-autoscale --min=1 --max=3 --cpu-percent=2 --dry-run=client -o yaml | kubectl apply -f -

kubectl get hpa
# kubectl get horizontalpodautoscalers.autoscaling
## NAME                   REFERENCE                         TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
## php-apache-autoscale   Deployment/php-apache-autoscale   <unknown>/2%   1         3         1          23s

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl top pod

kubectl get hpa -w

kubectl run -it debugger --rm --image=wayles54/debugger-alpine -- /bin/sh

curl 172.16.104.12
```

# K8S_Health-Liveness

```bash
cat << EOF | tee liveness-pod-1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod-1
spec:
  containers:
    - image: httpd
      imagePullPolicy: IfNotPresent
      name: liveness-pod-1
      command: ['sh','-c','echo liveness-pod-1 container 1 is running ; sleep 4000']
      ports:
        - name: livness-pod-1
          containerPort: 80
          hostPort: 8080
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 4
        periodSeconds: 10
EOF

kubectl apply -f liveness-pod-1.yaml

kubectl describe pod liveness-pod-1
```

# K8S_Health-Readiness

```bash
cat << EOF | tee readiness.yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
spec:
  containers:
    - image: k8s.gcr.io/goproxy:0.1
      imagePullPolicy: IfNotPresent
      name: goproxy
      ports:
        - containerPort: 8080
          hostPort: 8080
      readinessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      livenessProbe:
        tcpSocket:
          port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
EOF

kubectl apply -f readiness.yaml

kubectl describe pod goproxy
```

# K8S_Job

```bash
cat << EOF | tee job.yml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel
  labels:
    chapter: job
spec:
  parallelism: 5
  completions: 10
  template:
    metadata:
      labels:
        chapter: jobs
    spec:
      containers:
        - name: kuard
          image: gcr.io/kuar-demo/kuard-amd64:1
          imagePullPolicy: Always
          args:
            - "--keygen-enable"
            - "--keygen-exit-on-complete"
            - "--keygen-num-to-gen=5"
      restartPolicy: OnFailure
EOF

kubectl apply -f job.yml

kubectl get jobs.batch
```

# K8S_ConfigMap

## for-literal

```bash
kubectl create cm test2 --for-literal=name=bora

kubectl describe cm test2
```

## from-file

```bash
cat << EOF | tee configmap.txt
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice
EOF

kubectl create configmap game-config --from-file=configmap.txt

kubectl describe configmaps game-config
```

# K8S_Secret

## for-literal

```bash
kubectl create secret generic test --from-literal=test=bora

kubectl describe secrets test
```

## Declarative

```bash
cat << EOF | tee secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret-yaml
type: Opaque
data:
  username: b3BlbnNoaWZ0
  password: cmVkaGF0
EOF

kubectl apply -f secret.yml

kubectl describe secrets mysecret-yaml

kubectl get secrets mysecret-yaml -o yaml

echo -n "cmVkaGF0" | base64 -d
```

# K8S_PV

## NFS

```bash
yum install -y nfs-utils

mkdir -p /nfs

cat << EOF | tee /etc/exports
/nfs *(rw,no_root_squash)
EOF

exportfs -avrs

cat << EOF | tee pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/nfs"

---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

---

apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage

EOF

kubectl apply -f pv.yml

kubectl get pod -o wide

echo hello > /nfs/index.html

kubectl run -it debugger --rm --image=wayles54/debugger-alpine -- /bin/sh

curl 172.16.104.22
```

# K8S_Node-Label

## Imperative

```bash
kubectl run nginx1 --image=nginx --labels="ver=1"

kubectl run nginx2 --image=nginx --labels="ver=2,env=prod"

kubectl label nodes node2 key=val
```

## Declarative

```yml
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
environment: production
app: nginx
spec:
  containers:
    - name: nginx
image: nginx:1.14.2
ports:
  - containerPort: 80
```

# K8S_Node-Get

```bash
kubectl label node node2 "ssd=true"

kubectl get nodes --selector "ssd=true"
```

# K8S_Node-Drain

```bash
kubectl get nodes

kubectl drain master

kubectl drain master --ignore-daemonsets

kubectl uncordon master
```

# K8S_Hard-Registry

```bash
cd /etc/containers

grep -Ev '^#|^$' registries.conf
## unqualified-search-registries = ["registry.fedoraproject.org", "registry.access.redhat.com", "docker.io"]
## podman, crio, buildah 는 상기 registry 지원
## containerd 는 상기 registry 미지원
```

# K8S_Hard-UFS

```bash
cd /var/lib/containers
## [kubectl cp] ---> API[kubelet] ---> NODE[kubeproxy] ---> RUNTIME[crio] --- {socket} --- { /var/lib/containers/ }

cd storage   				;; 오버레이 및 포드 같은 자질구리 정보들?
## /containers: image 정보
## /overlay-containers: runtime 정보, crictl ps
## /overlay-images: container images 정보, crictl images
## /overlay-layers: container image layer 정보, crictl inspecti + diff data
```

# Openshift-Intro

오픈시프트는 pause 컨테이너가 보이지 않는다. (통합되어 있음)

# Question

## 01

cri-o 가 구동되면 containerd 를 내부적으로 사용하는가?

K8S 설치 문서를 보면 Container Runtime 에 대해 docker , cri-o , containerd 중 하나를 선택하게 되어있는데, K8S 에서 cri-o 런타임을 사용하면 그림과 같이 내부적으로 containerd 를 사용하는 건가요?

containerd 도 cri-o 와 같은 컨테이너 런타임이고 containerd 내부 API 를 cri-o 에서 사용한다고 보면 된다.

https://containerd.io/docs/getting-started/

https://github.com/opencontainers/runc

https://containerd.io/docs/getting-started/#creating-an-oci-spec-and-container

```bash
ps -ef | grep runc
## conmon 이 containerd 에 포함된 spec 이다.

ps aux | grep conmon

man conmon
## -r

ps aux | grep conmon | grep /usr/bin/runc
## -r /usr/bin/runc -> 런타임이 runc

ls /usr/share/containers/
## containerd 라고 보면 됨
## mounts.conf
## oci
## containers.conf
## seccomp.json
```

## 02

강사님 혹시 쿠버네티스에 사용하는 인증서는 외부에서 구매한 사설 인증서도 사용하기도 하나요?

```bash
Cloud 환경에서는 사설 인증서를 연동해서 사용하기도 하지만, 베어메탈에서는 굳이 사설 인증서는 잘 사용하지 않는다. (비용 등)

예전에는 사설인증서 사용에 문제가 없었으나, 현재는 tls 등 통신에 이슈가 있을 수 있다.
```

## 03

CertificateSigningRequest 로 crt 를 받는 방식이 있어 보이는데 이렇게 직접 명령어로 crt 를 받는 방식과 차이가 있을까요?

```bash
openssl x509 -req -in satellite.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out satellite.crt -days 500
## 차이 없음
```

## 04

서비스 생성 시, SEP 와 같은 iptables 체인을 기재해주는 역할이 kube-proxy 인가요? 아니면 별개로 봐야 할까요?

```bash
https://fossies.org/linux/kubernetes/cmd/kube-proxy/app/server.go

"k8s.io/kubernetes/pkg/proxy/iptables"
"k8s.io/kubernetes/pkg/proxy/ipvs"
"k8s.io/kubernetes/pkg/proxy/userspace"
"k8s.io/kubernetes/pkg/util/ipvs"
"k8s.io/kubernetes/pkg/proxy"
"k8s.io/kubernetes/pkg/proxy/apis"
"k8s.io/client-go/rest"
"k8s.io/client-go/tools/clientcmd"
"k8s.io/client-go/tools/clientcmd/api"
"k8s.io/apimachinery/pkg/util/wait"
"k8s.io/apiserver/pkg/server/healthz"
"k8s.io/apiserver/pkg/server/mux"
"k8s.io/apiserver/pkg/server/routes"
"k8s.io/apimachinery/pkg/fields"
"k8s.io/apimachinery/pkg/labels"
"k8s.io/apimachinery/pkg/runtime"
"k8s.io/apimachinery/pkg/runtime/serializer"
## kube-proxy 가 iptables 를 갱신한다.
```
