# ICMPv4

## 역할

- 게이트웨이나 목적지 호스트에서 에러가 나면 이를 소스 호스트에 리포트하는 용도로 사용
- IP는 reliable한 프로토콜이 아님! ICMP는 그냥 전달이 안 되었는지 확인하는 용도일 뿐, 이걸로 reliability가 보장되지 않음!

## Messege Formats

- ICMP 메세지는 IP header를 이용해서 보내짐
  - Protocol: ICMP = 1
  - Source Address
    - 에러메세지를 보내는 곳 주소 (게이트웨이든 호스트든)
  - Destination Address
    - 에러메세지를 받는 곳 주소. 즉 에러메세지의 원인이 된 패킷을 보낸 곳
  - Checksum

## 종류

- ICMP error messages
  - Destination Unreachable
    - Code
      - set by gateways
        - 0 = net unreachable
        - 1 = host unreachable
        - 4 = fragmentation needed and DF set
        - 5 = source route failed
      - set by hosts
        - 2 = protocol unreachable
        - 3 = port unreachable
  - Redirect (처음 간 곳이 최종 목적지가 아니라 다시 보내야 하는 상황)
    - Code
      - 0 = Network
      - 1 = Host
      - 2 = Type of Service and Network
      - 3 = Type of Service and Host
    - 1 -> 2로 보냈는데 게이트웨이 3이 "4로 보내는 게 맞음ㅇㅇ": 3 -> 1로 ICMP 메세지 보냄
  - Time Exceeded
    - Code
      - sent by gateways
        - 0 = time to live exceeded in transit
      - sent by hosts
        - 1 = fragment reassembly time exceeded
  - Parameter Problem
    - Code
      - 0 = pointer indicates the error
    - Pointer
      - code = 0이면 정확히 어디가 에러인지 포인팅
- ICMP query (or diagnostic) messages
  - Echo or Echo Reply (ping)
    - Code: 0
    - Identifier
    - Sequence Number
  - sender가 에코 리퀘스트 보내면 reciever는 무조건 증가한 sequence number로 에코 리스폰스를 보내야 함. Identifier는 같아야 함. 즉 각 쌍이 매치되는지 알 수 있어야 함
  - Timestamp (안 중요함)

## 주의

- ICMP error message는 다음 경우들에 대해 만들어지지 않는다
  - ICMP error message
  - IP broadcast, IP multicast address
  - link-layer broadcast
  - 첫번째가 아닌 fragment
  - 소스 주소가 싱글 호스트로 정의되지 않은 경우
 
## Path MTU Discovery (그리 중요할 것 같진 않음)

- fragmentation을 줄이기 위해 path maximum transmission unit (MTU)를 찾는 데 쓰임
- basic idea
  - 걍 시간 남으면 읽어봐라

## Rate Limiting (얘도 그닥...)
