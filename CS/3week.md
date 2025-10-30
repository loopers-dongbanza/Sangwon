# Socket

## Socket이란?

+ Socket은 네트워크 통신의 끝점(endpoint)이다. 두 프로그램이 네트워크를 통해 데이터를 주고받기 위한 "문(door)"이라고 생각하면 된다.
+ Socket = IP 주소 + Port 번호
+ 예시: `192.168.0.10:8080`

## 왜 필요한가?

컴퓨터 한 대에서 여러 프로그램이 동시에 네트워크 통신을 한다.
- 브라우저 → 웹서버 통신
- 카카오톡 → 메시지 서버 통신
- 게임 클라이언트 → 게임 서버 통신

이때 IP 주소만으로는 어떤 프로그램과 통신할지 알 수 없다. Port 번호로 프로그램을 구분하고, Socket이 통신의 끝점 역할을 한다.

---
## Socket vs Port vs IP

### 건물 비유로 이해하기

| 개념     | 비유                | 실제 의미             | 예시                  |
| ------ | ----------------- | ----------------- | ------------------- |
| IP 주소  | 서울시 강남구 xx빌딩      | 컴퓨터(호스트) 식별       | `192.168.0.10`      |
| Port   | 201호              | 컴퓨터 내 프로그램 식별     | `8080`              |
| Socket | 서울시 강남구 xx빌딩 201호 | 통신 끝점 (IP + Port) | `192.168.0.10:8080` |

### 같은 컴퓨터, 다른 서비스

```
IP: 192.168.0.10 (같은 서버)
├─ Port 8080: Spring Boot 서버    → Socket: 192.168.0.10:8080
├─ Port 3306: MySQL 서버          → Socket: 192.168.0.10:3306
└─ Port 6379: Redis 서버          → Socket: 192.168.0.10:6379
```

- Port는 프로그램을 식별하는 번호
- Socket은 실제 통신이 일어나는 지점 (IP + Port)

---
## Socket의 종류

### 1. Stream Socket (TCP Socket)
- TCP 프로토콜 사용
- 연결 지향적, 신뢰성 보장
- 3-way handshake로 연결 수립
- Spring의 대부분 통신: HTTP, DB 연결, 외부 API 등

### 2. Datagram Socket (UDP Socket)
- UDP 프로토콜 사용
- 비연결형, 빠르지만 신뢰성 없음
- Handshake 없음
- 사용 예시: DNS 조회, 실시간 스트리밍, 온라인 게임

---
## Spring 백엔드 실무 관점

### 1. HTTP 요청/응답

```text
클라이언트 Socket: 192.168.0.100:54321 (임시 포트)
         ↓ TCP 연결
서버 Socket: 192.168.0.10:8080
```

Spring Boot 설정
```yml
server:
  port: 8080          # 서버 Socket의 포트 번호
  address: 0.0.0.0    # 모든 IP에서 접속 가능 (기본값)
```

의미
- `server.port=8080`: "Spring Boot는 8080 포트로 들어오는 요청을 처리합니다"
- 클라이언트가 요청할 때마다 임시 포트(ephemeral port) 자동 할당됨 (49152~65535 범위)

### 2. 데이터베이스 연결

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    # 3306은 MySQL 서버의 Socket 포트
```

실제 동작
- HikariCP가 TCP Socket 연결을 Connection Pool에 유지
- 3-way handshake 비용 절감 (연결 재사용)
- `localhost:3306` → MySQL 서버 Socket

### 3. 외부 API 호출

```java
RestTemplate restTemplate = new RestTemplate();
String result = restTemplate.getForObject(
    "https://api.example.com/data",
    String.class
);
// 443 포트 (HTTPS)로 Socket 연결
```

Spring이 내부적으로 하는 일
1. `api.example.com:443`으로 TCP Socket 연결
2. 3-way handshake
3. TLS/SSL handshake (HTTPS의 경우)
4. HTTP 요청 전송
5. HTTP 응답 수신
6. Socket 종료 또는 재사용

### 4. ServerSocket의 IP 주소

```java
ServerSocket serverSocket = new ServerSocket(8080);
// 실제로는: new ServerSocket(8080, 50, InetAddress.getByName("0.0.0.0"))
```

IP 주소 옵션

| 설정 | 의미 | 접속 가능 IP |
|------|------|-------------|
| `0.0.0.0` (기본) | 모든 네트워크 인터페이스 | `127.0.0.1`, `192.168.x.x`, 외부 IP |
| `127.0.0.1` | localhost만 | `127.0.0.1`만 |
| `192.168.0.10` | 특정 IP만 | `192.168.0.10`만 |

**Spring Boot 예시:**
```yaml
server:
  address: 0.0.0.0    # 모든 IP 허용 (기본값)
  # address: 127.0.0.1  # localhost만 허용
```

### 5. 한 서버에서 여러 Spring Boot 앱 실행

```yaml
# 첫 번째 앱
server.port=8080

# 두 번째 앱
server.port=8081

# 세 번째 앱
server.port=8082
```

**각각 독립적인 Socket으로 동작**

---

## Socket 프로그래밍 코드 예시

### TCP Socket Server (서버 측)

```java
import java.io.*;
import java.net.*;

public class SimpleSocketServer {
    public static void main(String[] args) {
        int port = 8080;

        try {
            // 1. ServerSocket 생성 - 8080 포트에서 대기
            ServerSocket serverSocket = new ServerSocket(port);
            System.out.println("[서버] 포트 " + port + "에서 대기 중");

            // 2. 클라이언트 연결 대기 (Blocking)
            // 이 시점에 3-way handshake 자동 발생
            Socket clientSocket = serverSocket.accept();
            System.out.println("[서버] TCP 연결 수립 완료");
            System.out.println("[서버] 클라이언트: " +
                clientSocket.getRemoteSocketAddress());

            // 3. 입력 스트림 - 클라이언트로부터 데이터 받기
            BufferedReader in = new BufferedReader(
                new InputStreamReader(clientSocket.getInputStream())
            );

            // 4. 출력 스트림 - 클라이언트에게 데이터 보내기
            PrintWriter out = new PrintWriter(
                clientSocket.getOutputStream(), true
            );

            // 5. 클라이언트 메시지 읽기
            String clientMessage = in.readLine();
            System.out.println("[서버] 받은 메시지: " + clientMessage);

            // 6. 클라이언트에게 응답 보내기
            out.println("서버 응답: 메시지 받았습니다 - " + clientMessage);
            System.out.println("[서버] 응답 전송 완료");

            // 7. 연결 종료 (4-way handshake 자동 발생)
            clientSocket.close();
            serverSocket.close();
            System.out.println("[서버] TCP 연결 종료.");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### TCP Socket Client (클라이언트 측)

```java
import java.io.*;
import java.net.*;

public class SimpleSocketClient {
    public static void main(String[] args) {
        String serverIP = "localhost";
        int port = 8080;

        try {
            // 1. Socket 생성 및 서버 연결
            // 이 시점에 3-way handshake 자동 발생
            Socket socket = new Socket(serverIP, port);
            System.out.println("[클라이언트] 서버 연결됨: " +
                socket.getRemoteSocketAddress());
            System.out.println("[클라이언트] 내 Socket: " +
                socket.getLocalSocketAddress());

            // 2. 출력 스트림 - 서버에게 데이터 보내기
            PrintWriter out = new PrintWriter(
                socket.getOutputStream(), true
            );

            // 3. 입력 스트림 - 서버로부터 데이터 받기
            BufferedReader in = new BufferedReader(
                new InputStreamReader(socket.getInputStream())
            );

            // 4. 서버에게 메시지 보내기
            out.println("안녕하세요, 서버님!");
            System.out.println("[클라이언트] 메시지 전송 완료");

            // 5. 서버 응답 받기
            String serverResponse = in.readLine();
            System.out.println("[클라이언트] 서버 응답: " + serverResponse);

            // 6. 연결 종료 (4-way handshake 자동 발생)
            socket.close();
            System.out.println("[클라이언트] TCP 연결 종료!");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 코드 분석 - 핵심 개념

#### 1. ServerSocket vs Socket

```java
ServerSocket serverSocket = new ServerSocket(8080);  // 서버용
Socket clientSocket = serverSocket.accept();         // 클라이언트 연결 수락
```

| 클래스              | 역할     | 용도                |
| ---------------- | ------ | ----------------- |
| **ServerSocket** | 연결 대기  | 클라이언트 연결을 기다리는 용도 |
| **Socket**       | 데이터 통신 | 실제 데이터를 송수신하는 끝점  |

동작 과정
1. ServerSocket: "8080 포트로 연결 기다림" (LISTEN 상태)
2. 클라이언트 연결 요청 들어옴
3. `accept()` 메서드가 새로운 Socket 객체 생성
4. 이 Socket으로 클라이언트와 통신

#### 2. 양방향 통신

```java
// 데이터 받기 (InputStream)
BufferedReader in = new BufferedReader(
    new InputStreamReader(socket.getInputStream())
);

// 데이터 보내기 (OutputStream)
PrintWriter out = new PrintWriter(
    socket.getOutputStream(), true
);
```

Socket은 양방향(Full-Duplex) 통신
- `InputStream`: 상대방이 보낸 데이터 읽기
- `OutputStream`: 상대방에게 데이터 보내기
- 동시에 읽고 쓰기 가능

#### 3. 임시 포트 (Ephemeral Port)

```java
Socket socket = new Socket("localhost", 8080);
System.out.println(socket.getLocalPort());  // 54321 (예시)
```

**왜 임시 포트인가?**
- 클라이언트는 요청만 하면 끝 → 포트를 계속 점유할 필요 없음
- OS가 자동으로 사용 가능한 포트 할당 (49152~65535 범위)
- 다음 요청 시 다른 포트 사용

서버는 고정 포트
```
서버: "나는 항상 8080에 있어. 필요하면 여기로 와!"
클라이언트들: "알겠어!" (각자 임시 포트로 접속)
```

---
## TCP 연결 수립 과정

### TCP 연결 vs 데이터 교환

| 단계     | 코드                                | 실제 동작               | 레벨              |
| ------ | --------------------------------- | ------------------- | --------------- |
| 연결 수립  | `new Socket()` / `accept()`       | TCP 3-way handshake | **전송 계층 (TCP)** |
| 데이터 교환 | `out.println()` / `in.readLine()` | TCP로 데이터 송수신        | **애플리케이션 계층**   |
| 연결 종료  | `socket.close()`                  | TCP 4-way handshake | **전송 계층 (TCP)** |
### 전체 통신 흐름

```text
┌─────────────────────────────────────────────────────────────┐
│ TCP 연결 수립 (3-way handshake) - 자동 발생              
└─────────────────────────────────────────────────────────────┘

서버: ServerSocket(8080) 생성 → LISTEN 상태
서버: accept() 호출 → 연결 대기 중...

클라이언트: new Socket("localhost", 8080) 호출

    [TCP 3-way handshake 시작]

    클라이언트 → 서버: SYN
    서버 → 클라이언트: SYN-ACK
    클라이언트 → 서버: ACK

    [연결 수립 완료! ESTABLISHED 상태]

서버: accept() 반환 → Socket 객체 생성
클라이언트: Socket 객체 생성 완료

┌─────────────────────────────────────────────────────────────┐
│ 애플리케이션 데이터 교환 - 개발자가 직접 작성            
└─────────────────────────────────────────────────────────────┘

// 이미 TCP 연결은 수립된 상태!
// 이제 애플리케이션 데이터를 주고받음

클라이언트: out.println("안녕하세요!")
    → TCP Segment로 패킹하여 전송

서버: in.readLine()
    → "안녕하세요!" 받음

서버: out.println("응답합니다!")
    → TCP Segment로 패킹하여 전송

클라이언트: in.readLine()
    → "응답합니다!" 받음

┌─────────────────────────────────────────────────────────────┐
│ TCP 연결 종료 (4-way handshake) - 자동 발생              
└─────────────────────────────────────────────────────────────┘

클라이언트: socket.close() 호출

    [TCP 4-way handshake 시작]

    클라이언트 → 서버: FIN
    서버 → 클라이언트: ACK
    서버 → 클라이언트: FIN
    클라이언트 → 서버: ACK

    [연결 종료 완료!]

서버: clientSocket.close()
서버: serverSocket.close()
```

### 핵심 포인트

1. TCP 연결 수립은 `new Socket()`과 `accept()` 시점에 자동 발생
2. `PrintWriter/BufferedReader`는 이미 수립된 연결 위에서 데이터를 주고받는 것
3. 3-way handshake와 4-way handshake는 개발자가 직접 제어하지 않음

---
## Spring Boot와 Socket 비교

### 저수준 Socket 프로그래밍

```java
ServerSocket serverSocket = new ServerSocket(8080);
Socket clientSocket = serverSocket.accept();
BufferedReader in = new BufferedReader(...);
String data = in.readLine();
// 복잡한 Socket 관리, HTTP 파싱 필요...
```

### Spring Boot (추상화됨)

```java
@RestController
public class UserController {

    @GetMapping("/api/users")
    public List<User> getUsers() {
        // Socket 관리는 Spring이 알아서
        return userService.getAllUsers();
    }
}
```

### Spring Boot 내부 동작 (Tomcat)

```text
1. Tomcat이 server.port(8080)로 ServerSocket 생성 → LISTEN
2. 클라이언트 요청 들어오면 accept() → 3-way handshake
3. Socket에서 HTTP 요청 읽기
   GET /api/users HTTP/1.1
   Host: localhost:8080

4. HTTP 파싱 → DispatcherServlet이 Controller로 라우팅
5. Controller 메서드 실행
6. HTTP 응답 생성하여 Socket으로 전송
   HTTP/1.1 200 OK
   Content-Type: application/json

   {"users": [...]}

7. Connection: keep-alive면 Socket 유지
   Connection: close면 4-way handshake로 종료
```

Spring이 내부적으로 해주는 것
1. ServerSocket 생성 및 관리
2. 클라이언트 연결 accept()
3. HTTP 요청 파싱
4. Controller로 라우팅
5. HTTP 응답 생성 및 전송
6. 연결 관리 (Keep-Alive, Connection Pool)

---
## 핵심 정리

### Socket 계층별 이해

| 구분        | 저수준 Socket               | Spring Boot        |
| --------- | ------------------------ | ------------------ |
| Socket 생성 | `new ServerSocket(8080)` | `server.port=8080` |
| 연결 수락     | `accept()` 직접 호출         | Tomcat이 자동 처리      |
| 데이터 파싱    | `InputStream` 직접 읽기      | HTTP 요청 자동 파싱      |
| 비즈니스 로직   | 직접 구현                    | Controller 메서드     |
| 응답 전송     | `OutputStream` 직접 쓰기     | Return 값 자동 변환     |
| 복잡도       | 높음 (모든 걸 직접)             | 낮음 (프레임워크가 처리)     |

---

## 모의면접 질문

### Q1. Socket과 Port의 차이를 설명해주세요.

A1
- **Port**는 컴퓨터 내에서 프로그램을 식별하는 번호입니다 (0~65535).
- **Socket**은 네트워크 통신의 끝점으로, IP 주소와 Port 번호의 조합입니다.
- 건물 비유로 설명하면, IP는 "건물 주소", Port는 "호실 번호", Socket은 "건물 주소 + 호실"입니다.
- 예를 들어, `192.168.0.10:8080`이 Socket이고, 여기서 8080이 Port입니다.

### Q2. Spring Boot의 `server.port=8080`은 정확히 무엇을 의미하나요?

A2
- Spring Boot 애플리케이션이 8080 포트에서 HTTP 요청을 받아들인다는 의미입니다.
- 내부적으로 Tomcat이 `ServerSocket(8080)`을 생성하여 LISTEN 상태로 만듭니다.
- 클라이언트가 `http://서버IP:8080`으로 요청하면, Tomcat이 `accept()`를 호출하여 연결을 수락하고, HTTP 요청을 파싱하여 적절한 Controller로 라우팅합니다.
- 서버는 고정 포트(8080)를 사용하고, 클라이언트는 OS가 자동 할당한 임시 포트(49152~65535)를 사용합니다.

### Q3. 클라이언트가 서버에 연결할 때 임시 포트를 사용하는 이유는?

A3
- 클라이언트는 요청을 보내고 응답을 받으면 연결이 끝나기 때문에, 포트를 계속 점유할 필요가 없습니다.
- 한 클라이언트가 여러 번 요청할 때마다 다른 임시 포트를 사용할 수 있어, 동시에 여러 연결을 만들 수 있습니다.
- 서버는 항상 고정된 Well-Known Port(예: 8080)에 있어야 클라이언트가 찾을 수 있지만, 클라이언트는 서버 위치만 알면 되므로 임시 포트로 충분합니다.
- OS가 자동으로 사용 가능한 포트를 할당하므로 (Ephemeral Port, 49152~65535), 개발자가 신경 쓸 필요가 없습니다.

### Q4. TCP 연결은 언제 수립되나요? `new Socket()`과 `PrintWriter`의 차이는?

A4
- TCP 연결은 **`new Socket("localhost", 8080)` 호출 시점**에 3-way handshake를 통해 자동으로 수립됩니다.
- `accept()` 메서드가 반환되는 시점에는 이미 TCP 연결이 완료된 상태입니다.
- `PrintWriter`와 `BufferedReader`는 **이미 수립된 TCP 연결 위에서** 애플리케이션 데이터를 주고받는 도구입니다.
- 즉, TCP 연결 수립은 전송 계층(Transport Layer)에서 자동으로 처리되고, 데이터 교환은 애플리케이션 계층(Application Layer)에서 개발자가 직접 처리합니다.

### Q5. Spring Boot에서 데이터베이스 연결 시 Socket이 어떻게 사용되나요?

A5
```yaml
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
```
- `localhost:3306`이 MySQL 서버의 Socket 주소입니다.
- 애플리케이션이 DB 쿼리를 실행할 때마다 TCP Socket을 통해 MySQL과 통신합니다.
- HikariCP 같은 Connection Pool은 TCP Socket 연결을 미리 생성해두고 재사용합니다.
- 이렇게 하면 매번 3-way handshake를 할 필요가 없어 성능이 크게 향상됩니다.
- Connection Pool 없이 매번 새로운 연결을 만들면 handshake 오버헤드가 크기 때문에, 실무에서는 반드시 Connection Pool을 사용합니다.

### Q6. ServerSocket과 Socket의 차이는 무엇인가요?

A6
- **ServerSocket**은 서버가 클라이언트 연결을 기다리는(LISTEN) 용도로만 사용됩니다.
- `accept()` 메서드를 호출하면 클라이언트 연결이 들어올 때까지 Blocking됩니다.
- 연결이 들어오면 `accept()`가 **새로운 Socket 객체**를 반환합니다.
- **Socket**은 실제로 데이터를 송수신하는 통신 끝점입니다.
- 서버와 클라이언트 모두 Socket 객체로 데이터를 주고받습니다.
- 정리하면: ServerSocket = 연결 대기, Socket = 실제 통신

### Q7. HTTP Keep-Alive와 Socket의 관계는?

A7
```
HTTP/1.0 (기본 동작):
요청 → TCP 연결 → 응답 → 연결 종료 (매번 handshake)

HTTP/1.1 Keep-Alive (기본):
요청1 → TCP 연결 → 응답1 → (연결 유지)
요청2 → (기존 Socket 재사용) → 응답2 → (연결 유지)
요청3 → (기존 Socket 재사용) → 응답3 → 연결 종료
```
- Keep-Alive는 하나의 TCP Socket을 여러 HTTP 요청/응답에 재사용하는 기술입니다.
- 매번 3-way handshake를 하지 않아도 되므로 성능이 향상됩니다.
- Spring Boot의 Tomcat은 기본적으로 Keep-Alive를 지원합니다.
- 타임아웃 시간이 지나거나 클라이언트가 `Connection: close` 헤더를 보내면 Socket이 종료됩니다.

### Q8. 같은 서버에서 여러 Spring Boot 애플리케이션을 실행하려면?

A8
```yaml
# 첫 번째 앱
server.port=8080

# 두 번째 앱
server.port=8081
```
- 같은 IP 주소의 같은 포트에는 하나의 프로세스만 바인딩할 수 있습니다.
- 따라서 서로 다른 포트를 사용해야 합니다.
- 각 애플리케이션은 독립적인 Socket(IP:Port)으로 동작합니다.
- 만약 같은 포트를 사용하려고 하면 `java.net.BindException: Address already in use` 에러가 발생합니다.
- 실무에서는 Nginx 같은 리버스 프록시를 사용하여 80 포트로 들어온 요청을 내부 8080, 8081 포트로 분산할 수 있습니다.

---