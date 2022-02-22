# Docker 설치

```bash
#[Ubuntu 20.04 LTS 에 Docker 설치]

# 1. 먼저 apt update & apt upgrade 해줍니다.

# 패키지 설치
 apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common

# GPG Key 인증
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
#       ok가 나와야함

# 2. docker repository 등록
#아키텍쳐에(x86-64) 맞춰서 Docker repository를 등록

 add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) \
stable"

# 3.  docker 설치
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io

#  완료되면 docker -v 로 확인

# 4.  시스템 부팅시 docker가 시작되도록 설정 하고 상태 확인
 systemctl enable docker
 systemctl start docker
 systemctl status docker

```




