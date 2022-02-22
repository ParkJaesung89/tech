# Docker command
## Docker stack Deploy
```bash
docker stack deploy -c zabbix.yml [stack name]
```

## Docker swarm node control
```bash
docker node ls
docker node demote [worker node]   # 현재 리더인 노드 제거 (node02 / node03 제거)
docker node promote [worker node]  # 제거된 노드 재가입
```

## DISK stat Test
```bash
dd if=/dev/zero bs=1G count=100 of=write_1GB_test oflag=direct # 쓰기 (1GB *100 파일 생성)
dd if=/dev/zero bs=1M count=61440 of=write_1GB_test oflag=direct 
# 쓰기 (1MB *61440 파일 생성)
dd if=write_1GB_test of=/dev/null bs=1024 # (1byte 단위로 파일 읽기)
```