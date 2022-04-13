# Server Name Indication.
아파치 2.2.12이상 / 아파치 2.4.8 이상 버전에서는 Server Name Indication 를 지원한다.
Server Name Indication은 TLS를 이용한 핸드쉐이크의 확장 기능이라고 보면 되겠다.
몇몇 브라우져 및 기기의 접속 제한이 있지만 이를 무시할 수 있을경우 하나의 서버에서
여러사이트의 SSL(https)를 단일 443 포트로 연결할 수 있다.

 

SNI를 설정할때는 먼저 서버명의 443포트를 먼저 선언해야 고객들의 혼란을 방지 할수 있다.
아파치의 경우 커넥션이 웹서버에 도달했을때 매칭이 되는 가상호스트가 없을경우 최상위에
선언되 버츄얼호스트로 연결한다.( nginx 는 반대로 마지막에 선언된 버츄얼호스트로 작동한다 )

서버 버전이 CentOS 6.5 미만에서는 ca-certificates / nss 업데이트가 필요할 수 있다.

#yum update ca-certificates nss
아파치 2.4의 경우 버전을 만족한다면 단순히 443포트로 선언된 여러 가상호스트를 추가하는것만으로 세팅이 완료가 된다.

예)
<VirtualHost *:443>
DocumentRoot /home/korsys00t/html
ServerName wp01.nayana.kr
SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/wp01.nayana.kr/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/wp01.nayana.kr/privkey.pem
</VirtualHost>
 
<VirtualHost *:443>
DocumentRoot /free/home/enteroa/html
ServerName www.enteroa.kr
ServerAlias enteroa.kr www.enteroa.kr wp.enteroa.kr
SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/enteroa.kr/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/enteroa.kr/privkey.pem
</VirtualHost>
 

아파치 2.2 버전은 NameVirtualHost 를 선언해야 한다.

예)
NameVirtualHost *:443
<VirtualHost *:443>
DocumentRoot /home/korsys00t/html
ServerName wp01.nayana.kr
SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/wp01.nayana.kr/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/wp01.nayana.kr/privkey.pem
</VirtualHost>
 
<VirtualHost *:443>
DocumentRoot /free/home/enteroa/html
ServerName www.enteroa.kr
ServerAlias enteroa.kr www.enteroa.kr wp.enteroa.kr
SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/enteroa.kr/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/enteroa.kr/privkey.pem
</VirtualHost>


위와 같은 세팅만으로 여러 사이트가 443 기본 포트로 ssl 이용이 가능하다.
단  아래 조합은 브라우져에서 SNI 접속 기능이 없기 때문에 접속이 불가능 하다.
Windows XP – 인터넷 익스플로어(모든버전), 사파리
Android 2.3.7(진저브레드)을 포함한 이전 버전
BlackBerry 7.1을 포함한 이전 버전
WindowsMobile 6.5을 포함한 이전 버전
https://en.wikipedia.org/wiki/Server_Name_Indication

***SNI 는 nginx 0.8.21 이상 / IIS 8.0 이상 역시 지원 한다.***