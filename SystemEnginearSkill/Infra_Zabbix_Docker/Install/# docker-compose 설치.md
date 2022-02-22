# docker-compose 설치
``` bash
# - 아래와 같이 설치 되어야함
curl -L "https://github.com/docker/compose/releases/download/v2.1.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# or
curl -L "https://github.com/docker/compose/releases/download/v2.1.0/docker-compose-linux-x86_64 quot;" -o /usr/local/bin/docker-compose


# 파일 권한수정
 chmod +x /usr/local/bin/docker-compose

# 버전 확인
docker-compose --version

```

