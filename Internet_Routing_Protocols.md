# Internet Routing Protocols

- Routing
  - (N)-layer의 라우팅은 (N)-entities의 연결로 설명
  - 실제로는 그 윗 레이어, 아래 레이어를 통해서 연결되더라도, 그냥 N-layer는 N-layer로 연결된다고 생각
- 그럼 라우팅 테이블 (포워딩 테이블)은 어떻게 만들어지는가?

### Unicast Packet Forwarding

- 포워더는 next hop IP 주소를 라우팅 테이블 가지고 정함
  - 호스트는 디폴트 게이트워이 혹은 목적지 호스트에 다이렉트로
  - 게이트웨이는 이웃 라우터 혹은 목적지 호스트에 다이렉트로 
- 게이트웨이는 interior gateway protocol (IGP)를 가지고 같은 도메인 안에 있는 (same autonomous system, intra-AS) 전체적인 토폴로지를 알아내서 라우팅
  - IGP: RIPv2, OSPFv2, IS-IS, EIGRP (cisco)
- 다른 도메인에 있는 경우 (inter-AS) exterior gateway protocol (EGP)를 이용해서 라우팅 (예: 미국 - 한국, SKT - KT)
  - EGP: BGPv4
- IGP는 최대한 자세한 정보를 이용해 optimality에 초점, EGP는 정보 최소화

## Routing General Concept: Routing Algorithms

### IP forwarding mechanism

- Hop-by-Hop routing (default)
  - 목적지 주소를 가지고 다음 이웃 라우터를 계속 찾음
  - 포워딩 할 때마다 무조건 목적지에 가까워져야 함
- Source routing (Source and Record Route Option)
  - IP option 참고
  - 근데 잘 안 쓰임

### 라우팅을 위한 알고리즘들

- Distance vector algorithm
  - **나는 목적지로부터 이만큼 멀어!**
  - 각 라우터는 reachability 정보를 이웃에게 공유
    - reachability 정보는 목적지, metric (cost to the destination), 버전 정보가 담겨 있음 (RIP에는 버전 정보가 안 담겨서 문제가 생김)
  - Bellman-Ford algorithm 이용해서 라우팅 테이블이 바로바로 업데이트됨
- Link-state algorithm
  - **나는 이런 애들이랑 이웃해 있어!**
  - 각 라우터는 자신의 토폴로지 정보를 모든 자신의 인터페이스에 advertise
    - 토폴로지 정보는 reachable한 네트워크, 각 네트워크의 cost, 버전 정보가 담겨 있음
  - 이렇게 받은 토폴로지 정보는 저장 및 릴레이됨. 저장은 network topology database에 되는 게 전부
  - 라우터 자신 기준으로 토폴로지 정보를 통해 shortest path tree (SPT)를 구함
    - dijkstra algorithm 이용
    - 라우팅 테이블은 이 때 만들어짐
- Path vector algorithm (Distance vector algorithm의 subset으로 보기도 함)
  - 각 라우터는 reachability 정보를 이웃에게 공유
    - reachability 정보는 목적지, 경로가 담겨 있음. 그리고 경로의 각 노드가 Autonomous System Identifier로 이루어짐
    - 경로는 {A, (B|C), D}, {(A, D)|(A, B, D)}와 같은 방식으로 sequential or parallel로 표현 가능
  - 이러한 reachability 정보는 다른 알고리즘들과 달리 공유하거나 지우거나를 같이 함

## Distance Vector Algorithm in RIPv2

- IP에서 네트워크는 하나의 entity
- Distance Vector Algorithm은 라우터가 목적지와 목적지까지의 cost 정보를 교환해야 성립
- 각 게이트웨이는 라우팅 프로토콜을 가지고 목적지 관련 정보를 모아둠
- 일반적으로, 라우터는 연결에 대한 정보와 목적지에 도달하는 방법에 대한 정보를 한 엔트리에 저장해두기 때문에, 호스트 입장에서는 (IP 관점에서) 그냥 라우터한테 보내면 알아서 잘 보내게 될 것이다. 신경쓰지 마라!
- 가끔은 호스트 주소 그 자체를 라우트하는 경우도 있음. 이게 바로 네트워크 prefix가 32비트인 경우
  - Host-specific routing
- RIP(뿐만 아니라 대부분의 프로토콜)는 호스트와 네트워크를 구분하지 않음. 걍 prefix가 32비트면 호스트, 아니면 네트워크
- Distance Vector Algorithm은 거리 정보만 교환되어도 최적의 루트를 찾을 수 있음
  - 다만 시스템 내의 모든 목적지에 대한 베스트 루트를 테이블로써 가지고 있어야 하고, 이를 만들기 위해 metric을 각 hop의 cost의 총합으로써 구할 수 있어야 함

### Entry in The Routing Database

- 목적지 (마스크 필수)
- 다음 라우터
- metric (cost) - RIPv2에서는 그냥 라우터 개수로 계산되는데 IGRP에서는 딜레이, 코스트 등을 고려해서 계산. 그래도 더해서 쓸 수는 있어야 함

### Bellman-Ford Algorithm

- D(i,j) = metric i to j
- d(i,j) = cost i to j -> 바로 붙어있는 애들한테 쓰는 거라고 보면 됨
- D(i,i) = 0 for all i
- D(i,j) = min\_k(d(i,k)+D(k,j)) for i!=j
- 이 알고리즘에서 D(i,j)는 몇번 반복하면 결국 어떤 값으로 수렴함
- basic assumption
  - 엔티티는 메트릭의 업데이트와 재계산을 무조건 해야 함
  - 보내주는 업데이트 정보가 반드시 특정 시간 안에 상대 라우터에 도착해야 함
  - 빼고는 아무런 assumption 존재 안 함

### The Basic Distance Vector Algorithm

- 각 라우터는 모든 가능한 destination N에 대해 라우팅 엔트리 테이블을 가지고 있음
  - <N, D, G> where D: Distance, G: first router of network N
- 각 라우터는 주기적으로 업데이트된 <N, D>를 주변 G에게 보냄
- 이웃 G'으로부터 업데이트가 오면, 그걸 받은 라우터는 코스트를 associate (D' = X.cost+D'')
- <N, D, G>와 <N, D', G'>을 비교
  - D'이 더 작으면 <N, D', G'>로 업데이트
  - D'이 더 작지 않은데, G = G'이면 \<N, D', G\>로 업데이트 -> 같은 라우터에 대해 코스트가 증가했다는 의미이므로 이럴 때에는 예외로 업데이트 해야 함

### RIPv2에서의 Distance Vector Algorihm의 한계

- RIPv2는 주고받는 정보에 버전정보가 없음 -> **counting to infinity**
- 15hop 이하의 경로에 대해서만 쓸만함
- 고정되어 있는 (홉 개수) 메트릭을 사용하기 때문에 매우 쉽게 구현됨
- **위 단점들은 RIPv2의 단점이지, Dist Vector Algo의 단점이 아님!!**

### Counting to Infinity

- 중간에 어떤 링크(edge)가 끊겨버렸을 때, 남은 라우터들이 <N, D, G>, <N, D', G'>을 비교하는 과정에서 G = G' 조건에 걸려서 업데이트를 하는데, 그걸 계속 둘이 왔다갔다 하면서 D가 커짐 (예시 참조)
- 다행히 언젠가는 그 커지는 metric보다 작은 경로가 나타나면 멈추겠지만, 그때까지 왔다갔다를 오지게 해서 문제
- RIPv2가 이를 해결하는 방법
  - Split horizon
  - Split horizon with Poison Reverse
  - Triggered update
- 그냥 버전정보(sequence number and/or age)를 업데이트하는 게 제일 편한데 위 방법으로도 충분히 커버는 됨

#### Split Horizon

- 이웃 라우터에게서 배운 루트는 해당 라우터에게 다시 보내지 않기로 함
- ex: C: <TN, B, 3>는 B에게 업데이트를 보내지 않음

#### Split Horizon with Poison Reverse

- 이웃 라우터에게서 배운 루트는 해당 라우터에게 다시 보낼 때 infinite distance로 보냄
- ex: C: <TN, B, 3>는 B에게 <TN, infinite>로 보내줌
- 이 방법을 쓰면 두 라우터끼리의 루프는 해결 가능. 근데 3개 이상은 루프 해결 불가능
- 그래도 웬만하면 2개씩만 루프가 돌아서 걍 이 방법을 많이 씀

#### Triggered Updates

- Counting to Infinity는 새 정보를 다시 이웃에게 보내는 부분이 너무 느려서 문제가 되는 것
- 그래서 Triggered updates는 메트릭이 바뀌는 순간 매우 빠르게 업데이트 메세지를 보냄
- 이렇게 되면 속도가 빨라져서 해결

### RIPv2의 구조

#### Assumption of RIPv2

- RIP는 IPv4-based network상에서 정보를 계산하고 교환함
- 당연히 자신의 네트워크 정보를 알고 있어야 함
  - 최대 메트릭이 15이므로 infinite는 15로 설정

#### RIPv2 Routing Table Entry

- RIP 라우팅 테이블의 엔트리 필드
  - 목적지 (IPv4와 network mask)
  - 메트릭
  - next-hop의 주소
  - 인터페이스 (next-hop 가는 데 쓰이는)
  - 루트가 최근에 바뀌었는지 확인하기 위한 flag
  - 루트와 관련된 타이머
- 처음 엔트리 만들 땐 직접 연결되어 있는 네트워크에 대한 정보만 만들어짐. 메트릭은 1로 (표준이 강제한 건 아니지만 다 이렇게 만듦)

#### RIPv2 Spec

- UDP-based
- Source/Destination UDP port number: 520
- 최대 메세지 크기: IP header (20) + UDP header (8) + RIPv2 (4+500) = 576
- Command (1옥텟)
  - (1) request, (2) response
- Version (1옥텟)
  - (1) RIPv1, (2) RIPv2
- RIP Entry 리스트 (500옥텟 이하) -> 25엔트리 이하
  - RIP Entry Size가 20옥텟
    - Address Family Identifier (2옥텟)
      - AF\_INet (2)
    - Route Tag (2옥텟)
      - 다른 라우터로부터 엔트리를 새로 채택하면 해당 라우터의 태그를 가져옴
      - 아예 다른 프로토콜을 쓰면 그걸 구분하기 위해 쓰임 (AS 넘버 등)

