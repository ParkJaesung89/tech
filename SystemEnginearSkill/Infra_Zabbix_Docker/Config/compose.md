# zabbix compose 

## Compose 파일 위치

```bash
vi /data/zabbix/zabbix.yml
```

## compose 파일 내용

```bash
version: '3.7'
services:
 zabbix-nginx:
  image: zabbix/zabbix-web-nginx-mysql:ubuntu-5.4-latest
  user: root
  ports:
   - "80:8080"
   - "443:8443"
  volumes:
   - /etc/localtime:/etc/localtime:ro
   - /etc/timezone:/etc/timezone:ro
   - ./volumes/zabbix_nginx/assets/:/usr/share/zabbix/assets/
   - ./volumes/zabbix_nginx/modules/:/usr/share/zabbix/modules/
   - ./volumes/zabbix_nginx/ssl/:/etc/ssl/
   - ./volumes/zabbix_nginx/nginx/:/etc/zabbix/
  env_file:
   - .env_db_mysql
   - .env_web
  secrets:
   - MYSQL_USER
   - MYSQL_PASSWORD
  depends_on:
   - mysql-server
   - zabbix-server
  healthcheck:
   test: ["CMD", "curl", "-f", "http://localhost:8080/"]
   interval: 10s
   timeout: 5s
   retries: 3
   start_period: 30s
  networks:
   public_zbx_network:
   internal_zbx_network:
    aliases:
     - zabbix-nginx
     - zabbix-web-nginx-mysql
     - zabbix-web-nginx-ubuntu-mysql
     - zabbix-web-nginx-mysql-ubuntu
  stop_grace_period: 10s
  deploy:
   update_config:
    failure_action: rollback
   replicas: 3
  labels:
   com.zabbix.description: "Zabbix frontend on Apache web-server with MySQL database support"
   com.zabbix.company: "Zabbix LLC"
   com.zabbix.component: "zabbix-frontend"
   com.zabbix.webserver: "apache2"
   com.zabbix.dbtype: "mysql"
   com.zabbix.os: "ubuntu"

 zabbix-server:
  image: zabbix/zabbix-server-mysql:ubuntu-5.4-latest
  user: root
  ports:
   - "10051:10051"
  volumes:
   - /etc/localtime:/etc/localtime:ro
   - /etc/timezone:/etc/timezone:ro
   - ./volumes/zabbix/usr/lib/zabbix/alertscripts:/usr/lib/zabbix/alertscripts:rw
   - ./volumes/zabbix/usr/lib/zabbix/externalscripts:/usr/lib/zabbix/externalscripts:rw
   - ./volumes/zabbix_modules:/var/lib/zabbix/modules:ro
   - ./volumes/zabbix_enc:/var/lib/zabbix/enc:rw
   - ./volumes/zabbix_ssh_keys:/var/lib/zabbix/ssh_keys:ro
   - ./volumes/zabbix_mibs:/var/lib/zabbix/mibs:ro
   - ./volumes/snmptraps:/var/lib/zabbix/snmptraps:rw
   - ./volumes/zabbix_export:/var/lib/zabbix/export
   - ./volumes/zabbix/zabbix-server:/etc/zabbix:rw
  deploy:
   update_config:
    failure_action: rollback
   placement:
    constraints:
            #     - 'node.hostname == zabbix-node01'
     - 'node.role == manager'
  env_file:
   - .env_db_mysql
   - .env_srv
  secrets:
   - MYSQL_USER
   - MYSQL_PASSWORD
   - MYSQL_ROOT_PASSWORD
  depends_on:
   - mysql-server
  networks:
   internal_zbx_network:
    aliases:
     - zabbix-server
     - zabbix-server-mysql
     - zabbix-server-ubuntu-mysql
     - zabbix-server-mysql-ubuntu
   public_zbx_network:
  stop_grace_period: 30s
  sysctls:
   - net.ipv4.ip_local_port_range=1024 65530
   - net.ipv4.conf.all.accept_redirects=0
   - net.ipv4.conf.all.secure_redirects=0
   - net.ipv4.conf.all.send_redirects=0
  labels:
   com.zabbix.description: "Zabbix server with MySQL database support"
   com.zabbix.company: "Zabbix LLC"
   com.zabbix.component: "zabbix-server"
   com.zabbix.dbtype: "mysql"

 mysql-server:
  image: mariadb:10.5
  user: root
  command:
   - mysqld
   - --character-set-server=utf8mb4
   - --collation-server=utf8mb4_unicode_ci
   - --default-authentication-plugin=mysql_native_password
  volumes:
   - ./volumes/zabbix_db/mysql:/var/lib/mysql:rw
   - ./volumes/zabbix_db/mariadb.conf.d/:/etc/mysql/mariadb.conf.d/
  env_file:
   - .env_db_mysql
  secrets:
   - MYSQL_USER
   - MYSQL_PASSWORD
   - MYSQL_ROOT_PASSWORD
  stop_grace_period: 1m
  networks:
   internal_zbx_network:
    aliases:
     - mysql-server
     - zabbix-database
     - mysql-database
     - zabbix-mysql
  ulimits:
   memlock:
    soft: -1
    hard: -1
   nofile:
    soft: 65536
    hard: 65536
  deploy:
   update_config:
    failure_action: rollback
   placement:
    constraints:
     - 'node.role == manager'
       #     - 'node.hostname == zabbix-node02'

 zabbix-agent:
  image: zabbix/zabbix-agent2:ubuntu-5.4-latest
  user: root
  ports:
   - "10050:10050"
  volumes:
   - /etc/localtime:/etc/localtime:ro
   - /etc/timezone:/etc/timezone:ro
   - /var/run:/var/run
     #   - ./volumes/zabbix/zabbix_agentd.d:/etc/zabbix/zabbix_agentd.d:ro
   - ./volumes/zabbix_modules:/var/lib/zabbix/modules:ro
   - ./volumes/zabbix_enc:/var/lib/zabbix/enc:ro
   - ./volumes/zabbix_ssh_keys:/var/lib/zabbix/ssh_keys:ro
   - ./volumes/zabbix/usr/local/bin:/usr/local/bin:ro
   - ./volumes/zabbix/zabbix:/etc/zabbix:ro
     # - /proc:/proc:ro
   - /dev:/dev:ro
  deploy:
   mode: global
  env_file:
   - .env_agent
     #  privileged: true
  networks:
   internal_zbx_network:
    aliases:
     - zabbix-agent
     - zabbix-agent-passive
     - zabbix-agent-ubuntu
  stop_grace_period: 5s
  labels:
   com.zabbix.description: "Zabbix agent"
   com.zabbix.company: "Zabbix LLC"
   com.zabbix.component: "zabbix-agentd"
   com.zabbix.os: "ubuntu"

 zabbix-proxy-mysql:
  image: zabbix/zabbix-proxy-mysql:ubuntu-5.4-latest
  ports:
   - "10071:10051"
  volumes:
   - /etc/localtime:/etc/localtime:ro
   - ./volumes/zabbix/usr/lib/zabbix/externalscripts:/usr/lib/zabbix/externalscripts:ro
   - ./volumes/zabbix_modules:/var/lib/zabbix/modules:ro
   - ./volumes/zabbix_enc:/var/lib/zabbix/enc:ro
   - ./volumes/zabbix_ssh_keys:/var/lib/zabbix/ssh_keys:ro
   - ./volumes/zabbix_mibs:/var/lib/zabbix/mibs:ro
   - ./volumes/snmptraps:/var/lib/zabbix/snmptraps:rw
  ulimits:
   nproc: 65535
   nofile:
    soft: 20000
    hard: 40000
  deploy:
   resources:
    limits:
     cpus: '0.70'
     memory: 1G
  env_file:
   - .env_db_mysql_proxy
   - .env_prx
   - .env_prx_mysql
  depends_on:
   - mysql-server
  secrets:
   - MYSQL_USER
   - MYSQL_PASSWORD
   - MYSQL_ROOT_PASSWORD
  networks:
   internal_zbx_network:
    aliases:
     - zabbix-proxy-mysql
     - zabbix-proxy-ubuntu-mysql
     - zabbix-proxy-mysql-ubuntu
   public_zbx_network:
  stop_grace_period: 30s
  labels:
   com.zabbix.description: "Zabbix proxy with MySQL database support"
   com.zabbix.company: "Zabbix LLC"
   com.zabbix.component: "zabbix-proxy"
   com.zabbix.dbtype: "mysql"
   com.zabbix.os: "ubuntu"

 zabbix-snmptraps:
  image: zabbix/zabbix-snmptraps:ubuntu-5.4-latest
  ports:
   - "162:1162/udp"
  volumes:
   - ./volumes/snmptraps:/var/lib/zabbix/snmptraps:rw
  deploy:
   resources:
    limits:
      cpus: '0.5'
      memory: 256M
    reservations:
      cpus: '0.25'
      memory: 128M
  networks:
   public_zbx_network:
    aliases:
     - zabbix-snmptraps
   internal_zbx_network:
  stop_grace_period: 5s
  labels:
   com.zabbix.description: "Zabbix snmptraps"
   com.zabbix.company: "Zabbix LLC"
   com.zabbix.component: "snmptraps"
   com.zabbix.os: "ubuntu"

 zabbix-java-gateway:
  image: zabbix/zabbix-java-gateway:ubuntu-5.4-latest
  ports:
   - "10052:10052"
  deploy:
   resources:
    limits:
      cpus: '0.5'
      memory: 512M
    reservations:
      cpus: '0.25'
      memory: 256M
  env_file:
   - .env_java
  networks:
   internal_zbx_network:
    aliases:
     - zabbix-java-gateway
     - zabbix-java-gateway-ubuntu
  stop_grace_period: 5s
  labels:
   com.zabbix.description: "Zabbix Java Gateway"
   com.zabbix.company: "Zabbix LLC"
   com.zabbix.component: "java-gateway"
   com.zabbix.os: "ubuntu"

 grafana:
  image: grafana/grafana:8.0.4
  ports:
   - "3000:3000"
  volumes:
   - ./volumes/grafana-volumes:/var/lib/grafana
   - ./volumes/grafana_provisioning:/etc/grafana/provisioning
   - /etc/localtime:/etc/localtime:ro
   - /etc/timezone:/etc/timezone:ro
  environment:
   - .env_grafana
  networks:
   public_zbx_network:
   internal_zbx_network:
  user: root
  deploy:
   update_config:
    failure_action: rollback
   placement:
    constraints:
            #     - 'node.hostname == zabbix-node03'
     - 'node.role == manager'

 renderer:
  image: grafana/grafana-image-renderer:latest
  ports:
   - 8081
  networks:
   public_zbx_network:
   internal_zbx_network:

networks:
 public_zbx_network:
  driver: overlay
  driver_opts:
   com.docker.network.enable_ipv6: "false"
  attachable: true

 internal_zbx_network:
  driver: overlay
  attachable: true
  internal: true
  ipam:
   driver: default
   config:
    - subnet: "10.20.30.0/24"

secrets:
  MYSQL_USER:
    file: ./.MYSQL_USER
  MYSQL_PASSWORD:
    file: ./.MYSQL_PASSWORD
  MYSQL_ROOT_PASSWORD:
    file: ./.MYSQL_ROOT_PASSWORD

volumes:
 zabbix_agentd:

```

## zabbix agent 환경변수
``` bash
cat /data/zabbix/.env_agent

ZBX_SERVER_HOST=zabbix-server
ZBX_PASSIVE_ALLOW=true
ZBX_PASSIVESERVERS=0.0.0.0/0
ZBX_ACTIVE_ALLOW=true
ZBX_REFRESHACTIVECHECKS=60
ZBX_TIMEOUT=30
ZBX_UNSAFEUSERPARAMETERS=1

```

## mariadb 환경변수

```bash
cat /data/zabbix/.env_db_mysql

DB_SERVER_HOST=mysql-server
DB_SERVER_PORT=3306
MYSQL_USER_FILE=/run/secrets/MYSQL_USER
MYSQL_PASSWORD_FILE=/run/secrets/MYSQL_PASSWORD
MYSQL_ROOT_PASSWORD_FILE=/run/secrets/MYSQL_ROOT_PASSWORD
MYSQL_DATABASE=zabbix
TZ=Asia/Seoul

```

## zabbix proxy 환경변수

```bash
cat /data/zabbix/.env_db_mysql_proxy

MYSQL_USER_FILE=/run/secrets/MYSQL_USER
MYSQL_PASSWORD_FILE=/run/secrets/MYSQL_PASSWORD
MYSQL_ROOT_PASSWORD_FILE=/run/secrets/MYSQL_ROOT_PASSWORD
MYSQL_DATABASE=zabbix_proxy


cat /data/zabbix/.env_prx

ZBX_SERVER_HOST=zabbix-server
ZBX_SERVER_PORT=10051

```

## grafana 환경변수

```bash
cat /data/zabbix/.env_grafana

GF_INSTALL_PLUGINS=grafana-clock-panel,briangann-gauge-panel,natel-plotly-panel,grafana-simple-json-datasource,alexanderzobnin-zabbix-app
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=admin
GF_PATHS_CONFIG="/etc/grafana/grafana.ini"
GF_RENDERING_SERVER_URL: http://renderer:8081/render
GF_RENDERING_CALLBACK_URL: http://grafana:3000/
GF_LOG_FILTERS: rendering:debug

```

## zabbix-server 환경변수

```bash
cat /data/zabbix/.env_srv

ZBX_CACHESIZE=1024M
ZBX_CACHEUPDATEFREQUENCY=60
ZBX_STARTDBSYNCERS=8
ZBX_HISTORYCACHESIZE=1024M
ZBX_HISTORYINDEXCACHESIZE=16M
ZBX_TRENDCACHESIZE=1024M
ZBX_VALUECACHESIZE=4096M
ZBX_TIMEOUT=15
ZBX_HOUSEKEEPINGFREQUENCY=1
ZBX_MAXHOUSEKEEPERDELETE=3000
ZBX_ENABLE_SNMP_TRAPS=true
ZBX_STARTJAVAPOLLERS=5
ZBX_JAVAGATEWAY_ENABLE=true

```

## mysql 접근정보

```bash
cat /data/zabbix/.MYSQL_USER
zabbix

cat /data/zabbix/.MYSQL_PASSWORD
zabbix

cat /data/zabbix/.MYSQL_ROOT_PASSWORD
sjaksahffk.

```

## zabbix-server Config

```bash
cat /data/zabbix/volumes/zabbix/zabbix-server/zabbix_server.conf | grep -v "#" | grep -v ^$ 
# 주석과 공백 제거 후 출력
 SourceIP=0.0.0.0
LogType=console
DBHost=mysql-server
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix
DBPort=3306
 StartPollers=100
 StartTrappers=10
 StartPingers=50
 StartDiscoverers=10
 StartHTTPPollers=20
 StartTimers=10
 StartEscalators=10
JavaGateway=zabbix-java-gateway
StartJavaPollers=5
SNMPTrapperFile=/var/lib/zabbix/snmptraps/snmptraps.log
StartSNMPTrapper=1
HousekeepingFrequency=1
MaxHousekeeperDelete=3000
CacheSize=1024M
CacheUpdateFrequency=60
StartDBSyncers=8
HistoryCacheSize=1024M
HistoryIndexCacheSize=16M
TrendCacheSize=1024M
ValueCacheSize=4096M
Timeout=15
AlertScriptsPath=/usr/lib/zabbix/alertscripts
ExternalScripts=/usr/lib/zabbix/externalscripts
FpingLocation=/usr/bin/fping
Fping6Location=/usr/bin/fping6
SSHKeyLocation=/var/lib/zabbix/ssh_keys
AllowRoot=1
SSLCertLocation=/var/lib/zabbix/ssl/certs/
SSLKeyLocation=/var/lib/zabbix/ssl/keys/
SSLCALocation=/var/lib/zabbix/ssl/ssl_ca/
LoadModulePath=/var/lib/zabbix/modules/
WebServiceURL=http://zabbix-web-service:10053/report

```

## mariadb Config

```bash
cat /data/zabbix/volumes/zabbix_db/mariadb.conf.d/50-server.cnf  | grep -v "#" | grep -v ^$ 
# 주석과 공백 제거
[server]
[mysqld]
pid-file                = /run/mysqld/mysqld.pid
basedir                 = /usr
datadir                 = /var/lib/mysql
tmpdir                  = /tmp
lc-messages-dir         = /usr/share/mysql
lc-messages             = en_US
skip-external-locking
expire_logs_days        = 2
character-set-server  = utf8mb4
collation-server      = utf8mb4_general_ci
innodb_buffer_pool_size = 90G
[embedded]
[mariadb]
[mariadb-10.5]
innodb_thread_concurrency=7
open_files_limit                = 60000
table_open_cache                = 16384
table_definition_cache          = 8192
innodb_flush_log_at_trx_commit  = 2
innodb_buffer_pool_load_at_startup=ON
performance_schema = ON
innodb_log_file_size = 8G

```

