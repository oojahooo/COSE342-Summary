# OSPFv2

## Link State Algorithm

- 네트워크 토폴로지 DB를 만듦
  - 각 라우터는 로컬 토폴로지 정보를 넘겨
  - 각 라우터는 받은 토폴로지 정보를 데이터베이스에 저장하고 넘겨
- 네트워크 토폴로지로부터 라우팅 테이블을 만듦
  - 모든 라우터는 독립적으로 자신이 루트가 되어 SPT(Shortest Path Tree)를 만듦 (using Dijkstra algorithm)
  - SPT를 만들면서 라우팅 테이블을 같이 만듦

### Dijkstra Algorithm

- node들은 next-hop, interface, cost를 attributes로 가짐
  - edge cost: 라우터 -> 네트워크 (cost), 네트워크 -> 라우터 (0)
  - next hop: 루트로부터 첫 라우터. 그런 거 없으면 null로
  - interface: next hop이 연결되어 있는 루트의 인터페이스
  - cost: root부터 node까지의 edge costs의 총합

## OSPFv2

- SPF (Shortest Path First) based routing protocol
  - link-state or distributed-datebase protocols
  - OSI에서 ISO IS-IS routing protocol에 SPF-based algorithm을 적용
    - 토폴로지 정보를 플루딩하다 보니 routing traffic이 많이 발생하는데, 이를 줄이기 위한 메소드 존재: Designated Router
- OSPF (Open Shortest Path First)
  - SPF-based routing protocol의 한 종류
  - link-state algorithm 이용
    - 모든 라우터가 AS의 토폴로지를 설명하는 데이터베이스를 똑같이 가짐
    - 데이터베이스에서 SPT를 만들어서 라우팅 테이블 생성  
    **주의: 각 프로토콜의 라우팅 테이블은 IPv4에서의 라우팅 테이블(포워딩 테이블)과 독립적인 것**
  - 하나의 AS 내부에서 돌아감 (IGP)
  - 2-tier hierarchical 라우팅 프로토콜
    - Backbone Area (Area 0): 모든 area와 연결된 대표 area
    - Non-backbone Area
  - area 안에서는 토폴로지 데이터베이스를 고대로 만들어서 사용 (using Dijkstra algorithm)
  - area 밖에서는 (inter-area, exterior-AS routing) reachablility 정보를 사용함 (like distance vector algorithm)
    - 그래서 area border router 부분은 SPT에서 leaves일 수밖에

### Link State Advertizement Types

- Intra-Areea Topology Information
  - Router-LSA (LSA Type 1): a router information
  - Network-LSA (LSA Type 2): a transit network information
- Inter-Area Reachablility Information (Summary LSA)
  - LS Type 3: destination information <N, cost>
  - LS Type 4: ASBR information <ASBR, cost>
- Exterior-AS Reachablility Information
  - AS-external-LSA (LS Type 5): external network information
- 즉 Intra-AS는 순수 link state지만 그 밖은 distance-like

### Building Shortet Path Tree (SPT)

1. Area 단위로 SPT 생성
    - (Phase 1) Intra-area의 intermediate 노드에 대한 SPT 생성 (Type 1, 2 생성)
    - (Phase 2) stub destinations를 SPT에 추가 (Type 1 추가)
1. Intra-area destinations와 ASBRs를 SPF에 추가
    - summary-LSA (Type 3)
    - summary-LSA (Type 4)
1. External destinations를 SPF에 추가
    - LSA type 5
- destinations는 모두 leaf로 붙인다고 보면 됨

## OSPFv2 Terminology

- Router Classification
  - Internal routers (normal routers): 같은 area의 라우터들 -> router-LSA
  - Area border routers (ABR): 여러 area에 붙어있는 라우터들. backbone area에 인터페이스로 연결 -> summary-LSA
  - Backbone routers: backbone area에 인터페이스로 연결된 라우터들
  - AS boundary routers (ASBRs): 다른 AS의 라우팅 정보를 주고받을 수 있는 라우터들 -> external-AS-LSA
- Network Classification
  - Point-to-point networks: 직접 1대1로 연결한 unnebered link or numbered긴 한데 stub link
  - Broadcast network: stub networks 또는 transit networks
  - Non-broadcast networks (몰라도 됨)

<img src="resources/OSPF_Term.png" title="OSPF Terminology" alt="OSPF Terminology"></img>

<img src="resources/OSPF_Term_Network.png" title="OSPF Terminology - Network" alt="OSPF Terminology - Network"></img>

### Representing

PPT를 보셈

## OSPFv2 Router Database

- Protocol Data Structure (per router)
  - 라우터 ID
    - AS에서 유니크하게 정해진 32-bit 숫자
  - Area structure
    - Area ID (32bits)
      - backbone: 0.0.0.0
    - List of area address ranges
    - 라우터의 인터페이스 정보
    - List of router-LSAs
    - List of network-LSAs
    - List of summary-LSAs
    - Shortest Path Tree (SPF)
  - Backbone structure
  - Virtual links configured
  - List of external routes
    - ASBR 라우터인 경우에 존재하는 정보
  - List of AS-external-LSAs
  - Routing table

### LSA (Link-State Advertizement) Header

- LSA Header
  - LSA Idenrifier: <LS Type, Link State ID, Advertising Router>
  - LSA Age: <LS Sequence Number, LS checksum LS Age>
    - new: larger sequence number, then larger checksum, then smaller LS age

<img src="resources/Router_Topology.png" title="Router Topology" alt="Router Topology"></img>

- Designated Router: 2개 이상의 라우터를 가지는 네트워크에서 대표가 되는 라우터라고 보면 됨  
링크 스테이트 표현할 때에도 얘 기준으로 설명하고, 정보도 얘한테만 보내서 트래픽 줄임

### Routing Protocol Packets (아마 안 나올 듯)

- Hello
  - 각 인터페이스로 이웃 찾기
- Database Description (DD)
- Link State Request
- Link State Update
- Link State Ack


