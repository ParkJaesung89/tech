# proxy
프록시란
서버와 서버 사이의 중계 역할을 담당한다.
보안상 이유로 직접 통신할 수 없는 두 노드 사이에 위치하여 통신을 수행함.

[client] --- <proxy server> --- [internet and server]

## proxy 종류
1. forward proxy
클라이언트가 서버에 데이터를 요청하면 프록시 서버가 서버에 데이터를 받아서 제공
- > 서버에게 클라이언트가 누구인지 감추는 역할을 함

예)                           (클라이언트를 대신해서 프록시가 서버에 요청)
[client] --------->               --------------------------------> [server] 
[client] . . . . . [Forward proxy] . . . . . . <internet> . . . . . [server]
[client] <---------               <-------------------------------- [server]
                                    (서버는 데이터를 프록시에 전달)   
                                                          
- 클라이언트가 요청한 리소스를 원격 리소스에서 가지고와서 클라이언트에 전달하는 역할
- 만약 캐시가 존재하면 다음요청에는 캐시된 데이터를 제공하여 서버의 대역폭을 감소시킬 수 있다.


2. reverse proxy
클라이언트가 프록시 서버에 데이터를 요청하면 프록시 서버가 서버에 데이터를 받아서 제공
- > 클라이언트에게 서버가 누구인지 감추는 역할을 함

예)      (클라이언트가 리버스 프록시에 요청)
[client] -------------------------------->               ---------> [server] 
[client] . . . . . <internet> . . . . . . [reverse proxy] . . . . . [server]
[client]                                       (DMZ)                [server]
[client] <--------------------------------                <-------- [server]
    (프록시가 서버 대신 클라이언트에 데이터를 전달)                    (내부망)

- 내부 서버에서 직접 서비스를 제공하지 않고 보안상의 이유로 DMZ(내부, 외부 사이에 위치하는 구간)에 프록시 서버를 두어 외부 서비스를 제공한다.
- 보통 웹 서비스 구성시 DMZ에 web서버를 두고 내부망으로 was, db를 두어 구성시에 web서버를 리버스 프록시라한다.