# 1. OSI 7계층

## 핵심 개념

OSI 7계층은 네트워크 통신을 7개의 계층으로 나눈 표준 모델입니다. 각 계층은 독립적인 역할을 수행하며, 하위 계층의 서비스를 이용해 상위 계층에 서비스를 제공합니다.

### 계층 구조

1. **응용 계층 (Application Layer)**: 사용자와 직접 상호작용하는 계층
    - HTTP, HTTPS, FTP, SMTP 등의 프로토콜
    - e.g.) Spring에서 작성한 REST API, 외부 API 호출
2. **표현 계층 (Presentation Layer)**: 데이터 형식 변환, 암호화/복호화
    - 데이터 인코딩, 압축
    - e.g.) JSON 직렬화/역직렬화, SSL/TLS 암호화
3. **세션 계층 (Session Layer)**: 통신 세션 관리
    - 연결 설정, 유지, 종료
    - e.g.) HTTP 세션, 로그인 세션 관리
4. **전송 계층 (Transport Layer)**: 종단 간 신뢰성 있는 데이터 전송
    - TCP, UDP 프로토콜
    - e.g.) DB 커넥션, HTTP 통신의 TCP 연결
5. **네트워크 계층 (Network Layer)**: 패킷 라우팅 및 논리 주소 지정
    - IP 프로토콜, 라우터
    - e.g.) 서버 IP 주소, 외부 API 서버로의 라우팅
6. **데이터 링크 계층 (Data Link Layer)**: 물리적 연결의 신뢰성 확보
    - MAC 주소, 이더넷
    - e.g.) 네트워크 인터페이스 카드(NIC)
7. **물리 계층 (Physical Layer)**: 물리적 연결 및 비트 전송
    - 케이블, 허브, 전기 신호
    - e.g.) 서버 랙의 물리적 네트워크 케이블

### Spring 백엔드 실무 연결

Spring으로 REST API를 개발할 때의 과정을 OSI 7계층으로 본다면

```text
클라이언트 요청: GET /api/users/1

(7) 응용 계층: HTTP 프로토콜로 요청 생성
(6) 표현 계층: JSON 데이터 직렬화
(5) 세션 계층: HTTP 세션 관리
(4) 전송 계층: TCP 연결 수립 (3-way handshake)
(3) 네트워크 계층: 서버 IP 주소로 패킷 라우팅
(2) 데이터 링크 계층: MAC 주소로 프레임 전송
(1) 물리 계층: 전기 신호로 비트 전송

Spring 서버에서는 역순으로 처리됨
```

### 모의면접 질문

#### Q1. OSI 7계층 모델이 무엇이고, 왜 계층을 나누었나요?

A: OSI 7계층은 네트워크 통신 과정을 7개의 독립적인 계층으로 나눈 표준 모델입니다. 각 계층을 나눈 이유는 다음과 같습니다.

첫째, 복잡한 네트워크 통신을 계층별로 분리하여 이해하기 쉽게 만듭니다. 둘째, 특정 계층에 문제가 발생했을 때 해당 계층만 수정하면 되므로 유지보수가 용이합니다. 셋째, 각 계층이 독립적이므로 특정 계층의 프로토콜을 변경해도 다른 계층에 영향을 주지 않습니다.

예를 들어, HTTP에서 HTTPS로 변경할 때 응용 계층만 수정하면 되고, 하위 계층인 TCP는 그대로 사용할 수 있습니다.

#### Q2. Spring으로 REST API를 개발할 때 주로 어느 계층에서 작업하나요?

A: 주로 7계층인 응용 계층에서 작업합니다. Spring의 `@RestController`에서 HTTP 요청을 받고 응답을 반환하는 것이 응용 계층의 일입니다.

하지만 실무적으로는 여러 계층을 이해해야 합니다. 예를 들어, DB 커넥션 풀을 설정할 때는 전송 계층의 TCP 연결을 다루고, 외부 API 호출 시 타임아웃을 설정할 때도 전송 계층의 동작을 이해해야 합니다. 또한 HTTPS를 설정할 때는 표현 계층의 SSL/TLS를 다룹니다.

#### Q3. 외부 결제 API를 호출하는데 타임아웃이 발생했습니다. 어느 계층에서 문제가 발생한 것일까요?

A: 타임아웃은 주로 전송 계층(TCP)이나 응용 계층(HTTP)에서 발생합니다.

전송 계층에서는 TCP 연결 자체가 수립되지 않거나, 패킷이 전송 중 손실되어 재전송이 반복되는 경우 타임아웃이 발생할 수 있습니다. 응용 계층에서는 외부 API 서버가 응답을 생성하는데 시간이 오래 걸려 설정된 read timeout을 초과하는 경우입니다.

Spring에서 `RestTemplate`이나 `WebClient`를 사용할 때 connectionTimeout(연결 타임아웃)과 readTimeout(읽기 타임아웃)을 구분해서 설정하는 이유가 이 때문입니다.

---
# 2. TCP/IP의 개념

## 핵심 개념

TCP/IP는 인터넷에서 실제로 사용되는 프로토콜 모음입니다. OSI 7계층보다 단순한 4계층 구조로 되어 있습니다.

### TCP/IP 4계층

1. **응용 계층 (Application Layer)**: HTTP, FTP, SMTP 등
    - OSI의 응용, 표현, 세션 계층을 통합
2. **전송 계층 (Transport Layer)**: TCP, UDP
    - OSI의 전송 계층과 동일
3. **인터넷 계층 (Internet Layer)**: IP, ICMP, ARP
    - OSI의 네트워크 계층과 유사
4. **네트워크 인터페이스 계층 (Network Interface Layer)**: 이더넷, Wi-Fi
    - OSI의 데이터 링크, 물리 계층을 통합

### Spring 백엔드 실무 연결

외부 API를 호출할 때

```java
// RestTemplate으로 외부 API 호출
RestTemplate restTemplate = new RestTemplate();
String result = restTemplate.getForObject(
    "https://api.payment.com/charge", 
    String.class
);
```

이 과정에서:

- **응용 계층**: HTTP 프로토콜로 GET 요청
- **전송 계층**: TCP를 통해 신뢰성 있는 데이터 전송
- **인터넷 계층**: IP 주소로 패킷 라우팅
- **네트워크 인터페이스 계층**: 물리적 네트워크를 통해 전송

### 모의면접 질문

#### Q1. OSI 7계층과 TCP/IP 4계층의 차이는 무엇인가요?

A: OSI 7계층은 이론적인 표준 모델이고, TCP/IP 4계층은 실제 인터넷에서 사용되는 프로토콜 모음입니다.

가장 큰 차이는 계층 수입니다. OSI는 7계층으로 세분화되어 있지만, TCP/IP는 4계층으로 더 단순합니다. 예를 들어, OSI의 응용, 표현, 세션 계층이 TCP/IP에서는 응용 계층 하나로 통합되어 있습니다.

실무에서는 TCP/IP 모델이 더 자주 사용됩니다. 우리가 Spring에서 HTTP API를 개발하거나, DB에 TCP로 연결하는 것이 모두 TCP/IP 모델을 따릅니다.

#### Q2. Spring 애플리케이션에서 데이터베이스에 연결할 때 TCP/IP의 어떤 계층을 사용하나요?

A: 모든 계층을 사용하지만, 특히 전송 계층의 TCP가 중요합니다.

```java
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
```

이 설정에서:

- 응용 계층: MySQL 프로토콜
- 전송 계층: TCP (포트 3306)
- 인터넷 계층: IP 주소 (localhost = 127.0.0.1)
- 네트워크 인터페이스 계층: 로컬 루프백 인터페이스

MySQL은 신뢰성 있는 데이터 전송이 필요하므로 TCP를 사용합니다. HikariCP 같은 커넥션 풀도 TCP 연결을 관리하는 것입니다.

#### Q3. TCP/IP에서 IP의 역할은 무엇인가요?

A: IP(Internet Protocol)는 인터넷 계층에서 패킷을 목적지까지 전달하는 역할을 합니다. IP 주소를 사용해 네트워크 상에서 각 장치를 식별하고, 라우팅을 통해 패킷을 목적지로 보냅니다.

하지만 IP는 비신뢰성 프로토콜입니다. 패킷이 제대로 도착했는지, 순서가 맞는지 보장하지 않습니다. 그래서 그 위의 전송 계층에서 TCP나 UDP를 사용해 신뢰성을 추가하거나 빠른 전송을 선택합니다.

실무에서 외부 API 서버의 IP 주소나, `application.yml`에 설정하는 Redis 서버 IP(예: 192.168.1.100) 등이 모두 IP의 사용 예시입니다.

---

# 3. TCP와 UDP

## 핵심 개념

TCP와 UDP는 모두 전송 계층의 프로토콜이지만, 특성이 매우 다릅니다.

**TCP (Transmission Control Protocol)**

- 연결 지향적: 3-way handshake로 연결 수립
- 신뢰성 보장: 패킷 손실 시 재전송
- 순서 보장: 패킷이 순서대로 도착
- 흐름 제어 및 혼잡 제어
- 속도가 상대적으로 느림
- 사용 예시: HTTP, HTTPS, FTP, SMTP, MySQL

**UDP (User Datagram Protocol)**

- 비연결성: 연결 수립 과정 없음
- 신뢰성 미보장: 패킷 손실해도 재전송 안함
- 순서 미보장: 패킷이 순서 없이 도착 가능
- 흐름/혼잡 제어 없음
- 속도가 빠르고 오버헤드가 적음
- 사용 예시: DNS, 실시간 스트리밍, 온라인 게임

### Spring 백엔드 실무 연결

**TCP 사용 사례**

```java
// REST API - HTTP는 TCP 기반
@GetMapping("/api/orders/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderService.findById(id);
}

// 데이터베이스 연결 - JDBC는 TCP 기반
@Autowired
private DataSource dataSource;
```

주문 데이터나 고객 정보처럼 정확성이 중요한 데이터는 반드시 TCP를 사용합니다.

UDP는 주로 실시간성이 중요한 게임 서버나 DNS 조회에서 사용됩니다.

### 비교표

|특성|TCP|UDP|
|---|---|---|
|연결 방식|연결 지향 (3-way handshake)|비연결성|
|신뢰성|높음 (재전송)|낮음 (재전송 없음)|
|순서 보장|보장함|보장 안함|
|속도|상대적으로 느림|빠름|
|헤더 크기|20바이트 이상|8바이트 고정|
|실무 사용|REST API, DB, 외부 API|DNS, 스트리밍|

### 모의면접 질문

#### Q1. TCP와 UDP의 가장 큰 차이점은 무엇인가요?

A: 가장 큰 차이점은 신뢰성입니다.

TCP는 연결을 수립하고, 데이터가 제대로 전송되었는지 확인하며, 손실된 패킷은 재전송합니다. 순서도 보장합니다. 반면 UDP는 연결 수립 없이 데이터를 보내기만 하고, 도착 여부나 순서를 확인하지 않습니다.

실무적으로 설명하면, REST API로 주문을 처리할 때는 주문 데이터가 정확히 전달되어야 하므로 TCP를 사용합니다. 하지만 실시간 채팅의 "상대방이 입력 중입니다" 같은 메시지는 일부 손실되어도 큰 문제가 없으므로 UDP를 사용할 수 있습니다.

#### Q2. 왜 REST API는 TCP를 사용하나요? UDP를 사용하면 더 빠르지 않나요?

A: REST API에서는 데이터의 정확성과 완전성이 속도보다 중요하기 때문입니다.

예를 들어, 사용자가 상품을 주문하는 API를 호출했는데, 주문 데이터의 일부가 손실되면 큰 문제가 됩니다. 가격 정보나 배송 주소가 빠지면 거래가 무효화됩니다. TCP는 재전송과 순서 보장을 통해 이런 문제를 방지합니다.

또한 HTTP 프로토콜 자체가 TCP 위에서 동작하도록 설계되어 있습니다. HTTP 요청과 응답의 헤더, 바디가 순서대로 완전히 전달되어야 의미가 있기 때문입니다.

속도가 조금 느리더라도 정확한 거래 처리가 더 중요한 것이 비즈니스 로직입니다.

#### Q3. 데이터베이스 연결은 TCP와 UDP 중 무엇을 사용하나요? 그 이유는?

A: 데이터베이스 연결은 TCP를 사용합니다.

데이터베이스는 데이터의 정확성과 일관성이 가장 중요합니다. INSERT, UPDATE, DELETE 같은 쿼리가 일부만 전달되거나 순서가 바뀌면 데이터베이스가 손상될 수 있습니다. 트랜잭션 처리에서도 SQL 명령어가 정확한 순서로 실행되어야 합니다.

예를 들어

```java
@Transactional
public void transferMoney(Long fromId, Long toId, BigDecimal amount) {
    accountRepository.decreaseBalance(fromId, amount);
    accountRepository.increaseBalance(toId, amount);
}
```

이 계좌이체 로직에서 첫 번째 쿼리만 실행되고 두 번째가 손실되면 돈이 사라집니다. TCP의 신뢰성 보장이 반드시 필요한 이유입니다.

#### Q4. 외부 API 호출 시 타임아웃을 설정하는 이유는 무엇인가요? TCP와 어떤 관련이 있나요?

A: 타임아웃을 설정하는 이유는 무한정 응답을 기다리지 않기 위해서입니다.

TCP는 신뢰성을 보장하기 위해 재전송을 시도합니다. 만약 외부 API 서버가 응답이 느리거나 네트워크에 문제가 있으면, TCP는 계속 재시도하며 연결을 유지하려고 합니다. 타임아웃이 없으면 Spring 애플리케이션의 스레드가 무한정 대기하게 되어 시스템이 멈출 수 있습니다.

```java
RestTemplate restTemplate = new RestTemplate();
HttpComponentsClientHttpRequestFactory factory = 
    new HttpComponentsClientHttpRequestFactory();
factory.setConnectTimeout(3000);  // 연결 타임아웃 3초
factory.setReadTimeout(5000);     // 읽기 타임아웃 5초
restTemplate.setRequestFactory(factory);
```

connectTimeout은 TCP 연결 수립 시간 제한이고, readTimeout은 데이터를 받는 시간 제한입니다. 이렇게 설정하면 시스템이 무한 대기에 빠지는 것을 방지할 수 있습니다.

---

# 4. TCP와 UDP의 헤더 분석

## 핵심 개념

헤더는 데이터를 전송할 때 함께 보내는 제어 정보입니다. TCP와 UDP의 헤더 구조를 보면 왜 TCP가 더 복잡하고 신뢰성이 높은지 알 수 있습니다.

### TCP 헤더 (최소 20바이트)

주요 필드
- **Source Port (2바이트)**: 출발지 포트 번호
- **Destination Port (2바이트)**: 목적지 포트 번호
- **Sequence Number (4바이트)**: 전송하는 데이터의 순서 번호
- **Acknowledgment Number (4바이트)**: 받은 데이터 확인 번호
- **Flags (1바이트)**: SYN, ACK, FIN 등 제어 플래그
- **Window Size (2바이트)**: 수신 가능한 데이터 크기 (흐름 제어)
- **Checksum (2바이트)**: 오류 검출

### UDP 헤더 (고정 8바이트)

주요 필드
- **Source Port (2바이트)**: 출발지 포트 번호
- **Destination Port (2바이트)**: 목적지 포트 번호
- **Length (2바이트)**: 헤더 + 데이터의 전체 길이
- **Checksum (2바이트)**: 오류 검출

### Spring 백엔드 실무 연결

REST API를 개발할 때, 클라이언트와 서버 간 통신에서 TCP 헤더가 사용됩니다.

```java
// 클라이언트가 API 호출
GET /api/users/1 HTTP/1.1
Host: localhost:8080

// TCP 헤더에서 중요한 정보:
// - Destination Port: 8080 (Spring 서버 포트)
// - Sequence Number: 데이터 순서 보장
// - ACK Flag: 데이터 수신 확인
```

### 모의면접 질문

#### Q1. TCP 헤더에서 Sequence Number와 Acknowledgment Number의 역할은 무엇인가요?

A: 두 필드는 TCP의 신뢰성과 순서 보장을 위해 사용됩니다.

Sequence Number는 송신자가 보내는 데이터의 순서를 나타냅니다. 각 바이트마다 번호가 매겨지므로 수신자는 데이터가 어떤 순서로 왔는지 알 수 있습니다.

Acknowledgment Number는 수신자가 "여기까지 데이터를 잘 받았습니다"라고 송신자에게 알려주는 번호입니다. 이를 통해 송신자는 어떤 데이터가 전달되었는지 확인할 수 있습니다.

실무 예시로, 클라이언트가 대용량 파일을 업로드할 때 네트워크가 불안정하면 일부 패킷이 손실될 수 있습니다. TCP는 Sequence Number로 어떤 부분이 손실되었는지 파악하고, Acknowledgment Number로 재전송 여부를 결정합니다.

#### Q2. TCP 헤더의 Flags 필드에 있는 SYN, ACK, FIN은 무엇인가요?

A: 이 플래그들은 TCP 연결의 상태를 제어합니다.

- **SYN (Synchronize)**: 연결을 시작할 때 사용합니다. "연결하고 싶어요"라는 의미입니다.
- **ACK (Acknowledgment)**: 데이터를 받았음을 확인할 때 사용합니다. "잘 받았어요"라는 의미입니다.
- **FIN (Finish)**: 연결을 종료할 때 사용합니다. "통신을 끝내고 싶어요"라는 의미입니다.

실무에서 이 플래그들은 3-way handshake(SYN, SYN-ACK, ACK)와 4-way handshake(FIN, ACK, FIN, ACK)에서 사용됩니다.

예를 들어, 외부 API를 호출하고 응답을 받은 후, 연결을 종료할 때 FIN 플래그가 사용됩니다.

#### Q3. UDP 헤더가 TCP보다 훨씬 단순한 이유는 무엇인가요?

A: UDP는 신뢰성 보장 기능이 없기 때문입니다.

TCP 헤더에는 Sequence Number, Acknowledgment Number, Window Size 같은 필드들이 있어서 순서 보장, 재전송, 흐름 제어를 수행합니다. 반면 UDP는 이런 기능이 필요 없으므로 헤더가 단순합니다.

UDP는 출발지 포트, 목적지 포트, 데이터 길이, 체크섬만 있으면 됩니다. "데이터를 보낼 테니 받든 말든 알아서 해"라는 방식입니다.

이런 단순함 때문에 UDP는 헤더 오버헤드가 적고(8바이트) 처리 속도가 빠릅니다. TCP는 최소 20바이트 이상이고 추가 제어 작업이 필요해 상대적으로 느립니다.

#### Q4. Spring 애플리케이션의 포트 번호는 TCP 헤더의 어디에 들어가나요?

A: Spring 애플리케이션의 포트 번호는 Destination Port 필드에 들어갑니다.

```yaml
server:
  port: 8080
```

이렇게 설정하면, 클라이언트가 API를 호출할 때 TCP 헤더의 Destination Port에 8080이 들어갑니다. 서버는 이 포트 번호를 보고 어떤 애플리케이션으로 패킷을 전달할지 결정합니다.

반대로 서버가 응답을 보낼 때는 Source Port가 8080이 됩니다. 이처럼 포트 번호는 한 서버에서 여러 애플리케이션이 동시에 실행될 때 트래픽을 구분하는 역할을 합니다.

만약 같은 서버에서 Spring(8080), PostgreSQL(5432), Redis(6379)가 함께 실행되면, 각 포트 번호로 트래픽이 정확히 라우팅됩니다.

---

# 5. TCP의 3-way Handshake와 4-way Handshake

## 핵심 개념

### 3-way Handshake (연결 수립)

TCP 연결을 시작할 때 수행하는 과정입니다.

```
클라이언트 → 서버: SYN (연결 요청)
서버 → 클라이언트: SYN-ACK (요청 수락 및 연결 준비)
클라이언트 → 서버: ACK (연결 확립 완료)
```

1. **1단계 (SYN)**: 클라이언트가 서버에 연결을 요청합니다.
    - 클라이언트는 SYN 플래그를 1로 설정하고, 초기 Sequence Number를 보냅니다.
2. **2단계 (SYN-ACK)**: 서버가 요청을 수락하고 준비되었음을 알립니다.
    - 서버는 SYN과 ACK 플래그를 모두 1로 설정합니다.
    - 서버도 자신의 초기 Sequence Number를 보냅니다.
3. **3단계 (ACK)**: 클라이언트가 최종 확인을 보냅니다.
    - ACK 플래그를 1로 설정하여 연결이 확립되었음을 알립니다.

이 과정이 끝나면 양방향 데이터 전송이 가능해집니다.

### 4-way Handshake (연결 종료)

TCP 연결을 종료할 때 수행하는 과정입니다.

```
클라이언트 → 서버: FIN (연결 종료 요청)
서버 → 클라이언트: ACK (종료 요청 확인)
서버 → 클라이언트: FIN (서버도 종료 준비 완료)
클라이언트 → 서버: ACK (최종 종료 확인)
```

1. **1단계 (FIN)**: 클라이언트가 연결 종료를 요청합니다.
    - FIN 플래그를 1로 설정합니다.
2. **2단계 (ACK)**: 서버가 종료 요청을 받았음을 확인합니다.
    - 하지만 서버는 아직 보낼 데이터가 있을 수 있어 바로 종료하지 않습니다.
3. **3단계 (FIN)**: 서버가 모든 데이터 전송을 완료하고 종료 준비가 되었음을 알립니다.
    - FIN 플래그를 1로 설정합니다.
4. **4단계 (ACK)**: 클라이언트가 최종 확인을 보내고 연결이 완전히 종료됩니다.

### Spring 백엔드 실무 연결

**REST API 호출 과정**

```java
// 클라이언트가 API 호출
RestTemplate restTemplate = new RestTemplate();
String result = restTemplate.getForObject(
    "http://localhost:8080/api/users/1", 
    String.class
);
```

이 코드 실행 시 내부에서 일어나는 일:

1. **3-way Handshake**: TCP 연결 수립
    - 클라이언트 → Spring 서버: SYN
    - Spring 서버 → 클라이언트: SYN-ACK
    - 클라이언트 → Spring 서버: ACK
    - 이제 HTTP 요청을 보낼 수 있음
2. **HTTP 요청/응답**: 데이터 전송
    - 클라이언트 → Spring 서버: GET /api/users/1
    - Spring 서버 → 클라이언트: HTTP 200 OK + 사용자 데이터
3. **4-way Handshake**: TCP 연결 종료
    - 클라이언트 → Spring 서버: FIN
    - Spring 서버 → 클라이언트: ACK
    - Spring 서버 → 클라이언트: FIN
    - 클라이언트 → Spring 서버: ACK

**데이터베이스 커넥션 풀**

```java
// HikariCP 설정
spring.datasource.hikaricp.maximum-pool-size=10
```

커넥션 풀은 TCP 연결을 미리 수립해서 재사용합니다. 매번 3-way handshake를 하면 시간이 오래 걸리기 때문에, 연결을 유지하고 재사용하는 것입니다.

### 모의면접 질문

#### Q1. 3-way Handshake가 필요한 이유는 무엇인가요? 2-way로는 안 되나요?

A: 3-way Handshake는 양방향 통신을 안전하게 수립하기 위해 필요합니다.

만약 2-way로 한다면, 클라이언트가 SYN을 보내고 서버가 SYN-ACK를 보낸 후 바로 연결이 수립됩니다. 하지만 이 경우 클라이언트가 서버의 SYN-ACK를 받았는지 서버가 확인할 수 없습니다.

예를 들어, 네트워크 지연으로 서버의 SYN-ACK가 늦게 도착하면, 클라이언트는 이미 포기하고 연결을 끊었을 수 있습니다. 하지만 서버는 연결이 수립된 줄 알고 자원을 할당하게 됩니다. 이런 상황이 반복되면 서버 자원이 낭비됩니다.

3-way Handshake의 마지막 ACK는 "나 준비됐어, 이제 데이터 보내도 돼"라는 확인입니다. 이를 통해 양쪽 모두 통신 준비가 되었음을 확실히 합니다.

#### Q2. 데이터베이스 커넥션 풀이 성능에 좋은 이유를 3-way Handshake와 연결지어 설명해주세요.

A: 커넥션 풀은 TCP 연결을 재사용하여 3-way Handshake 오버헤드를 줄입니다.

매번 쿼리를 실행할 때마다 새로운 DB 연결을 맺으면

1. 3-way Handshake로 TCP 연결 수립 (3번의 패킷 교환)
2. 쿼리 실행
3. 4-way Handshake로 연결 종료 (4번의 패킷 교환)

이 과정에서 총 7번의 패킷 교환이 필요합니다. API 하나에서 10개의 쿼리를 실행하면 70번의 패킷 교환이 발생합니다.

커넥션 풀을 사용하면

1. 처음 한 번만 3-way Handshake로 연결 수립
2. 연결을 유지한 채로 여러 쿼리 실행
3. 필요 없을 때만 4-way Handshake로 종료

HikariCP 같은 커넥션 풀은 10개의 연결을 미리 맺어두고 재사용하므로, 매번 Handshake 비용을 지불하지 않아도 됩니다.

```java
// 커넥션 풀 없이
for (int i = 0; i < 100; i++) {
    // 매번 3-way + 4-way = 성능 저하
}

// 커넥션 풀 사용
for (int i = 0; i < 100; i++) {
    // 연결 재사용 = 빠른 성능
}
```

#### Q3. 4-way Handshake에서 FIN을 두 번 보내는 이유는 무엇인가요?

A: 양방향 연결을 각각 독립적으로 종료하기 위해서입니다.

TCP는 전이중(Full Duplex) 통신입니다. 클라이언트에서 서버로 가는 방향과, 서버에서 클라이언트로 가는 방향이 독립적으로 존재합니다.

클라이언트가 먼저 FIN을 보내면 "나는 더 이상 데이터를 보내지 않겠다"는 의미입니다. 하지만 서버는 아직 보낼 데이터가 남아있을 수 있습니다. 그래서 서버가 ACK로 확인한 후, 남은 데이터를 모두 보낸 다음에 자신의 FIN을 보냅니다.

실무 예시로, 대용량 파일 다운로드 API에서

1. 클라이언트가 파일 요청 후 FIN 전송 (더 이상 요청할 게 없음)
2. 서버는 ACK 응답 (요청은 받았어)
3. 서버는 파일 전송을 계속함
4. 파일 전송 완료 후 서버가 FIN 전송
5. 클라이언트가 ACK 응답 (완전 종료)

이렇게 4단계로 나누어져야 데이터 손실 없이 안전하게 연결을 종료할 수 있습니다.

#### Q4. 외부 API 호출 시 Connection Timeout과 Read Timeout의 차이를 3-way Handshake와 연결지어 설명해주세요.

A: Connection Timeout은 3-way Handshake 과정의 제한 시간이고, Read Timeout은 데이터 전송 시간의 제한입니다.

```java
HttpComponentsClientHttpRequestFactory factory = 
    new HttpComponentsClientHttpRequestFactory();
factory.setConnectTimeout(3000);  // Connection Timeout
factory.setReadTimeout(5000);     // Read Timeout
```

**Connection Timeout (3000ms)**

- 3-way Handshake가 3초 안에 완료되지 않으면 에러
- 서버가 다운되었거나, 방화벽에 막혀 있거나, 네트워크가 불통인 경우 발생
- SYN을 보냈는데 SYN-ACK가 오지 않는 상황

**Read Timeout (5000ms)**

- 3-way Handshake는 성공했지만, HTTP 응답을 5초 안에 받지 못하면 에러
- 서버가 처리 중이거나, 응답이 느린 경우 발생
- 연결은 수립되었지만 데이터가 오지 않는 상황

실무에서

- Connection Timeout: 보통 짧게 설정 (2-3초) - 연결 자체가 안 되면 빨리 포기
- Read Timeout: 상대적으로 길게 설정 (5-30초) - 응답 처리 시간을 고려

#### Q5. TIME_WAIT 상태가 무엇이고, 왜 발생하나요?

A: TIME_WAIT는 4-way Handshake 후 연결이 완전히 종료되기 전의 대기 상태입니다.

4-way Handshake의 마지막 ACK를 보낸 후, 클라이언트는 바로 연결을 닫지 않고 일정 시간(보통 2분) 동안 TIME_WAIT 상태로 대기합니다. 이유는 두 가지입니다:

1. **마지막 ACK가 손실될 경우 대비**: 서버가 마지막 ACK를 받지 못하면 FIN을 재전송합니다. 이때 클라이언트가 이미 종료되었다면 재전송된 FIN에 응답할 수 없습니다.
2. **지연된 패킷 처리**: 이전 연결의 패킷이 네트워크에서 지연되어 나중에 도착할 수 있습니다. 동일한 포트로 새 연결을 맺으면 이 패킷과 혼동될 수 있습니다.

실무 문제

- 외부 API를 초당 수백 번 호출하면 TIME_WAIT 상태의 소켓이 쌓여 포트 고갈 발생
- 해결책: Connection Pool 사용, Keep-Alive 활성화로 연결 재사용

```java
// Keep-Alive 설정으로 연결 재사용
HttpComponentsClientHttpRequestFactory factory = 
    new HttpComponentsClientHttpRequestFactory();
HttpClient httpClient = HttpClientBuilder.create()
    .setConnectionReuseStrategy(new DefaultConnectionReuseStrategy())
    .build();
factory.setHttpClient(httpClient);
```