# Glusterfs 설치

``` bash
# 1. Install Gluster FS Server 
 # 모든 노드에서 다음과 같이 설치 진행 및 서비스 자동 시작 등록

 apt install software-properties-common -y
 wget -O- https://download.gluster.org/pub/gluster/glusterfs/10/rsa.pub | apt-key add -
 add-apt-repository ppa:gluster/glusterfs-10
 apt install glusterfs-server -y

 systemctl start glusterd
 systemctl enable glusterd
 systemctl status glusterd

 glusterfsd --version


# 2. 다음 명령어를 통하여 zabbix-node01에서 각 노드를 trustpool(클러스터)에 추가한다. 
gluster peer probe zabbix-node02
gluster peer probe zabbix-node03

# trustpool 추가 확인(각 Node의 Connect 확인)
gluster peer status

# Configure Gluster FS Servers
#  분산 스토리지 셋팅
 gluster volume create gfs-swarm replica 3 transport tcp zabbix-node01:/gfs-swarm zabbix-node02:/gfs-swarm zabbix-node03:/gfs-swarm force

  # 디렉터리를 생성 확인하고 볼륨 생성을 진행(Node1)
gluster volume start gfs-swarm

  # 볼륨 정보 확인
gluster volume info

# 4. Setup Gluster FS Client
# 각 노드에 볼륨을 마운트하기 위한 작업을 진행한다.

# 각 노드에서 진행
mount -t glusterfs zabbix-node01:/gfs-swarm /data
mount -t glusterfs zabbix-node02:/gfs-swarm /data
mount -t glusterfs zabbix-node03:/gfs-swarm /data

#부팅시 자동 마운트 되도록 추가(각 노드에)
vi /etc/fstab

zabbix-node01:/gfs-swarm /data glusterfs defaults,_netdev 0 0
zabbix-node02:/gfs-swarm /data glusterfs defaults,_netdev 0 0
zabbix-node03:/gfs-swarm /data glusterfs defaults,_netdev 0 0

## /data 폴더 내용이 3개 노드애서 동일하게 확인되어야함


```