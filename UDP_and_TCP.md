# UDP and TCP

## UDP

- UDP의 정의  
Packet-switched computer communication에서 datagram 단위로 전송하기 위해 만들어진 매우 간단한 프로토콜.
어플리케이션이 다른 어플리케이션에 메세지를 보내기 위해 필요한 최소한의 메커니즘만 정의하였다.
즉 transaction oriented하며 delivery와 duplicate protection은 보장하지 않는다.

- Header
    * IPv4 Protocol Number: 17
    * Source Port (16bits) -> optional
    * Destination Port (16bits)
    * Length (16bits) : 헤더와 데이터를 모두 포함한 총 길이
    * Checksum
        - optional field이며, 안 쓰이면 +0
        - 쓰이면 각 field에 대해 1의 보수로 더한 값들(carry를 버리지 않고 LSB쪽에 더해줌)의 총합의 1의 보수
        - 전송된 이후 다시 field에 대해 1의 보수로 더한 값들 + checksum을 하면 -0(0xFFFF)이 나와야 손실이 없다는 뜻


## TCP

- Motivation  
Highly reliable한 host-to-host protocol을 만들기 위해 설계됨.
(connected-oriented, end-to-end reliable)  
또한, variable-length segments 단위로 전송할 수 있도록 함.  

- 정의
    * Primary **virtual-circuit** transport protocol
    * 믿을 수 있으며, 전이중통신이 가능해야 함
    * connection-oriented transport service

- Operation
    1. 데이터 전송 (octet 단위)
        * octet 단위로 데이터를 보내면 TCP가 알아서 적당한 크기로 boxing 후 송신
        * 만약 반드시 이어서 보내야 할 데이터라면 push(PSH) flag 이용 - 버퍼에서 안 기다리고 보내기 **요청**가능
    1. 신뢰성
        * 신뢰성 보장(데이터의 파괴, 손실, 복제, 순서변경 예방)을 위해 acknowledgment(ACK)을 이용
        * ACK가 양수인 데이터를 받아야만 하는데, 시간 안에 못 받으면 retransmitted
        * sequence number로 순서 확인
        * checksum으로 데이터 파괴 확인
    1. 흐름 제어
        * 수신 호스트가 리소스를 얼마나 받을지 제어하여 손실(버퍼 오버플로우)이 없도록 함
        * window를 이용해 수용 가능한 octet 수를 sequence number의 범위 형태로 표현  
        -> 어떤 octet의 seq num이 마지막으로 받은 ack num과 window보다 작거나 같으면 octet을 보낼 수 있음
    1. 한 호스트가 여러 프로세스 감당 가능
        * 한 연결에서 socket 쌍이 \<address + port\>로 unique하게 결정
        * port는 정의상 아무거나 써도 되지만 어플리케이션마다 약속된 port를 자주 사용
    1. 데이터를 보내기 전에 **연결**을 만들어야 함 - TCB
        * clock-based seq num을 사용하는 handshake mechanism으로 연결 생성
    1. 보안
    * congestion control(혼잡 제어) 기능도 추가. But 따로 구현됨


### TCP connection management


