# SPF 레코드 등록

http://begis2.siteprotect.co.kr/cgi-bin/slither/Driver.py/Begis2/Manager/SupportTools/DNSManager/ChooseDomain.render

위 주소인 베지스2에서 supportTools 에서 설정 가능

source  type    target
위 세가지 목록이 존재하며 해당 내용 작성필요

source에는 도메인 전체인지 서브도메인인지 등 작성
ex) icis.co.kr
    ftp
    mail
    pop
    sitemail
    smtp
    www
    .
    .
    .

type에는 레코드의 종류를 선택
ex) A
    CNAME
    MX
    NS
    TXT

Target에는 해당 주소를 적어야됨.
ex) SPF는 "v=spf1 ip4:66.232.138.51 ~all"

이런식으로 작성.


다 작성이 되었으면 add new record 를 눌러 추가를함
이미 등록된 내용을 수정할 경우 update를 누르면 적용됨.

레코드 확인법
$nslookup
$icis.co.kr

$set type=all
$set q=TXT
$icis.co.kr

등으로 확인가능.