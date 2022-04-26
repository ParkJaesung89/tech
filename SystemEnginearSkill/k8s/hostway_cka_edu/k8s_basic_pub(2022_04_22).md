> # K8S-Install

`OS : Ubuntu 20.04 LTS`

```bash
# apt 이슈가 있는 경우
sudo killall dpkg
sudo kill -9 $(lsof -t /var/lib/dpkg/lock)
sudo kill -9 $(lsof -t /var/lib/apt/lists/lock)
sudo kill -9 $(lsof -t /var/cache/apt/archives/lock)

sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock

sudo dpkg --configure -a

# Memory: 2GB 이상
free -mh

# CPU: 2코어 이상
nproc

# 스왑의 비활성화 (kubelet 구동 조건)
## Swap 사용 시 IOPS 의 급격한 저하로 인해 미권장
sudo swapoff /swap.img
sudo sed -i -e '/swap.img/d' /etc/fstab

# 고유한 MAC 주소 사용 여부 확인 (노드 식별에 필요)
ip link

# 고유한 Mainboard product UUID 사용 여부 확인 (노드 식별에 필요)
sudo cat /sys/class/dmi/id/product_uuid

# 라우팅 확인
sudo vi /etc/netplan/50-vagrant.yaml

---
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
      - 203.248.23.212/25
      routes:
      - to: 0.0.0.0/0
        via: 203.248.23.129

cd /etc/netplan/ && sudo netplan apply

ip route
```

`컨테이너 도커 설치`

```bash
# 모든 서버
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

sudo systemctl status docker
```

`컨테이너 도커 Cgroup 모듈 설정`

```bash
# 모든 서버
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl restart docker
sudo docker info | grep Cgroup
## Cgroup Driver: systemd
```

`K8S 사용 모듈 활성화`

```bash
# 모든 서버
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
```

`K8S 패키지 설치`

```bash
# 모든 서버
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

`K8S 컨트롤러 설치`

```bash
# 컨트롤러
sudo kubeadm init --ignore-preflight-errors=all --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=203.248.23.212
## --apiserver-advertise-address=203.248.23.212 -> Controller 서버 IP 로 변경한다.

# 컨트롤러
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 컨트롤러
kubeadm token create --print-join-command
## 토큰 확인

# 워커
sudo kubeadm join --token <token> <controlplane-host>:<controlplane-port> --ignore-preflight-errors=all --discovery-token-ca-cert-hash sha256:<hash>
## 컨트롤러의 kubeadm init 설치 완료 메세지 혹은 상기 명령어 결과 복사하여 위와 같은 형태로 복사 및 붙여넣기
```

`K8S 클러스터 상태 확인`

```bash
# 컨트롤러
kubectl get nodes
## NotReady

sudo journalctl -u kubelet --no-pager
## "Unable to update cni config" err="no networks found in /etc/cni/net.d"
## "Container runtime network not ready" networkReady="NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized"
```

`K8S 클러스터 네트워크 플러그인 설치`

```bash
# 컨트롤러
curl https://projectcalico.docs.tigera.io/manifests/calico.yaml -O
kubectl apply -f calico.yaml

watch -n1 kubectl get pods -A
kubectl get nodes -o wide
## Ready
```

`다수의 NIC(=Network Interface Card) 를 가지는 환경에서 INTERNAL-IP 설정`

다수의 네트워크가 존재하는 서버의 경우 K8S 에서 첫 번째 NIC 의 IP 가 INTERNAL-IP 로 자동 설정된다.
해당 INTERNAL-IP 를 상기 kubeadm --apiserver-advertise-address=203.248.23.212 와 동일한 IP 대역을 사용하도록 설정한다.

```bash
# 컨트롤러
cat << EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS='--node-ip $(hostname -I | cut -d ' ' -f2)'
EOF

# 컨트롤러
cat /etc/default/kubelet
## KUBELET_EXTRA_ARGS='--node-ip 203.248.23.212'

# 워커
cat << EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS='--node-ip $(hostname -I | cut -d ' ' -f2)'
EOF

# 워커
cat /etc/default/kubelet
## KUBELET_EXTRA_ARGS='--node-ip 203.248.23.213'

## Multi NIC 환경에서 INTERNAL-IP 가 다른 NIC 으로 설정되는 경우 직접 수동으로 수정한다.
## IP 는 각 Node 의 환경에 맞게 설정한다.

sudo systemctl daemon-reload
sudo systemctl restart kubelet

kubectl get nodes -o wide
kubectl cluster-info
```

`K8S 클러스터 테스트`

K8S 클러스터의 작동 정상 여부 확인을 위해 간단하게 nginx 서버를 생성해보자.

```bash
# 컨트롤러
kubectl run hello --image=nginx --dry-run=client -o yaml
## hello: 파드 이름

kubectl run hello --image=nginx --dry-run=client -o yaml | kubectl apply -f -

kubectl get pods -o wide
## STATUS: Running

kubectl logs hello
## hello: 파드 이름
```

> # K8S-Cluster-Set-IKS

IKS: IBMCloud Kubernetes Service

워커 노드 1EA 는 무료로 제공하며(2Core / 4GB), 30일마다 생성한 자원은 삭제된다. (자동 삭제된 후 재생성하면 추가로 사용 가능하다.)

IBM Cloud 검색 - Catalog - Services - Kubernetes Service - Pricing plan: Free
(**신용카드만 가능**)

```bash
Cluster name: ibm-cluster
## K8S-Cluster name 은 원하는 이름으로 변경 가능
```

`IBM Cloud 명령어 설치`

```bash
wget https://download.clis.cloud.ibm.com/ibm-cloud-cli/2.0.2/IBM_Cloud_CLI_2.0.2_amd64.tar.gz

tar xvzf IBM_Cloud_CLI_2.0.2_amd64.tar.gz

cd Bluemix_CLI/

sudo ./install
```

`IBM Cloud 플러그인 설치`

```bash
ibmcloud login

ibmcloud plugin install container-service

ibmcloud plugin install container-registry

ibmcloud plugin install observe-service

ibmcloud plugin list
```

`IKS 정보 추가`

```bash
ibmcloud ks cluster ls

ibmcloud target -g Default

ibmcloud ks cluster ls

cp ~/.kube/config ~/.kube/config_ori
## ~/.kube/config 파일의 경우 K8S 의 접속 정보가 저장되어 있는 파일이다. 해당 파일에 IKS 정보를 추가하기 전에 백업을 진행한다.

ibmcloud ks cluster config --cluster ibm-cluster
# --cluster [Cluster Name]
## 본인이 설정한 클러스터 이름으로 입력한다.

kubectl config current-context

kubectl config get-contexts

kubectl get nodes -o wide
```

`IKS 테스트`

```bash
kubectl run nginx --image=nginx

kubectl get pod -o wide

kubectl run -it test --image=wayles54/debugger-alpine --rm=true -- sh

curl 172.30.250.202
```

`K8S 클러스터 Switch`

접속 정보 파일(.kube/config) 이 현재 IKS 로 되어있으므로 위에서 설치한 로컬 K8S 클러스터로 변경한다.

```bash
kubectl config get-contexts

kubectl config use-context kubernetes-admin@kubernetes

kubectl get nodes -o wide
```

> # K8S-Cluster-Set-Docker_Desktop

`Docker Desktop 설치`

```bash
Settings - Kubernetes - Enable Kubernetes

kubectl config get-contexts
## docker-desktop Node 확인

kubectl run hello --image=nginx

kubectl get pods

kubectl exec hello -- curl 127.0.0.1
```

`Docker Desktop 주요 파일`

docker-desktop 의 config 은 하기 경로에 있다.

```bash
/C/Users/admin-mgmt/.kube/config
```

> # K8S-Cluster-Set-Shell

`자동 완성`

```bash
echo '' >>~/.bashrc
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -F __start_kubectl k' >>~/.bashrc

. ~/.bashrc
```

`테스트`

```bash
k get nodes -o wide
kubectl get nodes -o wide
## Tab 자동완성 확인
```

> # K8S-Cluster-Set-Taint

Taint 는 특정 Node(=K8S 가 설치된 서버 혹은 VM) 에 자원이 생성되지 않도록 방지하는 기능이다.
컨트롤러 노드의 경우 자원이 생성되지 않도록 설정되어 있으므로 해당 설정을 제거하여 자원이 생성되도록 한다.
**(운영 환경에서는 Taint 설정 해제 미권장)**

`Taint 기본 정책 확인`

```bash
kubectl get nodes

kubectl describe nodes user-controller

kubectl describe nodes user-controller | grep Taint
## Taints:             node-role.kubernetes.io/master:NoSchedule

kubectl describe nodes user-worker | grep Taint
## Taints:             <none>

kubectl create deployment hello --image=nginx --replicas=3

kubectl get pods -o wide
# 모두 user-worker 노드에 스케줄링 되었음을 확인할 수 있다.
## NAME                     READY   STATUS    RESTARTS   AGE   IP                NODE          NOMINATED NODE   READINESS GATES
## hello-776c774f98-5xjj4   1/1     Running   0          12s   192.168.153.238   user-worker   <none>           <none>
## hello-776c774f98-t2ghw   1/1     Running   0          12s   192.168.153.239   user-worker   <none>           <none>
## hello-776c774f98-tl874   1/1     Running   0          12s   192.168.153.237   user-worker   <none>           <none>

kubectl delete deployment hello
```

`Taint 기본 정책 제거`

```bash
kubectl taint nodes user-controller node-role.kubernetes.io/master:NoSchedule-

kubectl describe nodes user-controller | grep Taint
## Taints:             <none>

kubectl create deployment hello --image=nginx --replicas=3

kubectl get pods -o wide
# 이제 user-controller 노드에도 스케줄링 되었음을 확인할 수 있다.
## NAME                     READY   STATUS    RESTARTS   AGE   IP                NODE              NOMINATED NODE   READINESS GATES
## hello-776c774f98-cr2tg   1/1     Running   0          9s    192.168.136.14    user-controller   <none>           <none>
## hello-776c774f98-fhqnr   1/1     Running   0          9s    192.168.153.240   user-worker       <none>           <none>
## hello-776c774f98-kk86m   1/1     Running   0          9s    192.168.153.241   user-worker       <none>           <none>
```


> # K8S-Intro-VM


> ### K8S-Intro-VM-Definition

VM 은 하이퍼바이저를 통해 구동되며, Host OS 에 여러 개의 가상 컴퓨터(=Software)를 구동한다.

VM 은 CPU, Memory, Disk, NIC 등 OS 에 필요한 가상화된 자원으로 구동된다.

```
- Host OS: 보통의 경우 물리 서버 OS
- Guest OS: VM 에서 구동되는 OS
```


> ### K8S-Intro-VM-Feature

`자원의 유연한 확장 및 축소`

```
- CPU, Memory 스펙의 유연한 조절 (=하드웨어가 아닌 소프트웨어로 조작 가능)
```

`유지보수`

```
- 백업 용이
```

> ### K8S-Intro-VM-Diff

VM 과 Container 의 차이에 대해 알아보자.

> ### K8S-Intro-VM-Diff-Virtualization-VM

- VM 은 커널, Library, OS 를 가상화 한다. 즉, 독립된 OS 시스템을 생성한다.

  - 독립된 커널로 인해 보안상 좋다.
  - 독립된 커널로 인해 Host OS 와 커널 버전이 다르다.

- 용량이 매우 크다. (Virtualbox CentOS 07 기준 1.8GB)

  - 용량이 크므로 구동 시간이 느리다.

- 많은 메모리를 사용한다.

```bash
ps aux | grep VirtualBox
## 약 714MB 메모리 에약 / 72MB 메모리 사용
```

> ### K8S-Intro-VM-Diff-Virtualization-Container

- Container 는 Library, OS 를 가상화 하며, Kernel 은 Host OS 와 공유한다.
  - 공유된 커널로 인해 보안상 좋지 않다.
  - 공유된 커널로 인해 Host OS 와 커널 버전이 동일하다.

```bash
# CentOS 07 Host OS
uname -r
## 3.10.0-1160.62.1.el7.x86_64

# Container
docker run ubuntu uname -r
## 3.10.0-1160.62.1.el7.x86_64
```

- 용량이 매우 작다. (Container CentOS 07 기준 144MB)
  - 용량이 작으므로 구동 시간이 빠르다.

```bash
docker run -d centos:7 sleep 1000
docker container ls --size
# SIZE: 0B (virtual 204MB)
## [Container Size] ([Image Size])
```

- 적은 메모리를 사용한다.

```bash
docker run -d centos:7 sleep 1000
## CentOS 07 Container 구동

ps aux | grep sleep
## 약 4MB 메모리 에약 / 0.7MB 메모리 사용
```

> ### K8S-Intro-VM-Diff-Compatibility-VM & Container

- VM
  - Host OS 의 커널이 아닌 별도로 생성 및 사용하므로 Host OS 커널에서 미지원하는 OS 를 구동할 수 있다. (Ex. Linux Host 에 Window VM 구동 가능)    
<br>


- Container
  - Host OS 의 커널을 사용하므로 Host OS 커널에서 지원하는 OS 만 구동 가능하다. (Ex. Linux Host 에 Window 컨테이너 구동 불가)


> ### K8S-Intro-Container

리눅스 격리 기술이 발전한 순서대로 나열 및 그 특징을 기재한다.

```bash
Chroot - Namespace - Cgroup - LXC Container - Docker Container - K8S
```

> ### K8S-Intro-Container-Chroot-Definition

Chroot = Change Root Directory

특정 경로를 새로운 Root 디렉토리로 변경하며, 해당 파일시스템 내부의 바이너리 파일을 격리된 환경에서 실행 및 격리하는 기능 (불완전 요소 존재)

격리된 chroot 환경에서 내부 바이너리 파일을 실행할 수 있으나, **사용 기능(namespace)** 및 **자원(cgroup)** 에 대한 제한이 어렵다.

> ### K8S-Intro-Container-Chroot-Set-Bash

chroot 환경에서 bash 를 구성해보자.

```bash
mkdir ~/bash/
## chroot 환경의 / 로 구성

mkdir ~/bash/bin/
# echo $PATH
# /sbin:/bin:/usr/sbin:/usr/bin
## bash 쉘의 기본 PATH 가 위와 같으므로 해당 예제에서는 /bin 에 바이너리 파일 복사

mkdir ~/bash/lib64/
## 동적 라이브러리 경로

sudo chroot ~/bash /bin/bash
## root 권한 필요
## chroot: failed to run command '/bin/bash'

# bash
sudo cp /usr/bin/bash ~/bash/bin/

ldd /bin/bash
## bash 가 어떤 동적 라이브러리를 참조하는지 조회

sudo cp /lib64/libtinfo.so.5 ~/bash/lib64/
sudo cp /lib64/libdl.so.2 ~/bash/lib64/
sudo cp /lib64/libc.so.6 ~/bash/lib64/
sudo cp /lib64/ld-linux-x86-64.so.2 ~/bash/lib64/
## bash 실행 시 /lib64 경로의 동적 라이브러리를 참조하므로 chroot 환경도 ~/bash/lib64/ 에 동적 라이브러리 파일 복사

# ls
sudo cp /usr/bin/ls ~/bash/bin/

ldd /usr/bin/ls

sudo cp /lib64/libselinux.so.1 ~/bash/lib64/
sudo cp /lib64/libcap.so.2 ~/bash/lib64/
sudo cp /lib64/libacl.so.1 ~/bash/lib64/
sudo cp /lib64/libc.so.6 ~/bash/lib64/
sudo cp /lib64/libpcre.so.1 ~/bash/lib64/
sudo cp /lib64/libdl.so.2 ~/bash/lib64/
sudo cp /lib64/ld-linux-x86-64.so.2 ~/bash/lib64/
sudo cp /lib64/libattr.so.1 ~/bash/lib64/
sudo cp /lib64/libpthread.so.0 ~/bash/lib64/

# cat
sudo cp /usr/bin/cat ~/bash/bin/

ldd /usr/bin/cat

sudo cp /lib64/libc.so.6 ~/bash/lib64/
sudo cp /lib64/ld-linux-x86-64.so.2 ~/bash/lib64/

sudo chroot ~/bash /bin/bash
# bash-4.2
## bash 로 접근된다.

bash-4.2# ls
```

> ### K8S-Intro-Container-Chroot-Set-CentOS_08

CentOS_07 에서 CentOS_08 을 chroot 로 구동해보자.

iso 파일은 설치 패키지가 포함된 파일로 파일시스템은 존재하지 않는다. 대신 도커의 파일시스템을 복사하여 chroot 환경에 사용한다.

```bash
# Host
docker run -it --name centos-8 centos:8 bash

Ctrl ^P ^Q

mkdir ~/centos_08/
mkdir ~/centos_08/proc/

docker cp centos-8:bin ~/centos_08/
docker cp centos-8:dev ~/centos_08/
docker cp centos-8:etc ~/centos_08/
docker cp centos-8:home ~/centos_08/
docker cp centos-8:lib ~/centos_08/
docker cp centos-8:lib64 ~/centos_08/
docker cp centos-8:media ~/centos_08/
docker cp centos-8:mnt ~/centos_08/
docker cp centos-8:opt ~/centos_08/
docker cp centos-8:run ~/centos_08/
docker cp centos-8:sbin ~/centos_08/
docker cp centos-8:srv ~/centos_08/
docker cp centos-8:var ~/centos_08/
sudo docker cp centos-8:usr ~/centos_08/
sudo docker cp centos-8:root ~/centos_08/

sudo chroot ~/centos_08/ bash

cat /etc/redhat-release
# CentOS Linux release 8.4.2105
## 기존 파일시스템의 /etc/redhat-release 의 참조가 아닌 새로운 파일시스템의 /etc/redhat-release 를 참조

mount -t proc none /proc
mount
cat /etc/mtab
cat /proc/version
## Host OS 의 버전이 출력된다. chroot 환경은 내부 커널의 사용이 아닌 Host OS 커널 사용 (불완전 격리 증거)
```

> ### K8S-Intro-Container-Chroot-Set-Ubuntu_18

Ubuntu_20 에서 Ubuntu_18 을 chroot 로 구동해보자.

Debian 계열의 OS 는 해당 계열 파일시스템을 제공하는 debootstrap 패키지를 지원한다. 해당 파일시스템을 다운로드하여 chroot 환경에 사용한다.

```bash
# Host
sudo apt update
sudo apt install -y debootstrap
sudo debootstrap --variant=minbase bionic ~/ubuntu_18

sudo chroot ~/ubuntu_18/ bash

# Chroot
cat /etc/issue
# Ubuntu 18.04 LTS \n \l
## 기존 파일시스템의 /etc/redhat-release 의 참조가 아닌 새로운 파일시스템의 /etc/redhat-release 를 참조

mount -t proc none /proc
mount
cat /etc/mtab
cat /proc/version
## Host OS 의 버전이 출력된다. chroot 환경은 내부 커널의 사용이 아닌 Host OS 커널 사용 (불완전 격리 증거)
```

> ### K8S-Intro-Container-Chroot-Set-Ubuntu_20

Ubuntu_20 에서 Ubuntu_20 을 chroot 로 구동해보자.

Debian 계열의 OS 는 해당 계열 파일시스템을 제공하는 debootstrap 패키지를 지원한다. 해당 파일시스템을 다운로드하여 chroot 환경에 사용한다.

```bash
# Host
sudo apt update
sudo apt install -y debootstrap
sudo debootstrap --variant=minbase focal ~/ubuntu_20

sudo chroot ~/ubuntu_20/ bash

# Chroot
cat /etc/issue
# Ubuntu 20.04 LTS \n \l
## 기존 파일시스템의 /etc/redhat-release 의 참조가 아닌 새로운 파일시스템의 /etc/redhat-release 를 참조

mount -t proc none /proc
mount
cat /etc/mtab
cat /proc/version
## Host OS 의 버전이 출력된다. chroot 환경은 내부 커널의 사용이 아닌 Host OS 커널 사용 (불완전 격리 증거)
```

> ### K8S-Intro-Container-Chroot-Feature-Path

chroot 로 격리된 환경은 부모 디렉토리에 접근하지 못한다.

```bash
# Host
sudo chroot ~/bash/ bash

# Chroot
bash-4.2# pwd
# /
## ~/bash 가 / 로 변경된다.

bash-4.2# ../
# /
## / 를 벗어날 수 없다.
## 격리된 chroot 환경에서 해당 디렉토리를 벗어날 수 없으며, Host 의 루트 디렉토리에 접근할 수 없다.
## Jail Breaking 공격이 있긴 하지만 보통의 경우 `cd ../` 결과가 지속 `/` 로 고정된다.
```

> ### K8S-Intro-Container-Chroot-Feature-User

chroot 로 격리된 환경의 세션은 root 권한을 소유한다. (보안 이슈)

```bash
# Host
sudo chroot ~/bash /bin/bash

# Chroot
bash-4.2# echo hello > test
bash-4.2# exit

# Host
ls -al ~/bash/test
## -rw-r--r--. 1 root root 6 Apr 19 10:09 /home/vagrant/bash/test
```

> ### K8S-Intro-Container-Chroot-Feature-UID

chroot 로 격리된 환경에서 생성된 UID 출력과정

```bash
# Host
sudo chroot ~/ubuntu_20/ bash

# Chroot
useradd test
cat /etc/passwd | grep test
ls -al test
# -rw-r--r-- 1 test test 0 Apr 19 01:57 test
## Chroot 환경에서 최초로 생성한 User 로 UID 1000 을 할당 받는다.

touch test
chown test.test test
exit

# Host
ls -al ~/ubuntu_20/test
# -rw-r--r-- 1 admin-mgmt admin-mgmt 0 Apr 19 01:57 /home/vagrant/ubuntu_20/test
## Host OS 에서는 UID 1000 에 대한 유저가 이미 있으므로 소유 권한이 잘못 표시된다.
```

> ### K8S-Intro-Container-Chroot-Feature-PID

chroot 로 격리된 환경에서 Host OS 의 전체 Process 목록을 볼 수 있다. (불완전 격리 증거)

`ps`

```bash
# Host
mkdir -p ~/bash/proc/

# mount
sudo cp /usr/bin/mount ~/bash/bin/

ldd /usr/bin/mount

sudo cp /lib64/libmount.so.1 ~/bash/lib64/
sudo cp /lib64/libblkid.so.1 ~/bash/lib64/
sudo cp /lib64/libuuid.so.1 ~/bash/lib64/
sudo cp /lib64/libselinux.so.1 ~/bash/lib64/
sudo cp /lib64/libc.so.6 ~/bash/lib64/
sudo cp /lib64/ld-linux-x86-64.so.2 ~/bash/lib64/
sudo cp /lib64/libpcre.so.1 ~/bash/lib64/
sudo cp /lib64/libdl.so.2 ~/bash/lib64/
sudo cp /lib64/libpthread.so.0 ~/bash/lib64/

# ps
sudo cp /usr/bin/ps ~/bash/bin/

ldd /usr/bin/ps

sudo cp /lib64/libprocps.so.4 ~/bash/lib64/
sudo cp /lib64/libsystemd.so.0 ~/bash/lib64/
sudo cp /lib64/libdl.so.2 ~/bash/lib64/
sudo cp /lib64/libc.so.6 ~/bash/lib64/
sudo cp /lib64/libcap.so.2 ~/bash/lib64/
sudo cp /lib64/libm.so.6 ~/bash/lib64/
sudo cp /lib64/librt.so.1 ~/bash/lib64/
sudo cp /lib64/libselinux.so.1 ~/bash/lib64/
sudo cp /lib64/liblzma.so.5 ~/bash/lib64/
sudo cp /lib64/liblz4.so.1 ~/bash/lib64/
sudo cp /lib64/libgcrypt.so.11 ~/bash/lib64/
sudo cp /lib64/libgpg-error.so.0 ~/bash/lib64/
sudo cp /lib64/libresolv.so.2 ~/bash/lib64/
sudo cp /lib64/libdw.so.1 ~/bash/lib64/
sudo cp /lib64/libgcc_s.so.1 ~/bash/lib64/
sudo cp /lib64/libpthread.so.0 ~/bash/lib64/
sudo cp /lib64/ld-linux-x86-64.so.2 ~/bash/lib64/
sudo cp /lib64/libattr.so.1 ~/bash/lib64/
sudo cp /lib64/libpcre.so.1 ~/bash/lib64/
sudo cp /lib64/libelf.so.1 ~/bash/lib64/
sudo cp /lib64/libz.so.1 ~/bash/lib64/
sudo cp /lib64/libbz2.so.1 ~/bash/lib64/

sudo chroot ~/bash/

mount -t proc none /proc

ps aux
## Host OS 의 전체 PID 목록이 출력된다. (불완전 격리 증거)
```

> ### K8S-Intro-Container-Chroot-Feature-Kill

chroot 로 격리된 환경에서 Host OS 의 Process를 kill 할 수 있다. (불완전 격리 증거)

```bash
# Host
sleep 1000 &

ps -ef | grep sleep

sudo chroot ~/test

# Chroot
bash-4.2# kill -9 _PID_
## Host OS PID 종료 가능

exit

# Host
ps -ef | grep sleep
## Host OS 의 Process가 종료되었다.
```

> ### K8S-Intro-Container-Chroot-Feature-Hostname

chroot 로 격리된 환경에서 Hostname 을 변경하면 Host OS 의 Hostname 도 변경된다. (불완전 격리 증거)

```bash
# Host
sudo chroot ~/ubuntu_20/

# Chroot
hostname test
exit

# Host
hostname
# test
## chroot 환경에서 설정한 호스트네임이 Host OS 에도 적용된다.
```

> ### K8S-Intro-Container-Chroot-Feature-Limit

chroot 로 격리된 환경에서는 Resource Limit 을 미지원한다. (불완전 격리 증거)

```bash
# Host
watch -n1 free -mh

# Chroot
sudo chroot ~/bash/ /bin/bash

while true; do bash & done
## 지속 생성되는 bash Process로 인해 메모리가 계속 증가된다.
```

> ### K8S-Intro-Container-Chroot-Feature-Summary

종합해보면 chroot 는 격리된 환경을 제공하지만,여러가지 측면에서 불완전 하다.

```bash
- chroot 로 격리된 환경의 세션은 root 권한을 소유한다. (사실 해당 이슈는 K8S 도 동일하다.)

- chroot 로 격리된 환경에서 Host OS 의 전체 Process 목록을 볼 수 있다.

- chroot 로 격리된 환경에서 Host OS 의 Process를 kill 할 수 있다.

- chroot 로 격리된 환경에서 Hostname 을 변경하면 Host OS 의 Hostname 도 변경된다.

- chroot 로 격리된 환경에서는 Resource Limit 을 미지원한다.
```

> # K8S-Intro-Container-Namespace-Definition

네임스페이스는 커널의 기능 중 하나이며, 특정 Process가 Host OS 커널의 기능이 제한될 수 있도록 격리한다.

리눅스에서 지원하는 네임스페이스 주요 기능은 다음과 같다.

```bash
- user namespace: UID, GID 등의 자원을 격리
- pid namespace: PID 자원을 격리
- network namespace: ip, routing 등의 자원을 격리
- mount namespace: mount 자원을 격리
- uts namespace: hostname, domain_name 등을 격리
```

> ### K8S-Intro-Container-Namespace-Set

unshare 명령어는 Namespace 의 사용을 간편하게 해주는 명령어이다. 해당 명령어를 패키지 관리자를 통해 설치해보자.

```bash
# CentOS
sudo yum install -y util-linux

# Ubuntu
sudo apt update
sudo apt install -y util-linux
## unshare
```

> ### K8S-Intro-Container-Namespace-Feature-PID

Namespace pid 기능이 적용된 chroot 환경에서 구동중인 PID 를 격리해보자. Host OS 전체 PID 가 아닌 chroot 환경에서 실행된 PID 만 출력 예상된다.

Namespace pid 기능이 적용되면 Host OS 의 PID 와 chroot 환경의 PID 가 다르다.

```bash
# Host
sudo unshare --pid -f chroot ~/ubuntu_20 bash

# Chroot
mount -t proc none /proc # process namespace

ps aux
## chroot 환경에서 생성된 Process의 PID 목록만 출력된다.
```

> ### K8S-Intro-Container-Namespace-Feature-Kill

Namespace pid 기능이 적용된 chroot 환경에서 Host OS 의 PID 를 kill 해보자. Host OS 의 PID 를 모르기 때문에 kill 불가 예상된다.

```bash
# Host
sleep 1000 &

ps -ef | grep sleep

sudo unshare --pid -f chroot ~/ubuntu_20 bash

# Chroot
kill -9 _PID_
# bash: kill: (70492) - No such process
```

> ### K8S-Intro-Container-Namespace-Feature-Hostname

Namespace uts 기능이 적용된 chroot 환경에서 Hostname 을 변경해보자. Host OS Hostname 은 미변경 예상된다.

```bash
# Host
sudo unshare --uts chroot ~/ubuntu_20 bash

# Chroot
hostname hello
exit

# Host
hostname
## chroot 환경에서 설정한 호스트네임이 Host OS 에 적용되지 않는다.
```

> # K8S-Intro-Container-Cgroup-Definition

Process가 사용하는 자원을 제한하는 리눅스 커널의 기능

Cgroup 제한 가능한 자원은 주로 다음과 같다.

```bash
- CPU
- Memory
- Network
- Device
- I/O
```

> ### K8S-Intro-Container-Cgroup-Set

libcgroup-tools 패키지는 Cgroup 의 사용을 간편하게 해주는 명령어이다. 해당 명령어를 패키지 관리자를 통해 설치해보자.

```bash
# CentOS
sudo yum install -y libcgroup-tools

# Ubuntu
sudo apt update
sudo apt install -y cgroup-tools
```

> ### K8S-Intro-Container-Cgroup-Oper-CPU-Host

Host 에서 생성된 Process 를 Cgroup 과 연결하여 CPU 사용량을 제한해보자.

CPU 사용량은 Cgroup 의 제한값으로 제한 및 유지되며, 해당 Process가 종료되지는 않는다.

`Manual`

```bash
# Host
sudo mkdir -p /sys/fs/cgroup/cpu/test/
## test cgroup 생성

cat /sys/fs/cgroup/cpu/cpu.cfs_quota_us
# -1
## 기본값은 무제한이다.

cat << EOF | sudo tee /sys/fs/cgroup/cpu/test/cpu.cfs_quota_us
20000
EOF
## CPU 사용량 20% 제한

stress -c 1 &

top
## CPU 100% 사용
## PID 참조

cat << EOF | sudo tee /sys/fs/cgroup/cpu/test/cgroup.procs
20462
EOF

top
## CPU 사용량이 약 20% 로 제한됨을 확인할 수 있다.

sudo cgdelete cpu:test
## test Cgroup cpu 삭제

ls -al /sys/fs/cgroup/cpu/ | grep test
## /sys/fs/cgroup/cpu/test 삭제됨
```

`Command`

```bash
sudo cgcreate -g cpu:test
## test Cgroup 생성

ls -al /sys/fs/cgroup/cpu/ | grep test

sudo cgset -r cpu.cfs_quota_us=20000 test
## 20%

sudo cgexec -g cpu:test stress -c 1

top
## 20% 언저리에서 유지된다.

sudo cgdelete cpu:test
## test Cgroup cpu 삭제

ls -al /sys/fs/cgroup/cpu/ | grep test
## /sys/fs/cgroup/cpu/test 삭제됨
```

> ### K8S-Intro-Container-Cgroup-Oper-CPU-Chroot

chroot 환경은 Process 를 격리하지 못하므로 상기 Host Process Cgroup 제한과 결과는 동일하다.


> ### K8S-Intro-Container-Cgroup-Oper-CPU-Namespace

Namespace 로 격리된 Process 를 Cgroup 과 연결하여 CPU 사용량을 제한해보자.

CPU 사용량은 Cgroup 의 제한값으로 제한 및 유지되며, 해당 Process가 종료되지는 않는다.

`Manual`

```bash
# Host
sudo mkdir -p /sys/fs/cgroup/cpu/test/
## test cgroup 생성

cat /sys/fs/cgroup/cpu/cpu.cfs_quota_us
# -1
## 기본값은 무제한이다.

cat << EOF | sudo tee /sys/fs/cgroup/cpu/test/cpu.cfs_quota_us
20000
EOF
## CPU 사용량 20% 제한

# stress
sudo cp /usr/bin/stress ~/ubuntu_20/bin/

ldd /usr/bin/stress

sudo cp /lib/x86_64-linux-gnu/libm.so.6 ~/ubuntu_20/lib/
sudo cp /lib/x86_64-linux-gnu/libc.so.6 ~/ubuntu_20/lib/
sudo cp /lib64/ld-linux-x86-64.so.2 ~/ubuntu_20/lib/

sudo unshare --pid -f chroot ~/ubuntu_20 bash

# Chroot
mount -t proc proc /proc
stress -c 1

# Host
top
## CPU 100% 사용
## PID 참조

cat << EOF | sudo tee /sys/fs/cgroup/cpu/test/cgroup.procs
21450
EOF

top
## CPU 사용량이 약 20% 로 제한됨을 확인할 수 있다.

sudo cgdelete cpu:test
## test Cgroup cpu 삭제

ls -al /sys/fs/cgroup/cpu/ | grep test
## /sys/fs/cgroup/cpu/test 삭제됨
```

`Command`

```bash
# Host
sudo cgcreate -g cpu:test
## test Cgroup 생성

ls -al /sys/fs/cgroup/cpu/ | grep test

sudo cgset -r cpu.cfs_quota_us=20000 test
## 20%

# stress
sudo cp /usr/bin/stress ~/ubuntu_20/bin/

ldd /usr/bin/stress

sudo cp /lib/x86_64-linux-gnu/libm.so.6 ~/ubuntu_20/lib/
sudo cp /lib/x86_64-linux-gnu/libc.so.6 ~/ubuntu_20/lib/
sudo cp /lib64/ld-linux-x86-64.so.2 ~/ubuntu_20/lib/

sudo cgexec -g cpu:test unshare --pid -f chroot ~/ubuntu_20 stress -c 1

top
## CPU 사용량이 약 20% 로 제한됨을 확인할 수 있다.

sudo cgdelete cpu:test
## test Cgroup cpu 삭제

ls -al /sys/fs/cgroup/cpu/ | grep test
## /sys/fs/cgroup/cpu/test 삭제됨
```

> ### K8S-Intro-Container-Cgroup-Oper-Memory-Host

Host 에서 생성된 Process 를 Cgroup 과 연결하여 Memory 사용량을 제한해보자.

Memory 사용량은 Cgroup 의 제한값에 도달하면 해당 Process가 종료되지 않고 재할당을 시도한다. 해당 작업이 실패하는 경우 OOM 과정이 진행되며 종료된다.

`Manual`

```bash
# Host
sudo mkdir -p /sys/fs/cgroup/memory/test/
## test cgroup 생성

cat /sys/fs/cgroup/memory/test/memory.limit_in_bytes
# 9223372036854771712
## 단위는 Bytes 이며, 매우 큰 숫자가 출력된다.

cat << EOF | sudo tee /sys/fs/cgroup/memory/test/memory.limit_in_bytes
15000000
EOF
## Memory 사용량 15MB 제한

stress --vm 1 --vm-bytes 100m
## --vm _process_count_
## --vm-bytes _bytes_

top
## PID 참조

ps aux | grep 29022
# vagrant    29022  101  1.3 106260 56332 pts/1    R+   04:20   0:15 stress --vm 1 --vm-bytes 100m
## 약 100MB 예약 할당 / 약 20 ~ 100MB 실점유

cat << EOF | sudo tee /sys/fs/cgroup/memory/test/cgroup.procs
29022
EOF

ps aux | grep 29022
# vagrant    29022 82.2  0.3 106260 14676 pts/1    R+   04:20   1:20 stress --vm 1 --vm-bytes 100m
## 약 14MB 에서 더 이상 메모리 실사용량이 증가하지 않는다.

sudo cgdelete memory:test
## test Cgroup memory 삭제

ls -al /sys/fs/cgroup/memory/ | grep test
## /sys/fs/cgroup/memory/test 삭제됨
```

`Command`

```bash
sudo cgcreate -g memory:test
## test Cgroup 생성

ls -al /sys/fs/cgroup/memory/ | grep test

sudo cgset -r memory.limit_in_bytes=15000000 test
## 15MB

sudo cgexec -g memory:test stress --vm 1 --vm-bytes 100m

top
## PID 참조

ps aux | grep 29230
# root       29230 51.1  0.3 106260 14344 pts/1    R+   04:25   0:16 stress --vm 1 --vm-bytes 100m
## 약 14MB 에서 더 이상 메모리 실사용량이 증가하지 않는다.

sudo cgdelete memory:test
## test Cgroup memory 삭제

ls -al /sys/fs/cgroup/memory/ | grep test
## /sys/fs/cgroup/memory/test 삭제됨
```

> ### K8S-Intro-Container-Cgroup-Oper-Memory-Chroot

chroot 환경은 Process 를 격리하지 못하므로 상기 Host Process Cgroup 제한과 결과는 동일하다.
<br>

> ### K8S-Intro-Container-Cgroup-Oper-Memory-Namespace

Namespace 로 격리된 Process 를 Cgroup 과 연결하여 Memory 사용량을 제한해보자.

Memory 사용량은 Cgroup 의 제한값에 도달하면 해당 Process가 종료되지 않고 재할당을 시도한다. 해당 작업이 실패하는 경우 OOM 과정이 진행되며 종료된다.

`Manual`

```bash
# Host
sudo mkdir -p /sys/fs/cgroup/memory/test/
## test cgroup 생성

cat /sys/fs/cgroup/memory/memory.limit_in_bytes
# 9223372036854771712
## 기본값은 무제한이다.

cat << EOF | sudo tee /sys/fs/cgroup/memory/test/memory.limit_in_bytes
15000000
EOF
## Memory 사용량 15MB 제한

# stress
sudo cp /usr/bin/stress ~/ubuntu_20/bin/

ldd /usr/bin/stress

sudo cp /lib/x86_64-linux-gnu/libm.so.6 ~/ubuntu_20/lib/
sudo cp /lib/x86_64-linux-gnu/libc.so.6 ~/ubuntu_20/lib/
sudo cp /lib64/ld-linux-x86-64.so.2 ~/ubuntu_20/lib/

sudo unshare --pid -f chroot ~/ubuntu_20 bash

# Chroot
mount -t proc proc /proc
stress --vm 1 --vm-bytes 100m

top
## PID 참조

ps aux | grep 29741
# root       29741  104  2.3 106260 94556 pts/1    R+   04:44   0:15 stress --vm 1 --vm-bytes 100m
## 약 100MB 예약 할당 / 약 20 ~ 100MB 실점유

cat << EOF | sudo tee /sys/fs/cgroup/memory/test/cgroup.procs
29741
EOF

ps aux | grep 29741
# root       29741 96.8  0.3 106260 14388 pts/1    D+   04:44   2:12 stress --vm 1 --vm-bytes 100m
## 약 14MB 에서 더 이상 메모리 실사용량이 증가하지 않는다.

sudo cgdelete memory:test
## test Cgroup memory 삭제

ls -al /sys/fs/cgroup/memory/ | grep test
## /sys/fs/cgroup/memory/test 삭제됨
```

`Command`

```bash
# Host
sudo cgcreate -g memory:test
## test Cgroup 생성

ls -al /sys/fs/cgroup/memory/ | grep test

sudo cgset -r memory.limit_in_bytes=15000000 test
## 15MB

# stress
sudo cp /usr/bin/stress ~/ubuntu_20/bin/

ldd /usr/bin/stress

sudo cp /lib/x86_64-linux-gnu/libm.so.6 ~/ubuntu_20/lib/
sudo cp /lib/x86_64-linux-gnu/libc.so.6 ~/ubuntu_20/lib/
sudo cp /lib64/ld-linux-x86-64.so.2 ~/ubuntu_20/lib/

sudo cgexec -g memory:test unshare --pid -f chroot ~/ubuntu_20 stress --vm 1 --vm-bytes 100m

top
## PID 참조

ps aux | grep 29955
# root       29955 52.9  0.3 106260 14176 pts/1    D+   04:48   0:43 stress --vm 1 --vm-bytes 100m
## 약 14MB 에서 더 이상 메모리 실사용량이 증가하지 않는다.

sudo cgdelete memory:test
## test Cgroup memory 삭제

ls -al /sys/fs/cgroup/memory/ | grep test
## /sys/fs/cgroup/memory/test 삭제됨
```

> ### K8S-Intro-Container-Summary

Container 는 **Namespace, Cgroup** 등의 기능과 연결된 Process 이다.
**Docker** 와 같은 컨테이너 관련 솔루션은 이를 개선 및 쉽게 사용하는 기능을 제공한다.
<br>

> # K8S-Intro-Term

`Workload`
```
- 쿠버네티스에서 구동되는 애플리케이션이다.
```

`ControlPlane`
```
- K8S 를 관리하는 역할을 충족하는 서버 집합
- Controller 서버 집합은 ControlPlane 에 속한다.
```

`Controller`
```
- Worker 노드를 관리하는 노드(=서버, 현재 환경에서는 VM)
```

`Worker`
```
- Workload(=Pod) 가 구동되는 노드(=서버, 현재 환경에서는 VM)
```

`Pod`
```
- K8S 의 가장 작은 배포 단위이다.
- 컨테이너 관리의 다양화를 위해 Container 의 직접 관리가 아닌 Pod 라는 논리 그룹으로 관리한다.
```

> ### K8S-Intro-Term-Component

K8S 의 구성 요소에 대해 알아보자

ControlPlane 역할의 노드 : `kube-apiserver`, `kube-scheduler`, `kube-controller-manager`, `etcd`
모든 노드 : `kubelet` 과 `kube-proxy`

> ### K8S-Intro-Term-Component-ControlPlane

`kube-apiserver`

```bash
kubectl get pods -A | grep kube-apiserver
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

`kube-scheduler`

```
- Pod 스케줄링
```

```bash
sudo cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

`kube-controller-manager`

```
- 컨트롤러 프로세스를 실행하는 플레인 구성 요소로 각 컨트롤러는 별도의 프로세스이지만 단일 프로세스에서 실행된다.
```

```bash
sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```

`etcd`

```bash
sudo cat /etc/kubernetes/manifests/etcd.yaml
```

> ### K8S-Intro-Term-Component-Node

`kubelet`

클러스트의 각 노드에서 실행되는 에이전트로 컨테이너가 Pod에서 실행되고 있는지 확인한다.

`kube-proxy`

클러스터의 각 노드에서 실행되는 네트워크 프록시로 k8s 서비스 개념의 일부를 구현한다.

> ### K8S-Intro-Term-Component-Controller

Types of Controllers
```bash
- ReplicaSet
- Deployment
- DaemonSet
- StatefulSet
- Job
- CronJob
```

> ### K8S-Intro-Term-Component-Resource

K8S 에서 지원하는 모든 Resource 타입을 출력한다.
```bash
kubectl api-resources
## NAME / SHORTNAMES / APIVERSION / NAMESPACED / KIND
## NAMESPACED: Scope 개념
## NAMESPACED: true -> 네임스페이스 별로 구분
## NAMESPACED: false -> 네임스페이스 별로 미구분
```

> # K8S-Kubectl

> ### K8S-Kubectl-Intro-Definition

K8S 클러스터를 제어하는 명령어의 형식은 `명령 방식` 과 `선언 방식` 이 있다.
2가지 방식 모두 사용 가능하며, 처음 사용자의 경우 명령 방식에서 선언 방식으로 변경하는 옵션 **(--dry-run=client)** 사용을 권장한다.

> ### K8S-Kubectl-Type-Intro-Imperative

명령 방식
```bash
kubectl run hello --image=nginx --dry-run=client -o yaml | kubectl apply -f -
## Pod name: hello
## Pod Image: nginx
## --dry-run=client:
kubectl delete pod hello
```

> ### K8S-Kubectl-Type-Intro-Declarative

선언 방식
```bash
kubectl run hello --image=nginx --dry-run=client -o yaml | tee ~/test.yml
# apiVersion: v1
# kind: Pod
# metadata:
#   creationTimestamp: null
#   labels:
#     run: hello
#   name: hello
# spec:
#   containers:
#   - image: nginx
#     name: hello
#     resources: {}
#   dnsPolicy: ClusterFirst
#   restartPolicy: Always
# status: {}
kubectl apply -f ~/test.yml
kubectl delete -f ~/test.yml
```

> ### K8S-Kubectl-Cmd-explain

```bash
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers
kubectl explain pod.spec.containers.name
kubectl explain pod.spec.containers.image
kubectl explain pod --recursive | grep -A6 -B6 -i runasuser
```

> ### K8S-Kubectl-Cmd-get

```bash
kubectl get all
## 모든 Resource 조회
```

> ### K8S-Kubectl-Cmd-create

기 생성된 자원과 동일한 이름으로 자원 생성 시 중복 에러가 발생한다.
```bash
kubectl run hello --image=nginx --dry-run=client -o yaml | kubectl create -f -
kubectl run hello --image=nginx --dry-run=client -o yaml | kubectl create -f -
## Error from server (AlreadyExists): error when creating "STDIN": pods "hello" already exists
```

> ### K8S-Kubectl-Cmd-apply

기 생성된 자원과 동일한 이름으로 자원 생성 시 중복 에러 발생이 아닌 변경사항을 **`업데이트`** 한다.
kubectl edit 와 같이 변경 가능한 값은 제한된다.
```bash
kubectl run hello --image=nginx --dry-run=client -o yaml | kubectl apply -f -
## pod/hello created
kubectl run hello --image=nginx --dry-run=client -o yaml | kubectl apply -f -
## pod/hello configured
kubectl run hello --image=nginx --dry-run=client -o yaml | kubectl apply -f -
## restartPolicy: Never
## * spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds`, `spec.tolerations` (only additions to existing tolerations) or `spec.terminationGracePeriodSeconds` (allow it to be set to 1 if it was previously negative)
```

> ### K8S-Kubectl-Cmd-edit

기생성된 Resource 를 변경할 수 있다. 단, 변경 가능한 값은 제한된다.
```bash
kubectl run hello --image=nginx --dry-run=client -o yaml | kubectl apply -f -
kubectl edit pod hello
kubectl edit pod hello
## image: httpd
## 자동으로 Pod 삭제 후 재생성된다.
kubectl edit pod hello
## restartPolicy: Never
## * spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds`, `spec.tolerations` (only additions to existing tolerations) or `spec.terminationGracePeriodSeconds` (allow it to be set to 1 if it was previously negative)
```

> # K8S-Resource-Namespace

> ### K8S-Resource-Namespace-Oper-Feature

K8S 네임스페이스와 Linux 네임스페이스는 동음이의어이다.
K8S 네임스페이스는 **`논리 그룹`** 이며, 해당 그룹에서 Quota 등을 설정 할 수 있다.
```bash
kubectl create namespace test
sudo lsns | grep test
## lsns: Linux Namespace 조회 명령어
## 목록이 미표시된다.
```

> ### K8S-Resource-Namespace-Cmd-explain

```bash
kubectl explain namespace
```

> ### K8S-Resource-Namespace-Cmd-create

```bash
kubectl create namespace test --dry-run=client -o yaml
kubectl create namespace test
```

> ### K8S-Resource-Namespace-Cmd-get

```bash
kubectl get namespaces
kubectl get namespaces -n default
kubectl get namespace test
kubectl get namespace test -o yaml
```

> ### K8S-Resource-Namespace-Cmd-describe

```bash
kubectl describe namespace test
## Name:         test
## Labels:       kubernetes.io/metadata.name=test
## nnotations:  <none>
## Status:       Active

## No resource quota.

## No LimitRange resource.

```

> ### K8S-Resource-Namespace-Cmd-delete

```bash
kubectl delete namespace test
```

> # K8S-Resource-Pod

> ### K8S-Resource-Pod-Intro-Definition

K8S 에서 관리하는 논리 그룹(정보는 ETCD 에 저장)으로 하나의 Pod 는 2개의 컨테이너로 구성된다.
(하나는 사용자가 생성한 컨테이너, 나머지 하나는 K8S 에서 컨테이너 관리를 위한 Pause 컨테이너)

> ### K8S-Resource-Pod-Oper-Feature-Domain

```bash
kubectl run hello --image=nginx --dry-run=client -o yaml | kubectl apply -f -
kubectl get pods -o wide
## 192.168.153.244
curl 244-153-168-192.default.pod.cluster.local
## could not be resoved... why?
kubectl get svc -n kube-system
## kube-dns: 10.96.0.10
dig @10.96.0.10 192-168-153-244.default.pod.cluster.local
## ;; ANSWER SECTION:
## 192-168-153-244.default.pod.cluster.local. 30 IN A 192.168.153.244
kubectl run -it --rm --restart=Never curl --image=curlimages/curl sh
## --rm: 터미널 로그아웃 후 Pod 삭제
cat /etc/resolv.conf
## nameserver 10.96.0.10
## search default.svc.cluster.local svc.cluster.local cluster.local
curl 192-168-153-244.default.pod.cluster.local
```

K8S Cluster 에 생성되는 Resource Pod 는 모두 Domain Name 을 가지고 있다. 다만 해당 Domain Name 에 IP 가 포함되므로 향후 이중화 구성에서 해당 Endpoint 사용은 어렵다.
-> **Resource Service**
```bash
# Pod Domain Name
dig @10.96.0.10 192-168-153-244.default.pod.cluster.local
# Service Domain Name
dig @10.96.0.10 service_name-svc.default.svc.cluster.local
```
> ### K8S-Resource-Pod-Cmd-explain

```bash
kubectl explain pod
```

> ### K8S-Resource-Pod-Cmd-create

```bash
kubectl run hello --image=nginx
kubectl run hello --image=nginx --dry-run=client -o yaml
```

> ### K8S-Resource-Pod-Cmd-get

```bash
kubectl get pods
kubectl get pods -n default
kubectl get pod hello
kubectl get pod hello -o yaml
```

> ### K8S-Resource-Pod-Cmd-describe

```bash
kubectl describe pod hello
```

> ### K8S-Resource-Pod-Cmd-delete

```bash
kubectl delete pod hello
```

> # K8S-Resource-Replicaset
Pod 를 여러 개 배포하고 유지하는 경우 사용된다.

> ### K8S-Resource-Replicaset-Cmd-explain

```bash
kubectl explain replicaset
```

> ### K8S-Resource-Replicaset-Cmd-create

```bash
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: hello-rs
  labels:
    app: hello-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-pod
  template:
    metadata:
      labels:
        app: hello-pod
    spec:
      containers:
      - name: hello-container
        image: nginx
EOF
```

> ### K8S-Resource-Replicaset-Cmd-get

```bash
kubectl get replicaset
kubectl get replicaset -n default
kubectl get replicaset replicaset_name
kubectl get replicaset replicaset_name -o yaml
```

> ### K8S-Resource-Replicaset-Cmd-describe

```bash
kubectl describe replicaset replicaset_name
```

> ### K8S-Resource-Replicaset-Cmd-delete

```bash
kubectl delete replicaset replicaset_name
```

> ### K8S-Resource-Deployment

> ### K8S-Resource-Deployment-Intro-Definition

Pod 와 ReplicaSet 을 관리하는 컨트롤러

> ### K8S-Resource-Deployment-Cmd-explain

```bash
kubectl explain deploy

kubectl explain deploy.spec.selector

kubectl explain deploy.spec.strategy
```

> ### K8S-Resource-Deployment-Cmd-create

```bash
kubectl create deployment nginx --image=nginx --replicas=3

kubectl get pods -o wide
```

> ### K8S-Resource-Deployment-Cmd-edit

```bash
kubectl edit pods nginx-85b98978db-fgvnn
# labels:
#   app: nginx -> app: nginx2 변경
## 별도의 라벨을 명시하지 않는 경우 app 이 기본 key 값이다.

kubectl get pods
## Pod 개수가 4개가 되었다. Deployment 의 selector 와 Pod 의 label 이 일치하지 않는 경우 더 이상 ReplicaSet 으로 관리되지 않는다.
```

> ### K8S-Resource-Deployment-Cmd-edit-rollout

```bash
kubectl create deployment nginx --image=nginx --replicas=3

kubectl describe deployments.apps nginx | grep Rolling
## 배포 전략: RollingUpdate

kubectl edit deployments.apps nginx
## image: nginx -> image: httpd 변경

kubectl get replicasets.apps
## ReplicaSet 이 추가되며, 추가된 ReplicaSet 에 Pod 가 1개씩 생성된다.

kubectl rollout status deployment nginx

kubectl get pod -o wide
```

> ### K8S-Resource-Deployment-Cmd-get

```bash
kubectl get deployments

kubectl get replicaset

kubectl get pods

kubectl get pods -L app

kubectl get pods -l=app=nginx
```

> ### K8S-Resource-Deployment-Cmd-describe

```bash
kubectl describe deployments.apps deployment_name
```

> ### K8S-Resource-Deployment-Cmd-delete

```bash
kubectl delete deployments.apps deployment_name
```

> ### K8S-Resource-Deployment-Cmd-scale

ReplicaSet 의 관리 Pod 개수를 조절한다. (scale in/out)

```bash
kubectl create deployment nginx --image=nginx --replicas=3

kubectl scale deployment nginx --replicas=10
```

> ### K8S-Resource-Deployment-Cmd-set-image

```bash
kubectl create deployment nginx --image=nginx --replicas=6

kubectl set image deployment nginx nginx=httpd
## kubectl set image deployment deploy_name container_name=new_container_image

kubectl describe deployments.apps nginx | grep Image
## Image:        httpd
```

> ### K8S-Resource-Deployment-Cmd-rollout-status

```bash
kubectl create deployment nginx --image=nginx --replicas=6

kubectl rollout status deployment nginx
```

> ### K8S-Resource-Deployment-Cmd-rollout-history

```bash
kubectl create deployment nginx --image=nginx --replicas=6

kubectl rollout status deployment nginx
# Waiting for deployment "nginx" rollout to finish: 0 of 6 updated replicas are available...
## 배포 상태에 대해 실시간으로 출력한다.

kubectl rollout history deployment nginx

kubectl set image deployment nginx nginx=httpd
## nginx 디플로이 컨테이너 이미지를 nginx -> httpd 변경

kubectl get replicasets.apps
## 새로운 replicaset 이 생성되면서 하나씩 생성되고 삭제된다. -> RollingUpdate

kubectl describe deploy nginx | grep Rolling
# StrategyType:           RollingUpdate
# RollingUpdateStrategy:  25% max unavailable, 25% max surge

kubectl rollout history deployment nginx

kubectl rollout history deployment nginx
# REVISION  CHANGE-CAUSE
# 1         <none>
# 2         <none>
## 간단한 배포 버전 현황 출력

kubectl rollout history deployment nginx --revision=1
kubectl rollout history deployment nginx --revision=2
## 각 버전의 변경 사항 출력
## nginx -> httpd 변경을 확인할 수 있다.
```

> ### K8S-Resource-Deployment-Cmd-rollout-history-record

`deprecated`

```bash
kubectl create deployment nginx --image=nginx --replicas=6

kubectl rollout history deployment nginx

kubectl set image deployment nginx nginx=httpd --record
## Flag --record has been deprecated, --record will be removed in the future

kubectl rollout history deployment nginx
## [CHANGE-CAUSE] 항목에 변경을 야기한 명령어 구문이 기록된다.
```

> ### K8S-Resource-Deployment-Cmd-rollout-undo

처음 생성했던 deployment의 image로 rollback한다.

```bash
kubectl create deployment nginx --image=nginx --replicas=6

kubectl set image deploy nginx nginx=httpd

kubectl rollout status deploy nginx

kubectl describe deploy nginx | grep Image
## Image:        httpd

kubectl rollout undo deployment nginx

kubectl describe deploy nginx | grep Image
## Image:        nginx
```

> ### K8S-Resource-Deployment-Cmd-rollout-pause

최초 Deployment 생성 간에는 pause 명령어가 적용되지 않는다. 즉, 자원이 정상 생성된다.
다만, 해당 pause 명령어를 미리 입력하는 경우 다음 업데이트가 발생하는 시점(=set image)에서 Pod 가 생성되지 않는다.

```bash
kubectl create deployment nginx --image=nginx --replicas=6

kubectl rollout pause deployment nginx

kubectl rollout status deployment nginx
## Pod 가 정상 실행된다.

kubectl set image deploy nginx nginx=httpd

kubectl rollout status deployment nginx
## 위에서 미리 입력한 pause 명령어로 인해 RollingUpdate 가 pause 된다.
## Waiting for deployment "nginx" rollout to finish: 0 out of 6 new replicas have been updated...

kubectl rollout resume deployment nginx
## 배포 작업이 다시 재개된다.
```
