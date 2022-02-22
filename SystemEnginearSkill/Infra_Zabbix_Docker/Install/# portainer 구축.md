# portainer 구축

```bash
# 1. portainer 컨테이너 설치에 앞서 컨테이너와 host(vm)간에 볼륨매칭을 위한 디렉터리 생성
mkdir -p /data/portainer

# 2.  docker run 명령어로 실행
## 여기서는 https://redmine.hostway.co.kr/issues/43287?issue_count=1&issue_position=1#note-18 에 있는 "portainer 설치" 를 참고하여 설치함
# 예)
docker run --name portainer -p 9000:9000 -d --restart always -v /data/portainer:/data -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer

#  설명) –-name 으로 컨테이너 이름 생성,
# -p 호스트 포트 9000 내부포트 9000번 ,
# -d 데몬으로 백그라운드,
# –restart always 재부팅시 자동시작,
# -v /data~~ 호스트와 컨테이너간 볼륨매칭,
# docker.sock도 마찬가지로 공유, 
#portainer/portainer 이미지 사용

# 3. 테스트
# VM 의 IP:9000 으로 웹브라우저로 접속합니다.
# http://ip:9000 입니다.

```