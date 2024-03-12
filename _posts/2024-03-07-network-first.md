---
title: Network first week
published: true
---


## <span style="color:#802548">_network_</span>
- network란 컴퓨터나 기기들이 데이터를 받기위해 유선/무선으로 연결된 통신 체계를 의미한다.
- 범위에 따른 분류로는 보통 LAN과 WAN, 공개여부에 따른 분류로는 인트라넷과 인터넷이 있다.
- LAN은 Ethernet과 wireless LAN이 중요한 기술인데, Ethernet은 유선 통신용이며, wifi는 무선 통신용이다.
- WAN은 여러 LAN을 묶은 네트워크다. 
- WAN은 은행의 ATM, wireless WAN, Internet 등이 있다.
- 인터넷 연결 서비스를 제공하는 사업자를 ISP로 부르며, 보통 통신사다.
- 서로 다른 통신사들끼리도 소통이 가능한 이유는 ISP의 네트워크끼리도 서로 연결되어 있기 때문이다.
- ISP도 tier가 나뉜다.
  - tier1은 국제 범위의 네트워크를 가지고 있다.
    - 인터넷의 모든 네트워크에 접근이 가능하다.
    - 모든 네트워크에 접근 가능하므로 backbone이라고 부른다.
    - tier1끼리는 트래픽 전송 비용이 없고, 하위 티어에게는 돈을 받는다.
  - tier2는 국가/지방 범위 네트워크다.
    - 일반사용자나 기업을 대상으로 서비스한다.
    - 인터넷의 모든 영역에 연결되기 위해 tier1 ISP에 비용을 내고 트래픽을 전송한다.
    - 최근의 망중립성과 관련된 부분이다.
  - tier3는 작은 지역 범위를 커버한다.
    - 일반사용자나 기업을 대상으로 서비스한다.
    - 상위 ISP에게 비용을 내고 트래픽을 구매한다.
    - 한국은 tier3가 거의 없다.
- ISP끼리는 복수개의 라우터를 통한 경로탐색을 통해서 서로의 IP로 이동한다.
- 네트워크의 끝에 있는 node를 end system, host라고 부른다.
- host는 네트워크를 사용하기 위해 연결된 노드다.
- end system은 client와 server로 나뉜다.
- client는 다른 host의 데이터나 리소스를 요청하는 host다.
- server는 다른 호스트에게 서비스를 제공하는 host다. 요청에 따라 데이터나 리소스를 제공한다.


## <span style="color:#802548">_인터넷의 발전_</span>
- 인터넷은 inter network라는 말과 같이, 네트워크를 모두 끌어 모은 것이다.
- 인터넷은 웹, 메일, FTP, 모바일, 스트리밍 등 다양한 프로토콜이 있다.
- 인터넷은 웹이 아니다. 웹은 인터넷 중의 일부다.
- 웹의 등장 배경은 tree-based file system의 한계 때문이다.
- 계층 구조의 파일 시스템은 연관성을 중심으로 정보를 검색하는데 있어 치명적인 결함이 있었기 때문이다.
- 그래서 링크를 통해 연관된 정보를 표현하게하는 Enquire가 개발된다. 그런데 Enquire는 같은 file system안에서만 작동했다.
- 문제의식은 발전으로 나아갔다. CERN에 맞게 발전시키려면 아래와 같은 조건이 있었다.
```
지리적으로 멀리 떨어진 대학에서 연구하는 팀도 있다. 네트워크를 통한 원격 접근이 가능해야 한다.
tree로는 안된다. 여러정보가 link를 통해서 연결되어야 한다. 즉 하이퍼텍스트여야 한다.
다양한 시스템에서도 접근이 가능해야 한다.
중앙 통제가 없어야 한다.
기존 정보에도 적용이 가능해야 한다.
```



## <span style="color:#802548">_망분리_</span>
- 인트라넷에 대해 먼저 살펴보자. 인트라넷은 대부분 망분리를 통해 인터넷을 사용불가능하게 하고 내부망을 통하게끔하는 작업이다.
- 물리적 망분리는 업무용컴퓨터와 인터넷용 컴퓨터를 분리하여 망을 물리적으로 다르게 배치하는 등의 방법이 있다.
- 논리적 망분리는 주로 VDI를 활용한다. 고성능 서버를 만들어 여러 데스크톱을 연동시키는데, 그 과정에서 내부망만 가능하게 환경을 구축할 수 있다.




## <span style="color:#802548">_LAN과 WAN, 인터넷과 공유기_</span>
- 인터넷으로 통신을 하기 위해서는 IP 주소가 있어야 한다.
  - 현실에서는 통신사에게 회선을 받거나 or 와이파이를 사용하면 IP가 자동으로 부여된다.
  - 그런데 기기가 여러개로 늘어나게 되면 어떡할까? 
  - 통신사랑 회선을 두 개를 계약할 수도 있다. 그러나 그러면 값이 많이 나간다.
  - 따라서 통신사와는 회선을 1개로만 놔두고, 공유기를 사용한다.
- 공유기는 어떤 역할을 하는가? WAN과 LAN을 볼 수 있는데, 통신사랑 받은 케이블은 WAN에 꽂는다.
  - 회선으로 받은 IP주소는 이제 공유기에 접속하게 된다. 무선은 안테나, 유선은 LAN에 꽂아서 기기들을 사용한다.
  - 외부 네트워크(59.6.66.238)라고 부여된다면, 공유기는 내부에서 쓰이는 또 다른 IP인 192.168.0.1로 부여되고, 그 외에 기기는 192.168.0.2, 192.168.0.3, 192.168.0.4 등으로 부여된다. 
  - 이러한 내부 IP(사설 IP)는 전 세계에 같은 사설 IP를 가진 컴퓨터가 동시에 여러개 존재하겠지만,  각각 다른 지역 네트워크에 속해있으므로 충돌을 걱정할 필요가 없다.
- 192.168.0.1과 같이 처음 WAN에서 LAN으로 거쳐가는 관문의 IP를 Gateway address, Router Address라고 부르기도 한다. 이른바 gateway다.
  - 기기는 무조건 Gateway address에 요청을 보낸다. 그럼 Gateway address(주로 공유기)는 WAN을 통해 인터넷에 해당 IP에 요청을 보내게 될 것이다. 
  - 그런데 사설 IP는 지역 네트워크만 쓰는 것이니 WAN에는 쓸 수가 없다. 따라서 기기의 사설 IP를 공인 IP로 바꿔서 WAN에 요청을 보낸다. 
  - 또한 사설 IP는 기기가 여러개 연결되면 복수 개이므로 정확히 어떤 기기가 해당 요청을 보냈는지 정보를 기록해둔다. 
  - 이렇게 하나의 공인 IP로 여러 대 호스트가 인터넷에 접속가능하게끔 하는 과정을 Network Address Translation(NAT)라고 한다.
    - 즉 어떤 사설 IP(기기)가 요청을 보냈는지 기록하고, 공유기가 이 요청을 공인 IP로 변경하여 WAN에 내보내고, response를 받으면 기록된 정보를 보고 정확히 해당 기기에 response를 준다.
    - 이러한 기술을 NAT라고 한다.

- 하나의 공유기에서 다른 단말로 가는 과정은 아래와 같다.
- 공유기에서 데이터가 bit로 변환되어 나간다. 이 과정은 encapsulation을 거친다.
  - encapsulation이란 데이터의 앞부분에 전송하는데 필요한 정보를 헤더에 넣어서 다음 계층으로 보내는 행위를 의미한다.
  - 전송계층에서는, 신뢰할 수 있는 통신이 이루어지도록 ack, syn, checksum 정보를 tcp 헤더에 넣는다.
  - 네트워크 계층에서는, 다른 네트워크와 통신하기 위해 목적지/출발지 IP 정보를IP 헤더에 넣는다.
  - 데이터 링크 계층에서는, 물리적인 통신 채널을 연결하기 위해  목적지/출발지 MAC주소를 이더넷 헤더에 넣고 전송중 오류를 감지하기 위한 트레일러를 붙인다.
  - 피지컬 계층은 별도의 헤더를 붙이지 않는다. 그저 전기 신호를 보내고 받을 뿐이다. 해당계층은 데이터 검증작업이 없다는 의미다.
- 공유기에서 연결된 ISP가 제공하는 router로 이동한다. 해당 router는 network 계층까지만 헤더를 까보고 유효성을 검증한다.
- 그 뒤에 라우팅을 통해 최적 경로를 탐색한다. tier2 망이라면 tier1망(백본망)을 거쳐야 하는 경우도 있다.
- 그렇게 된다면 tier1망의 router를 거쳐 서버로 들어가게 될 수 있다.
- 이 경우 다른 router로 다시 이동하게 된다. 그럼 다시 network 계층까지 헤더를 decapsulation했다가 데이터가 유효하면 encapsulation해서 다시 내보낸다.

<img src="/assets/router-encapsulation.png" />

- 그럼 해당 router에서 또 다시 라우팅을 하여 해당 서버로 가는 최적의 경로를 탐색한다.
- 서버로 가는 최적 경로 탐색을 마치면 다시 서버로 bit를 전달한다.
- 그럼 서버에서 decapsulation을 통해 데이터를 복원한다.


## <span style="color:#802548">_network data transfer process_</span>
- 데이터로 통신하는 것은 아래와 같은 과정을 거친다.
- 보내는 것은 데이터를 여러 헤더로 감싸서 보내기 때문에 encapsulation이라고 한다.

<img src="/assets/encapsulation.png" />

```
1. 보내는 측에서 데이터를 생성

2. 데이터를 패킷으로 분할
-   패킷에는 순서 번호와 오류 검사를 위한 정보가 포함

3. 패킷에 IP부여 및 라우팅
-   패킷에 목적지 IP 주소와 출발지 IP 주소를 부여 및 효율 경로 탐색
-   라우팅 테이블을 구성한다.

4. 패킷을 전기 신호(디지털/아날로그)로 변환

5. 실제 물리장비에 전송

캡슐화(Encapsulation)
ㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡ
```

- 받는 것은 헤더로 감싸진 데이터를 전부 풀어서 복원하기 때문에 decapsulation이라고 한다.

<img src="/assets/decapsulation.png" />

```
비캡슐화(DeEncapsulation )
1. 실제 물리장비(이더넷 케이블, 와이파이)에서 수신

2. 수신된 전기 신호(디지털/아날로그)를 패킷으로 변환 

3. 패킷 재조립
-   패킷의 목적지 IP 주소를 확인하여 해당 패킷이 올바른 수신자에게 도착했는지 확인/ 

4. 데이터의 정합성 검증
-   패킷의 순서가 올바르지 않거나 오류가 발견되면 송신자에게 재전송을 요청/ 

5.  데이터 복원
-   압축해제, 암호화 해제 등이 일어난다

비캡슐화
ㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡㅡ
```

## <span style="color:#802548">_network protocol_</span>
- 프로토콜은 네트워크에서 데이터를 주고 받기 위한 통신 규칙이다.
  - 네트워크가 생긴 초기에는, 네트워크에서 통신을 할 때 서로 규약이 달라 데이터가 해석되지 못하는 일이 흔했다.
  - 따라서 모두가 지키게끔 규약이 규정되게 된다.
  - 보통 데이터 전송방식, 패킷구조, 오류처리방식을 정의한다.
- OSI 7 계층보다 TCP/IP model이 먼저나왔고, 실무에서는 TCP/IP updated model이 사용된다.
  - 빠르지만, 신뢰성이 떨어지는 건 UDP고, 빠르진 않아도 신뢰성이 높은 건 TCP 방식이다.
  - UDP는 데이터를 검증하지 않기 때문에 조금 정보가 빠져도 상관없는 실시간 스트리밍 등 동영상에 많이 사용된다.
  - UDP 기반 중에는 DNS가 있고, TCP 기반 중에는 HTTP가 있다. 
    - HTTP의 패킷은 header와 body로 구성되는데, DNS의 패킷은 header와 Question, Answer, Authority, Additional로 구성된다.
    - HTTP와 DNS는 모두 request - response 구조지만, HTTP는 TCP기반으로 오류처리가 있고, DNS는 UDP기반으로 오류처리가 없다는 데서 다르다.
- 그외에도 FTP, SMTP, WEBSOCKET, SSH 등 다양한 protocol이 존재한다.

## <span style="color:#802548">_물리 장비_</span>
- 장거리 통신 시 신호가 감쇠하는 현상이 있었고, 이를 해결하기 위해 신호를 증폭하는 repeater가 개발되었다.
- 그러다가 리피터의 기능이 추가된 허브가 탄생했다. 허브는 신호를 증폭하는 것에 그치던 리피터와 달리 연결된 모든 장비에 신호를 보낼 수 있었다.
  - 허브는 모든 장비에 신호를 보내는 것은 그만큼 쓸모없는 데이터들이 기기에 많이 간다는 단점이 존재했다.
  - 내가 쓰지 않아도 허브에 들어온 데이터는 모두 기기에 가서 폐기되면서 쓸모없는 트래픽이 양산되었다.
- 그에 따른 부작용을 막기위해 bridge가 개발되었다. 브릿지는 호스트와 게스트 LAN을 연결하여 하나의 네트워크로 구성한다. 
  - MAC주소에 맞게 forwading하는 게 주 기능이며, nic처럼 신호를 데이터로 변환하지는 못한다.
  - 그러나 소프트웨어적인 제어로 forwarding이 이뤄졌기 때문에 속도가 아쉬운 부분이 있었다.
- 그에따라 속도를 향상시키기 위해 하드웨어를 통해 제어하는 switch가 개발되었다.
  - 스위치 또한 bridge 기능이 포함된다. 그런데 port까지로 확장되었다.
  - 따라서 IP가 아니라 IP 내 응용 프로그램에 필요한 packet만 공급받을 수 있게 되었다.
  - 불필요한 데이터를 걸러내는 효율이 더 크게 향상되면서 이더넷 기반 네트워크가 급증하게 되었다.
  - L2, L3, L4, L7 스위치 등 다양하게 있는데, 단계가 올라갈수록 성능이 좋아지고 처리범위가 넓어진다.
```
리피터
신호만 증폭한다. 기기를 연결할 수 없다.

허브          
기기 연결이 가능하다. 패킷을 받으면 연결된 모든 장비들에게 보내 데이터가 많이 폐기된다.

브릿지
자기에게 연결된 장비들의 MAC주소를 가지고 있다.
패킷이 오면 그 패킷의 목적지를 파악해서 그 IP로만 패킷을 보낸다
소프트웨어적 제어라 스위치보다 느림

스위치
자기에게 연결된 장비들의 MAC주소와 port까지 알고 있다.
패킷이 오면 그 패킷의 목적지를 파악해서 그 application으로만 패킷을 보낸다
하드웨어적 제어라 빠름
```

- 이젠 router와 nic에 대해 알아보자.
- nic는 network interface card의 약자이며, LAN카드, 이더넷카드, 네트워크 어뎁터 등으로 불린다.
  - nic는 1계층에서 받은 전기 신호를 데이터 형태로 만든다.
  - 그 후 목적지/출발지 MAC주소를 주소를 확인한다.
  - nic의 MAC주소와 맞으면 메모리에 데이터를 적재하고, 안 맞으면 폐기한다.

- router는 원격지로 쓸데없는 패킷이 전송되지 않게 브로드캐스트와 멀티캐스트를 제어한다.
  - 또한 불분명한 주소로 통신을 시도할 경우, 이를 폐기한다.
  - 패킷이 최적의 경로로 전송되도록 알고리즘이 사용된다.
  - 대표적으로는 거리-벡터 알고리즘(벨만-포드), 링크 상태 알고리즘(다익스트라)이 있다.
  - 거리-벡터는 인접 라우터까지만, 링크 상태는 전체 네트워크 고려하여 라우팅 경로를 최적화한다.
  - 일반 가정집에서는 보통 router보다는 공유기를 사용한다.




## <span style="color:#802548">_OSI 7 layer_</span>
- 네트워크의 기능은 신뢰성 있는 데이터, 최적 통신 경로, 목적지로 데이터 전송 등이 있다.
- 이러한 기능을 구현하기 위해 모듈화가 진행됐다. 즉 기능이 각 모듈별로 흩어졌다는 의미다.
  - 그 기능을 모듈로 나눌 떄, 계층별로 나누게 되었다. 그럼 단계별로 문제를 확인하기가 쉽기 때문이다.
  - 각 레이어의 프로토콜은 하위 레이어의 프로토콜을 사용해서 기능을 구현하는 형태다.
  - OSI 7 layer는 현실에선 잘 쓰이지 않는다. 인터넷 기반은 TCP/IP stack이고, 최근에는 TCP/IP updated model이 쓰이고 있다.
- 각 프로토콜을 서술하자면 아래와 같다.
```
application layer 
역할: 사용자에게 애플리케이션을 제공
프로토콜:FTP, SMTP, HTTP

presentation layer
역할: 애플리케이션 간 통일된 통신 가능한 형식으로 데이터를 변환
프로토콜: SSH, TLS

session layer
역할: 애플리케이션 간 연결-연결유지-연결해제를 관리
프로토콜: SSH, RPC

여기까지 PDU는 data, message

transport layer
역할:  다른 네트워크에 존재하는 IP port에 데이터가 정상적으로 전송/송신되는지 확인
헤더에 든 것: 목적지/출발지 port, seq/ack number 등
프로토콜: TCP, UDP
PDU: segment

network layer
역할: 다른 네트워크에 존재하는 논리주소 IP에 데이터를 전송/송신
헤더에 든 것: 목적지/출발지 IP 주소, Time To Live(패킷 수명)
프로토콜: IP, ARP, NAT
PDU: packet
장비: router, L3 스위치

data link layer
역할: 다른 네트워크에 존재하는 물리주소 MAC에 데이터를 전송/송신
헤더에 든 것: 목적지/출발지 MAC 주소
Trailer: FCS(비트열 오류 검사)
PDU: frame
장비: nic, bridge, switch

physical layer
역할: 다른 네트워크에 전기 신호를 전달한다.
PDU: bit
장비: 케이블, 리피터, 허브

```

## <span style="color:#802548">_TCP/IP_</span>
- Ip header와 TCP header를 제외한 TCP가 실을 수 있는 데이터를 segment라고 한다.
- TCP header가 가진 정보는 대략 아래와 같다.
  - 출발/목적지 port
  - ack/syn number
  - seq number
  - window size
  - checksum
- 이들을 기능으로 나누어보면 아래와 같다.
  - 흐름제어 -> window size, ack number, seq number
  - 신뢰성 보장 -> checksum
  - forwarding -> 출발/목적지 port
  - 혼잡 제어는 -> ack number
- 흐름제어는 아래와 같은 방법이 있다.
  - stop and wait
  - sliding winodw(go back n ARQ)
  - 재전송
- 혼잡제어는 ack number가 3번이상 중복 or timeout 시에 일어난다.
- 알고리즘으로는 아래와 같은 방법이 있다.
  - AIMD            -> 윈도우 크기 선형적 증가 1-2-3-4
  - slow start      -> 윈도우 크기 지수적 증가 1-2-4-8
  - fast retransmit -> 중복 ACK를 3개 받으면 재전송이 이루어진다. 
  - fast recovery   -> 혼잡한 상태가 되면 윈도우 크기를 1로 줄이지 않고 반으로 줄이고 선형 증가한다


## <span style="color:#802548">_TCP/IP stack_</span>
- TCP/IP stack에서는 application layer, transport layer, internet layer, link layer가 있다.
  - transport부터 link까지는 하드웨어/펌웨어, OS레벨에서 관리된다. 네트워크 기능을 지원하는 게 목적이다. systme이라고도 한다.
  - application layer는 application을 사용하는 데 목적이 있다. 네트워크 기능을 사용하는 게 목적이다. application이라고도 한다.
    - host 사이의 통신은 데이터를 보내는 데 목적이 있어 데이터를 안정적으로 주고 받지 않는다. IP가 대표적이다.
    - 그런 이유로 Internet Protocol(IP)은 unrelible하다. data loss, out-of-order 상태에 놓일 수 있다. 목적지 host까지 가도 100% 안전성을 보장할 수 없다.
    - 그러나 프로세스 간의 통신에서는 데이터를 안정적으로 주고받아야 하는 protocol이 필요했다. 
    - 그런 이유로 TCP가 차후에 개발되었다. TCP에서 데이터를 어떻게 안정적으로 주고받는 것일까?
- 흐름제어의 핵심은 TCP가 connection을 맺는다는 데 있다. connection은 프로세스 간의 안정적이고 논리적인 통신 통로다.
  - connection을 열면 3 way handshake가 일어나고, 연결되면 데이터를 주고 받는다. 주고 받는 게 끝나면 connection을 닫는다. 그게 4-way handshake다.
  - connection은 서로 다른 process 간의 data가 통신될 수 있는지 확인하는 channel이고, port는 다른 process에서 온 data를 system에서 올려보내는 통로다. 
  - process 간에는 connection을 맺고, port를 통해 data를 주고받을 수 있게 되는 것이다.
- connection을 만들기 위해서는 상대 host를 찾는 것을 넘어 상대 process를 찾아야 한다.
  - 그래서 궁리한 결과, port number가 정의되었다. port number는 16 bit의 숫자로 0 ~ 65535다.
  - host 내에서 유일한 프로그램임을 보장하는 port number가 있다.
  - 네트워크 상에서 유일한 host임을 보장하는 IP가 있다.
  - IP:port number의 조합으로 인터넷 상의 모든 프로그램을 unique하게 식별할 수 있는 것이다.
    - 그리고 IP와 port를 합친 걸 바로 socket이라고 한다. 즉 socket은 한 네트워크 내 프로그램의 유일한 데이터 진입로다.
    - 인터넷 상에서 존재하는 각 프로그램은 모두 socket을 통해 data를 받게 된다. 인터넷의 namespace인 셈이다.
- IP와 port number의 조합으로 프로그램은 유일성을 보장받게 되었으며, 따라서 유일한 것끼리 connection을 맺으면 그 connection도 유일하다.
  - 즉 한 쌍의 socket으로 connection을 지으면 반드시 인터넷 상에서 unique한 connection, port가 되는 것이다.
  - 하나의 socket은 동시에 여러 connection에서 사용될 수도 있다.
    - socket은 원래는 TCP에서만 쓰이다가 자연스레 UDP에서도 쓰기 시작했다.
    - 그래서 protocol까지 고려하게 되고, (protocol, IP, port number)의 조합이 socket이 된다.

<img src="/assets/tcp-ip-socket.png" />


## <span style="color:#802548">_실제 socket의 구현_</span>
- application은 system의 기능을 함부로 쓸 수 없다.
  - application은 커널 코드에 직접 접근이 안되고 인터페이스를 통한 I/O call로 간접적으로 system의 기능을 호출한다.
  - 그 중 네트워크 기능 쪽 인터페이스는 socket이다. 우리는 socket을 통해 data를 주고 받는다.
  - 네트워크 상의 다른 프로세스와 데이터를 주고받게 구현하는 것을 socket programming이라고 한다.
  - 물론 대부분의 application 개발자는 socket을 만지지 않는다. 대부분 내부 library나 모듈 형태로 해당 기능이 제공되기 때문이다.
  - 실제 socket은 (protocol, IP address, port number)로 정의된다. 
    - 프로토콜을 지정하고, 소켓주소(IP address, port number)를 지정해준다.
    - 서버쪽 프로그램은 port number를 명시해야 client에서 해당 port로 데이터를 보낼 수 있기에 주소 binding은 필수다. client는 필수는 아니고, OS가 알아서 binding해준다.

<img src="/assets/implementation-socket.png" />

- 이론적으론 본인 프로그램의 (protocol, ip address, port number)로 unique한 socket으로 식별이 되어야 한다.
  - 하지만 실제 구현에서는, TCP에선 그렇지 않다. OS에서 다르게 구현되었기 때문이다.
  - 8080이 server고 49999가 client라고 해보자.
  - server에선 client가 보내는 request를 항상 기다리는 socket이 필요하다.
    - connection을 맺는 요청을 기다리는 socket을 listening socket이라고 한다.
    - 이 socket에서 바로 3 way handshake가 이뤄진다.
    - connection이 성립되었는데 다른 process에서 request가 오면?
    - 다른 process와 listening socket에서 3 way handshake를 맺고, 소켓을 하나 더 발급한다.
    - 즉 program과 socket이 1:1로 대응하지 않고, 1 대 다로 대응하게 된다.
      - 이때 OS에서 세개의 TCP socket이 IP address와 port number가 똑같다.
      - 3개가 똑같은 socket같은데 식별을 하는 방법이 뭘까?

<img src="/assets/implementation-same-socket.png" />

- 실제 구현에서는 connection 연결 요청이 오는 경우는 listening socket으로 data를 보낸다.
- connection이 성립된 이후에는 보낸쪽의 IP정보와 port까지 보고서 어떤 socket인지 식별한다.
  - TCP는 (src IP, src port number, dest IP, dest port number)로 connection의 유일성을 보장한다.
  - 즉 실제 TCP socket 구현은 dest IP와 dest port number만으로는 connection의 유일성을 보장하지 못한다.

<img src="/assets/tcp-socket.png" />

- UDP는 connection을 맺지 않고 listening port도 없다. 한 프로그램에 하나의 socket만 존재하는 셈이다.
- 아래 사진에서 connection만 뺴면 UDP의 작동방식이다.

<img src="/assets/udp-socket.png" />

- UDP socket에서 데이터를 읽으면 어느 UDP socket에서 왔는지 알 수 있다.

- port는 0 ~ 1023 사이는 널리알려진 port다.
  - HTTP는 80, HTTPS는 443, DNS는 53이다.
- 1024 ~ 49151은 IANA에 등록한 번호다.
  - MYSQL DB는 3306이고, Tomcat은 8080이다. React는 3000이다.
- 49152 ~ 65535는 dynamic port로 OS가 자동할당해주는 비어있는 공간이다.


- 요약하자면 다음과 같다.
- TCP connection을 맺으면 데이터를 통신할 수 있다는 확인이 끝난 것이다. 그럼 서버-클라 간 데이터를 서로 주고 받기 시작한다.
- 해당 네트워크 내 시스템(OS 수준)에서 특정 application으로 데이터를 끌어올리는 논리적 통로를 port라고 한다.
  - 따라서 특정 application 내에 붙은 port는 여러개 일수도 있다. 웹서버 프로그램은 실제로 HTTP는 80, HTTPS는 443으로 두 port가 붙어있다.
  - 보통 port는 port number로 식별한다. 다만 목적지 IP/port number만으로는 상대방 program을 특정할 수 없다.
  - 따라서 실제로 TCP는 출발지 IP/port number도 같이 확인해 상대방 program을 특정하게 된다.


## <span style="color:#802548">_주요 출처_</span>
- https://www.youtube.com/watch?v=oFKYzp6gGfc&list=PLcXyemr8ZeoSGlzhlw4gmpNGicIL4kMcX (유튜브 쉬운코드)
