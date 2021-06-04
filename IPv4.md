# IPv4

## IPv4란?

- Motivation
  - packet-switched 네트워크 통신에서 사용하기 위해 디자인
    - source host에서 destination host로 datagram을 보낼 수 있도록
    - fragmentation, reassembly of long datagram을 지원
- internet module은 각 host와 gateway에 대해 존재
- routing
  - host-host의 경로를 결정하는 과정
- IP는 각 datagram을 독립적인 요소로 다룸
  - connection이나 logical circuit 없음
  - state 없음

### 용어 정리

- frame
  - link-layer에서 다루는 단위
- packet
  - network-layer에서 (IP가 주고받는) 다루는 단위 -> fragment된 경우도 있기 때문에 datagram과 완전히 같지는 않음
- datagram
  - IP가 주고받는 데이터 단위
- message
  - transport layer에서 주고받는 단위
- segment
  - TCP가 주고받는 데이터

### Key machanisms

- Type of Service (ToS)
  - 서비스의 퀄리티 (QoS)
- Time to Live (TTL)
  - datagram의 최대 수명
- Option
- Header Checksum

### Model of Operation

<img src="resources/Model_of_Operation.png" title="Model of Operation" alt="Model of Operation"></img>
- (1-A) upper-layer 프로토콜이 데이터를 보냄
- (1-B) 로컬 IP 모듈이 데이터를 보내도록 호출
  - 목적지 주소와 몇가지 파라미터를 넘겨서 호출
- (2-A) datagram header를 만들고 거기에 데이터를 붙임
  - Protocol Data Unit (PUD)
  - 다음 hop을 결정하여 그 IP 주소를 알아냄
- (2-B) 다음 hop에 대한 link-layer 주소를 알아냄
- (2-C) datagram을 보냄
- (3-A) local network interface가 link layer header를 만들어 datagram에 붙임
- (3-B) link layer frame을 local network를 이용하여 gateway쪽 interface로 보냄
- (4-A) gateway의 local network interface에서 link layer frame으로 wrapped된 datagram을 받음
- (4-B) frame에서 link layer header를 없앰
- (4-C) 다시 IP 모듈의 datagram으로 변환
- (5-A) (5-B) (5-C) (6-A) (6-B) (7-A) (7-B) (7-C)
- (8-A) datagram을 해당 호스트의 upper-layer 프로토콜에 전달
- (8-B) 데이터가 upper-layer 프로토콜에 넘겨져 system call

중요한 건 IP 모듈은 네트워크에 직접이 아니라 interface에게 전달한다는 점

### Addressing

- name, address, route
  - name: 우리가 찾는 게 무엇?
  - address: 어디에?
  - route: 어떻게 가지?
- applicatoin protocol이 name으로부터 address를 매핑(DNS), IP 모듈은 해당 address를 link layer address로 매핑(ARP)

### Fragmentation

- 특정 local network가 허용하는 가장 큰 packet size로 보냈다가, 다른 local network로 갈 때 size가 작아져야 하면 fragmentation 발생
  - don't fragment하라고 설정할 수 있는데, fragmentation이 필요한 크기면 그냥 전달이 안 됨
- fragmentation과 reassembly는 datagram을 여러 조각으로 쪼갰다가 합칠 수 있어야 한다는 것
- Fragmentation-related Fields
  - identification field (16bits): 섞이지 않게 구분
    - 근데 회선 속도가 빨라지니까 uniqueness를 지키면 속도를 제대로 못 써먹음
    - 느리면 느린대로 문제
    - 그래서 걍 non-atomic datagram일 때에만 써먹기로 함
  - offset field (13bits): fragment의 위치(원래 데이터 기준 순서) 표현
  - fragment flags (3bits)
    - 0: 무조건 0
    - 1 (DF): 0 = fragment 가능, 1 = don't fragment
    - 2 (MF): 0 = 마지막 조각, 1 = 조각 더 있음

### Internet Header Format

- Version (4bits): 4
- Internet Header Length (IHL) (4bits, unit of 4 octets)
  – 옵션이 없으면(거의 대부분) 5
  – 최대 60 옥텟 (옵션의 최대 길이는 40 옥텟)
- Type of Service (8bits)
  - DS Field
  - ECN
- Total Length (16bits)
  - 헤더와 데이터 모두 포함한 길이
  - Data Length (octets) = Total Length – (IHL\*4)
  - 모든 호스트는 적어도 576 옥텟(516 + 60)만큼은 받을 수 있어야 함
- Time to Live (8bits)
  - default 값은 64
  - 원래는 시간(초)단위였는데, hop 넘어갈 때마다 1씩 줄이도록 강제돼있어 사실상 hop counter
- Protocol (8bits)
  - 다음 level의 프로토콜 종류 (TCP, UDP, ...)
- Header Checksum (16bits)
  - 헤더에 대해서만 계산, 중간에 필드 바뀌면 당연히 다시 계산. 근데 사실 중간에는 TTL만 바뀜ㅋㅋ
  - HC' = ~(C + (-m) + m') = ~(~HC + ~m + m')
- Options (안 중요함)

## IPv4 Host Data Structure

- Host Initialization
  - IP address, address mask, gateway (router) 리스트는 반드시 있어야 함
  - disk가 있는 호스트는 알아서 동적으로 설정됨 - DHCPv4
- Gateway Selection
  - 같은 destination으로 여러 datagram을 보내기 위해서는 다음 hop을 매핑해주는 route cache (destination cache)가 필요함
  - route를 캐시하는 방법
    1. 부분적으로 목적지까지의 route cache 정보가 없으면 일단 기본 게이트웨이로 보냄
    1. 그 게이트웨이가 베스트가 아니면 베스트 hop으로 forwarding하고 ICMP redirect 메세지를 소스에 보냄
    1. redirect 메세지를 받으면 호스트는 next-hop 게이트웨이를 route cache에 추가
  - deault gateway는 여러개 존재해야 함
  - route cache entry에는 다음 field들이 있어야 함
    - local IP 주소
    - destination IP 주소
    - ToS
    - Next-hop 게이트웨이의 IP 주소

## Forwarding Datagrams in IPv4 hosts

### For Outgoing Datagrams

- IP layer는 transport layer에서 채우지 않은 필드를 마저 채움
- 연결된 네트워크에서 올바른 첫 hop을 선택
- 필요한 경우 fragment
- 적절한 link-layer drive (인터페이스)로 패킷 전달

#### Routing Outband Datagrams

  - IP layer는 각 datagram마다 올바른 next hop을 선택
    - destination이 연결되어 있으면 바로 보냄
    - 안 연결되어 있으면 연결된 gateway로 라우트
  - Local/Remote Decision (Local Link Determination)
    - source와 destination이 바로 연결되어 있는지 확인하는 방법
    - address & mask 했을 때 값이 같으면 이웃한 위치라는 의미 -> 연결
    - 값이 다르면 직접 연결이 안 되어 있다는 뜻
  - Gateway Selection (the default router selection)
    - route cache entry에서 destination address와 연결된 gateway를 찾아서 선택
      - 만약 해당 destination에 대한 route cache entry가 없으면 default gateway로 보냄
      - default gateway의 리스트는 preference 기준으로 순서가 정해져 있음
    - 어떤 호스트들(Type-C host)은 포워딩 테이블 사용. 이 때 **longest prefix matching algorihhm**을 사용하여 게이트웨이 선택

#### Host 모델 종류들

  - Type A: default gateways를 리스트 형태로 가짐
  - Type B: default gateway list가 있고, 게이트웨이에 선호도가 존재
  - Type C: 포워딩 테이블 - prefix, prefix length, preference value, lifetime, next-hop router, network interface (index or IP address)로 이루어짐

#### Longest Prefix Matching Algorithm (매우 중요!!)

  - Full matching (host matching, direct forwarding)
    - 목적지 주소와 완전히 매칭, 즉 바로 목적지로
      - 엔트리의 네트워크 prefix가 32bit (전부)
      - output: 사용할 인터페이스
    - 네트워크 매칭
      - 가장 길게 매칭되는 엔트리를 찾음
      - output: next hop (gateway) 주소, 사용할 인터페이스
    - default route
      - 네트워크 prefix가 0비트 (0.0.0.0/0)
      - outpyt: default gateway의 주소, 사용할 인터페이스
    - 이마저도 없으면 패킷 버려

### For Incoming Datagrams

- IP layer는 datagram이 제대로 됐는지 확인 (체크섬 == +0?)
- 로컬호스트가 정말 목적지였는지 확인
  - datagram의 destination address field가 다음 중 하나를 만족하면 됨
    - 호스트의 IP 주소 중 하나
    - 브로드캐스트 주소
    - 내가 조인한 멀티캐스트 주소
- 옵션 실행
- 패킷 재조합
- transport layer protocol module에 메세지 전달

## IPv4 Forwarding in IPv4 Routers

### Forwarding Algorithm

- 라우터가 link layer로부터 IP 패킷을 받음
- 라우터가 IP 헤더의 validity 검사
- 라우터가 IP 옵션대로 시행
  - 몇몇 옵션은 그 이후에 추가적인 프로세싱 필요
- 라우터가 destination IP 주소를 보고 다음 프로세스 결정
  - 자기한테 온 거면 local delivery에 넘겨서 재조합
  - 자기한테 온 게 아니면 포워딩
  - 둘 다 해야 할 수도

<img src="resources/IP_Router_Comp.png" title="IP Router Components" alt="IP Router Components"></img>

### Unicast Packet Forwarding

- 포워더(forwarder)가 라우터의 라우팅 테이블을 보고 next hop IP 주소를 결정 (사용되는 인터페이스도 함께 결정)
- 포워더는 해당 패킷의 포워딩이 허락됐는지 검사
- 포워더가 TTL을 1 줄임
- 아직 안 끝난 IP 옵션이 있다면 여기서 진행 (source and record route options 같은 것들)
- fragmentation이 필요하면 함. 같은 출신이면 같은 인터페이스로
- next hop의 link layer 주소를 결정
- IP datagram을 link layer 프레임으로 포장한 뒤 선택된 인터페이스로 보냄
- 필요하다면 ICMP 메세지를 보냄 (단, 리다이렉트 메세지는 첫번째 게이트웨이만 해당)

### Local Delivery and Forwarding Decision

- 패킷이 로컬로 배달되어 더이상 포워딩을 하지 않게 되는 경우들
  - 패킷의 목적지 주소가 정확히 라우터의 IP 주소와 일치
  - 패킷의 목적지 주소가 limited broadcast 주소 (255.255.255.255)
  - 패킷의 목적지가 멀티캐스트 주소의 멤버이고 포워딩하지 말라면
- 패킷을 포워더에 넘기면서 동시에 로컬로 받는 경우들
  - 패킷의 목적지 주소가 directed broadcast 주소
  - 패킷의 목적지가 멀티캐스트 주소의 멤버이고 포워딩이 허락됨
- 나머지 모든 경우는 바로 포워더에 넘겨짐

### Local/Remote Decision for forwarding packets

- IP 주소 없이 네트워크 인터페이스가 바로 연결되는 경우 (P2P 같은 거)
  - 다른 IP 목적지 주소가 있는 라우터 id를 비교해서 id가 같으면 패킷을 보냄
- IP 주소가 라우터에 바로 대응되는 경우 (라우터랑 호스트가 직접 연결된 경우)
  1. 인터페이스의 네트워크 prefix (인터페이스 주소 & 인터페이스 네트워크 마스크)
  1. 패킷의 목적지 IP 주소의 prefix (목적지 주소 & 목적지 네트워크 마스크)
  - 위 두 값이 같으면 패킷이 해당 인터페이스로 전달
- 둘 다 아니면 포워딩

### Next Hop Address

- Forwarding Information Base (FIB), 쉽게 말해 포워딩 테이블에서 베스트 루트를 찾는 것이 목표
- FIB에 있는 여러 후보를 두고 맞지 않는 루트를 탈락시키는 방식으로 베스트 루트를 찾음 (**Pruning Rules**)
- Pruning rule이 끝나면 단 하나의 베스트 루트가 남아야 함
  - 만약 하나도 남지 않으면 못 보낸다는 뜻
  - 두 개 이상 남으면 어떻게 보내도 똑같아서 맘대로 선택하면 됨. 라우터에 따라 둘 다 써먹기도

#### Pruning Rules

- Route attributes assumption
  - route.dest: 목적지
  - route.length: 네트워크 prefix 길이
  - route.tos: OSPF에서 ToS 정보 (default = 0000)
  - route.metric: 코스트
  - route.domain
- 라우터에서 longest prefix matching algorithm의 응용버전 사용
  1. Basic Match == Full Match
  1. Longest Match
  1. Weak ToS
    - 후보 루트들에 대해 route.tos == ip.tos인 것들만 남김
    - 같은 게 없으면 걍 route.tos == 0000인 것들 남김
  1. Best Metric
   - route.domain이 같은데 route.metric이 더 큰 애가 있으면(코스트가 높으므로 더 비효율) 후보에서 탈락
  1. Vendor Policy
    - 이제 사실상 제너럴한 방법은 다 썼으니 벤더가 더 pruning 할거면 해라
