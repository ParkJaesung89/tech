# docker swarm 설치

```bash
# 선행
vi /etc/sysctl.conf
net.ipv4.ip_forward=1

# 1) swarm mode cluster 구축 
docker swarm init --advertise-addr 10.30.97.56

# 2) node 생성
#    (1) manager node 생성(node01)
 #     - swarm init을 수행 곳이 manager node가 됩니다.
docker swarm join-token manager

#    (2) worker-node 생성 (Node1 입력)
 docker swarm join-token manager

# ex) 
#To add a manager to this swarm, run the following command:
#    docker swarm join --token SWMTKN-1-18lgfc5nfdksfmvbybbwofg2sfdt65lkcxrhvi9dezsk0iw3ox-8y6m1iu5sx5a764sp3onhjea6 10.30.97.56:2377

# 발급받은 토큰으로 가입 Worker Node 가입 (node2,3) 
docker swarm join --token SWMTKN-1-18lgfc5nfdksfmvbybbwofg2sfdt65lkcxrhvi9dezsk0iw3ox-8y6m1iu5sx5a764sp3onhjea6 10.30.97.56:2377

# - 노드 상태  및 네트웍 리스트 확인
docker node ls
docker network ls

 ## 나중에 token 기억 안날 때는 docker swarm join-token worker 로 확인


```

