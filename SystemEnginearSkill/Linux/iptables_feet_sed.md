# sed
sed는 파일에 내용 추가, 편집, 삭제가 가능한 리눅스 명령어이다.

sed '40s/^/$IPTABLES -A INPUT -p tcp -s 10.30.103.183 --dport 10050:10051 -j ACCEPT/' /etc/rc.d/rc.fw_220211
<!--
1. 40는 파일안에 줄넘버
2. s는 내용 수정
3. ^ 줄 맨 앞 / #는 줄 맨 뒤
4. 's/~/' 에서 ~는 변경할 내용
5. 마지막 's//' 뒤에 내용은 해당 명령이 수행 될 파일
-->

cat $IPTABLES -A INPUT -p TCP -j DROP >> /etc/rc.d/rc.fw 

<!--가장 아래에 내용 추가 하는게 sed로 넣기가 쉽지 않기 때문에 cat 명령으로 아래에 내용 추가-->

**
cat << EOF
작성된 내용을 파일에 저장할 수 있는 명령어(EOF는 임의의 이름으로 아무이름으로 대체가 가능함.)

예)
cat << EOF > file.txt
> hello
> world
> EOF

cat file.txt
hello
world

위와 같이 마지막에 EOF로 마무리 해준다.... 그럼 EOF안에 내용이 저장