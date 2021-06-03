# UDP and TCP

## UDP

### UDP의 정의

Packet-switched computer communication에서 datagram 단위로 전송하기 위해 만들어진 매우 간단한 프로토콜.
어플리케이션이 다른 어플리케이션에 메세지를 보내기 위해 필요한 최소한의 메커니즘만 정의하였다.
즉 transaction oriented하며 delivery와 duplicate protection은 보장하지 않는다.

### UDP Header

- Header
  - IPv4 Protocol Number: 17
  - Source Port (16bits) -> optional
  - Destination Port (16bits)
  - Length (16bits) : 헤더와 데이터를 모두 포함한 총 길이
  - Checksum
    - optional field이며, 안 쓰이면 +0
    - 쓰이면 각 field에 대해 1의 보수로 더한 값들(carry를 버리지 않고 LSB쪽에 더해줌)의 총합의 1의 보수
    - 전송된 이후 다시 field에 대해 1의 보수로 더한 값들 + checksum을 하면 -0(0xFFFF)이 나와야 손실이 없다는 뜻


## TCP

### Motivation

Highly reliable한 host-to-host protocol을 만들기 위해 설계됨.
(connected-oriented, end-to-end reliable)  
또한, variable-length segments 단위로 전송할 수 있도록 함.  

### 정의

- Primary **virtual-circuit** transport protocol
  - 믿을 수 있으며, 전이중통신이 가능해야 함
  - connection-oriented transport service

### Operation

1. Basic Data Transfer (데이터 전송) (octet 단위)
    - octet 단위로 데이터를 보내면 TCP가 알아서 적당한 크기로 boxing 후 송신
    - 만약 반드시 이어서 보내야 할 데이터라면 push(PSH) flag 이용 - 버퍼에서 안 기다리고 보내기 **요청**가능
1. Reliablility (신뢰성)
    - 신뢰성 보장(데이터의 파괴, 손실, 복제, 순서변경 예방)을 위해 acknowledgment(ACK)을 이용
    - ACK가 양수인 데이터를 받아야만 하는데, 시간 안에 못 받으면 retransmitted
    - sequence number로 순서 확인
    - checksum으로 데이터 파괴 확인
1. Flow Control (흐름 제어)
    - 수신 호스트가 리소스를 얼마나 받을지 제어하여 손실(버퍼 오버플로우)이 없도록 함
    - window를 이용해 수용 가능한 octet 수를 sequence number의 범위 형태로 표현  
    -> 어떤 octet의 seq num이 마지막으로 받은 ack num과 window보다 작거나 같으면 octet을 보낼 수 있음
1. Multiplexing (한 호스트가 여러 프로세스 감당 가능)
    - 한 연결에서 socket 쌍이 \<address + port\>로 unique하게 결정
    - port는 정의상 아무거나 써도 되지만 어플리케이션마다 약속된 port를 자주 사용
1. Connections (데이터를 보내기 전에 **연결**을 만들어야 함 - TCB)
    - clock-based seq num을 사용하는 handshake mechanism으로 연결 생성
1. Precedence and Security (보안)
- congestion control(혼잡 제어) 기능도 추가. But 따로 구현됨


### TCP connection management

#### 연결의 생성 및 해제

##### OPEN call

- local port와 foreign socket 아규먼트를 통해 active OPEN call
    - `int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);`
- TCB (Transmission Control Block) 구조체 사용
- passive OPEN call은 서버에 대해서 함 - `listen()` -> `accept()`

##### Hand Shake

- 3-way Hand Shake, 4-way Hand Shake
  - 연결의 생성은 **SYN** flag에서부터 시작하여 3번 메세지를 교환 -> 3-way hand shake
  - 양방향으로 sequence number가 동기화되면서 연결 생성
  - 연결 해제할 땐 FIN flag를 보내 4-way hand shake (각 방향에서 FIN <=> ACK)
  <img src="resources/TCP_Conn_States.png" width="70%" title="TCP Connection States" alt="TCP Connectoin States"></img>
  <img src="resources/TCP_Conn_Procedure.png" title="TCP Connection Procedure" alt="TCP Connectoin Procedure"></img>

#### TCP Urgent Point

##### Urgent mechanism

- Urgent mechanism (URG)
  - sending user가 receiving user에게 urgent data를 먼저 받도록 요청하는 상황
  - receiving TCP가 urgent data를 읽기 시작하면 **urgent mode** 진입. urgent pointer만큼 옥텟을 받고 나서 **normal mode**로 다시 되돌아감.
  - Urgent pointer: urgent data의 옥텟 수 (마지막 옥텟 + 1)

##### Semantics of Urgent Indication

- Urgent Indication의 의미
  - 사실 urgent mechanism은 긴급하게 보내져야 하는 데이터를 요청하기 위함이지 out-of-band 데이터를 위함이 아님
  - out-of-band 데이터는 갓길처럼 데이터를 보내야 한다는 의미
  - 그래서 flag에다 `SO_OOBINLINE`이라고 해야 순차적으로 데이터를 가져가고, `MSG_OOB`라고 하면 out-of-band 데이터로써 먼져 가져가기로 함 -> `MSG_OOB`를 쓰면 마지막 한 옥텟(바이트)를 먼저 가져가버림
  - 다시 말 해 `MSG_OOB`는 1바이트, `SO_OOBINLINE`은 길이 상관 없이 순서대로
  - 이러한 구현 과정에서 middleboxes들이 urgent indication을 없앱버리게끔 구현해버림
  - 그럼 걍 URG 자체를 지양하자!


#### TCP Segment

- TCP segment format
  - Source Port (16bits), Destination Port (16bits)
  - Sequence Number - 랜덤
  - Acknowledgement Number - (expected) next sequence number
  - Data Offset (4bits, units of 4 octets)
  - Control Bits -URG, ACK, PSH, RST, SYN, FIN
  - Window (16bits)  
  receiver 입장에서 acknowledgement field 이후부터 데이터를 얼마나 받을 수 있을지 sender에게 알리기 위한 필드
  - Checksum
  - Urgent Point
  - TCP options
    - Case 1: A single octet options (Type only)
      - End of options list (Type:0), No-Operation option (Type:1)
    - Case 2: Type-length-Value style option
      - Maximum Segment Size (MSS) (Type:2, Length:4)
        - 무조건 SYN일 때에만 존재하는 옵션
        - IP layer에 존재
        - MMS_S, MMS_R, MSS가 뭔지는 ppt를 읽어보렴

#### TCP Variables

- TCP variables for a connection
  - TCP 연결에서 기억하고 있어야 할 몇가지 변수들
  - TCB에 저장되어 있음
    - local, remote socket numbers
    - 연결 보안, 프로시저
    - 버퍼 포인터
    - retransmit queue와 최근 segment의 포인터
  - Send variables
    - SND.UNA: send unacknowledged
    - SND.NXT: send next
    - SND.WND: send window
    - SND.UP: send urgent pointer
    - SND.WL1: segment sequence number used for last window update
    - SND.WL2: segment acknowledgment number used for last window update
    - ISS: initial send sequence number
  - Receive variables
    - RCV.NXT: receive next
    - RCV.WND: receive window
    - RCV.UP: receive urgent pointer
    - IRS: initial receive sequence number
    <img src="resources/TCB.png" title="TCB" alt="TCB"></img>
  - Segment variables
    - SEG.SEQ - segment sequence number
      - first sequence number of the receiving TCP segment
    - SEG.ACK - segment acknowledgment number
      - acknowledgment from the receiving TCP segment
    - SEG.LEN - segment length
      - the data size in the receiving TCP segment.
      - Total length – IP header size (IHL*4) – TCP header Size (DataOffset*4).
      - Typically, Total length–20-20.
    - SEG.WND - segment window
      - the tx buffer position for data to be sent : SEG.ACK+SEG.WND
    - SEG.UP - segment urgent pointer (안 중요)
    - SEG.PRC - segment precedence value (안 중요)
  - TCP와 segment variables
    - 새 acknowledgement
      - SND.UNA < SEG.ACK <= SND.NXT (받을 ack num이 이미 보낸 것보단 크고 보낼 것보다 작거나 같아야 함)
    - 아래 조건 중 하나를 만족하면 segment가 수신됨
      - RCV.NXT <= SEG.SEQ < RCV.NXT+RCV.WND
      - RCV.NXT <= SEG.SEQ+SEG.LEN-1 < RCV.NXT+RCV.WND
      <img src="resources/TCP_seg.png" title="TCP and segment variable" alt="TCP and segment variable"></img>

#### Sequence Numbers

- 범위: 0 ~ 2^32 - 1
- 한 데이터는 SYN을 시작 옥텟, FIN을 끝 옥텟으로 하여 보내진다고 생각하도록 sequence number가 정해짐
- Initial Sequence Number Selection
  - Initial Sequence Number (ISN) generator
    - clock-driven으로 정해짐
    - 32bit clock는 4ms마다 비트가 올라가고, ISN 사이클은 4.55시간이 됨
    - 3-way hand shake (SYN -> SYN+ACK -> ACK)에서 다음과 같이 seq num이 정해짐
    <img src="resources/TCP_Seq.png" title="Sequence Number in 3-way Hand Shake" alt="Sequence Number in 3-way Hand Shake"></img>
  - TCP Quiet Time Concept (Time wait 상태에서)
    - 이전 연결에서 마지막의 seq num이랑 다음 연결에서 새로 만들어지는 seq num이 겹치면서 발생할 수 있는 문제를 해결하기 위해 time wait가 중요

#### Retransmission Timeout (for Reliablility)

- RTO = Retransmission Time Out
- RTT = RoundTripTime, 두 종단 간 패킷 전송에 필요한 시간
  - Retransmission Timeout Procedure: Smoothed average
    - 기존 RTT에 새 RTT를 일정 가중치만큼 더해줘서 새 평균 RTT를 정한 뒤, LBOUND(1s)와 UBOUND(60s) 사이의 적당한 값으로 정함
  - 그런데 이걸로는 부족. 그래서 mean과 variation을 사용하는 Jacobson's method 등장
    - 첫 RTT에 대해서 - SRTT는 RTT 그대로, MDEV는 RTT/2로 해서 RTO 계산
    - 여러 RTT에 대해서 - ERR를 RTT-SRTT로 하고 새 SRTT는 STRR + gERR로 결정, MDEV는 MDEV + h(|ERR|-MDEV)로 결정하여 RTO 계산
    - 이걸 잘 어케 shift 이용해서 fast하게 계산 가능
  - 그런데 또 이게 ERR < 0일 때에는 개오바가 돼버려서 이 땐 h 값을 줄임 (Eifel Algorithm)
- RTT sample을 구하는 데에는 **Karn's algorithm** 사용
  - segment가 재전송된 경우에는 RTT 샘플을 만들면 안 됨
- 결론적인 재전송 타이머:
  1. 타이머가 꺼져 있으면 새로 실행
  1. 모든 데이터가 ack되면 타이머 종료
  1. 새로 ack 메세지가 오면 타이머 재시작(더 받을 게 있다는 뜻)
  - 타이머가 결국 만료된 상황
  1. 타이머가 만료되면(즉 시간 안에 답을 못 받았으면), ack이 안 된 segment부터 재전송
  1. RTO = RTO * 2로 세팅
  1. 타이머 재시작


#### Managing the Window (for Flow Control)

- receiver는 window를 줄일 수 없다 (SND.UNA+SND.WND <= SEG.ACK+SEG.WND)
- window가 0으로 줄면(더 이상 못 보내게 되면), sender는 반드시 window를 늘리도록 probe
  - SND.UNA+SND.WND < SND.NXT이면 window가 0
  - sender는 2분마다 적어도 한 옥텟 이상을 새로 보냄으로써 window를 늘리길 시도하는데, 이 때 보내는 segment가 TCP probe
  - receiver는 window가 0이 되고 나서 segment를 받더라도, expected seq num과 cur window (0)을 보이기 위해 ack는 계속 보내야 함
    - RCV.NXT = RCV.NXT+RCV.WND, RCV.WND = 0
  - sender는 결론적으로 다시 보내야 하는 부분부터 segment를 repackage해서 보냄
- Silly Window Syndrome
  - 버퍼에서 아주 작은 window를 allocate
  - Nagle Algorithm
    - sender에서 아무리 작은 segment를 보낸다 해도 첫번째 segment를 보낸 다음 ACK가 오거나 MSS만큼 누적될 때까지 기다려야 함 (대신 끌 수 있긴 함)
- TCP sender가 데이터를 보낼 수 있는 상황
  1. full sized segment가 사용 가능한 윈도우보다 작아서 보낼 수 있다
  1. 이미 보낼 데이터가 다 보내졌고 윈도우보다 보낼 데이터가 작다
  1. 윈도우의 반보다 큰 데이터가 있으면 윈도우의 반만큼만 보내라
  1. Nagle 때문에 0.1 - 1.0초가 딜레이되면 그냥 보내라
- delayed ACK
  - ACK를 맨날 보내지 말고 적당히 하나씩 빼서 보내면 효율 증가
  - FTP처럼 full-sized segments이 계속 오면 ACK를 하나 건너 하나씩은 보내자
