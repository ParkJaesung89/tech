# gluster+docker+zabbix 3노드 구성

## 구성 전 환경
플클 CentOS7 - server 노드 3개

### /etc/hostname
1. master
2. slave1
3. slave2

### /etc/hosts
10.131.234.11 master
10.131.234.12 slave1
10.131.234.13 slave2

### ssh 키 교환
```
#ssh-keygen
#ssh-copy-id root@master
#ssh-copy-id root@slave1
#ssh-copy-id root@slave2
#ssh master
#ssh slave1
#ssh slave2
```

### 분산스토리지로 사용할 디스크 볼륨 마운트
```
#mkdir -p /gluster

#vi /etc/fstab
/dev/sdb1       /gluster                ext4    defaults        0  0

#mount -a
```

## GlusterFS 분산 스토리지 구성

### 3노드 각각 디스크 1개 추가
```
#fdisk /dev/sdb
n - - - w           // 전체 디스크 통으로 잡음 파티션분할x
#mkfs.ext4 /dev/sdb1
```

### 분산 스토리지 구성

#### glusterfs 설치 및 실행
```
#yum install -y centos-release-gluster           //yum install -y glusterfs-* 가능하면 이걸로
#yum install -y glusterfs-server                 //glusterfs 9.4  
#yum install -y glusterfs-fuse
#yum install -y glusterfs-client
#systemctl start glusterd  
#systemctl enable glusterd  
```

#### gluster volume 생성
```
#mkdir -p /gluster/brick1                        //각노드마다 brick1,2,3  
#gluster volume create gfsvol1 rep 3 transport tcp master:/gluster/brick1/ slave1:/gluster/brick2 slave2:/gluster/brick3 force  
#gluster volume start gfsvol1
```

#### gluster volume 마운트
```
#mkdir -p /data
#mount -t glusterfs master:/gfsvol1 /data/

[root@master data]# df -h
Filesystem       Size  Used Avail Use% Mounted on
/dev/sda1         20G  1.6G   19G   8% /
devtmpfs         1.9G     0  1.9G   0% /dev
tmpfs            1.9G     0  1.9G   0% /dev/shm
tmpfs            1.9G  8.7M  1.9G   1% /run
tmpfs            1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda2         80G   33M   80G   1% /home
tmpfs            379M     0  379M   0% /run/user/0
/dev/sdb1         98G   62M   93G   1% /gluster
master:/gfsvol1   98G  1.1G   93G   2% /data
```

#### 스토리지 공유 테스트
```
#echo 'hello' >> test.txt
#cat /data/test.txt
```
- > 각 노드에서 test.txt 파일 내용 'hello'가 출력되면 정상


## 도커

### 도커 데몬 설치
```
1)yum pakage 설치(안정적 버전)
 #sudo yum install -y yum-utils
 #sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

 #yum install docker-ce docker-ce-cli containerd.io
 #yum list docker-ce --showduplicates | sort -r

2)스크립트로 설치(안정적 버전)
 #curl -fsSL https://get.docker.com -o get-docker.sh
 #DRY_RUN=1 sh ./get-docker.sh

#systemctl start docker
#systemctl enable docker
```

### docker compose 설치
```
# curl -L "https://github.com/docker/compose/releases/download/v2.1.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
or
# curl -L "https://github.com/docker/compose/releases/download/v2.1.0/docker-compose-linux-x86_64 quot;" -o /usr/local/bin/docker-compose

#chmod +x /usr/local/bin/docker-compose
#docker-compose --version
Docker Compose version v2.1.0
```

### docker swarm 구성
1. swarm mode cluster 구성
    <master>
    #docker swarm init --advertise-addr 10.131.234.11
    ```
       Swarm initialized: current node (nzvwpeq284crdzk0pdp8lgcy5) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join --token SWMTKN-1-49y2hap0ljoq5plm7occ0ew8x6y7a6puvhvyu3h2ipld6gr7br-5flfstl88xnl8obvqfpbes22n 10.131.234.11:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```
    
2. swarm node 생성
    <master>
    #docker swarm join-token manager
    ```
    To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-49y2hap0ljoq5plm7occ0ew8x6y7a6puvhvyu3h2ipld6gr7br-2xnexhigzqi0l5onxlklmoe11 10.131.234.11:2377
    ```
    
    <slave1,2>
    #docker swarm join --token SWMTKN-1-49y2hap0ljoq5plm7occ0ew8x6y7a6puvhvyu3h2ipld6gr7br-2xnexhigzqi0l5onxlklmoe11 10.131.234.11:2377
    ```
    This node joined a swarm as a manager.
    ```

    <master>                <!-- swarm 구성 후 상태 확인 -->
    #docker node ls
    ```
    ID                            HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
    nzvwpeq284crdzk0pdp8lgcy5 *   master     Ready     Active         Leader           20.10.11
    5zd90s6idzqez956m1jkkr4is     slave1     Ready     Active         Reachable        20.10.11
    o9q01qn0bl7tc0wmtv3zov5ab     slave2     Ready     Active         Reachable        20.10.11
    ```

    #docker network ls
    ```
    NETWORK ID     NAME              DRIVER    SCOPE
    9558134cc04a   bridge            bridge    local
    8cec63234d88   docker_gwbridge   bridge    local
    6bba687ab6b2   host              host      local
    ecszxn2wjzy8   ingress           overlay   swarm
    e2fe02d4c619   none              null      local
    ```

### Portainer 설치
1. portainer라는 docker stack을 구성하는 yml 파일 설치  
    <master>
    #mkdir -p /data/portainer/data  
    #cd /data/portainer
    #curl -L https://downloads.portainer.io/portainer-agent-stack.yml     -o     portainer-agent-stack.yml   
    #vi portainer-agent-stack.yml
    ```
    version: '3.2'

    services:
      agent:
        image: portainer/agent:2.11.0
        volumes:
         - /var/run/docker.sock:/var/run/docker.sock
         - /var/lib/docker/volumes:/var/lib/docker/volumes
        networks:
         - agent_network
        deploy:
            mode: global
            placement:
                constraints: [node.platform.os == linux]

      portainer:
        image: portainer/portainer-ce:2.11.0
        command: -H tcp://tasks.agent:9001 --tlsskipverify
        ports:
         - "9000:9000"
         - "8000:8000"
        volumes:
         - /data/portainer/data:/data
        networks:
         - agent_network
        deploy:
            mode: replicated  
            replicas: 1
            placement:
                constraints: [node.role == manager]

    networks:
      agent_network:
        driver: overlay
        attachable: true
    ```

    #cd /data/portainer  
    #docker stack deploy -c portainer-agent-stack.yml portainer

2. Portainer 서비스 확인
   #docker ps -a
   #docker stack ls
   #docker stack services portainer
   #docker stack ps portainer

   - > http://주소:9000 으로 접속하여 portainer 페이지가 각 노드의 ip로 접속되는지 확인
            http://210.122.45.207:9000  
            http://210.122.45.208:9000  
            http://210.122.45.237:9000  


### zabbix 설치 <!-- 설치 및 정리 할 필요가 잇음 아직 진행안함 -->
# mkdir -p /data/zabbix/
\\fileserver01\HOSTWAYKOREA\HOSTWAYKOREA\HOSTWAYKOREA-시스템운영팀\IDC\HOSTWAY\HOSTWAY IDC - MSP\현대차\zbx-docker.tar.gz
파일 업로드

# tar zxvf zbx-docker.tar.gz
# cd zabbix-docker/
# docker stack deploy -c zabbix.yml zabbix
Creating network zabbix_internal_zbx_network
Creating network zabbix_public_zbx_network
Creating secret zabbix_MYSQL_ROOT_PASSWORD
Creating secret zabbix_MYSQL_USER
Creating secret zabbix_MYSQL_PASSWORD
Creating service zabbix_zabbix-server
Creating service zabbix_mysql-server
Creating service zabbix_zabbix-agent
Creating service zabbix_zabbix-proxy-mysql
Creating service zabbix_zabbix-snmptraps
Creating service zabbix_zabbix-java-gateway
Creating service zabbix_zabbix-nginx