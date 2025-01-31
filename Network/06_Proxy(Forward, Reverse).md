## 1. 프록시(Proxy)
- 클라이언트와 서버 사이의 중계 역할을 수행한다.
- 프록시가 사용되는 위치에 따라 Forward Proxy와 Reverse Proxy로 나뉜다.

## 2. Forward Proxy
Forward Proxy는 클라이언트 측에서 동작하는 프록시 서버이다. 클라이언트는 웹 사이트에 직접 접근하는 것이 아닌 Forward Proxy를 거쳐 접속하게 된다.

- 프록시 사용 전
<img width="748" alt="스크린샷 2025-01-31 오후 10 03 35" src="https://github.com/user-attachments/assets/79de1a15-2ded-4bb6-ac47-754519928026" />

사용자가 주소창에 www.google.com 을 입력하여 직접 웹 사이트에 접속한다.

- 프록시 사용 후
<img width="757" alt="image" src="https://github.com/user-attachments/assets/a0df6c5d-79d9-4087-8c71-db1251022c45" />

사용자가 주소창에 www.google.com 을 입력하면 Forward Proxy가 요청을 받아서 처리한다. 즉, 사용자는 해당 사이트에 직접 접근하는 것이 아닌 프록시 서버를 거쳐 접속되는 것이다.

### 2-1. Forward Proxy 사례
군대에서는 인트라넷(내부망)을 사용하여 군 업무와 관련없는 사이트의 접속을 차단한다. 예를 들어, 인트라넷을 사용하는 컴퓨터에서 www.google.com 을 입력하면 접속이 차단당한다. 이는 Forward Proxy에서 클라이언트의 요청을 차단했기 때문인데 과정을 자세히 알아보자.
1. 군 내부망에 연결된 컴퓨터에서 www.google.com 접속을 시도한다.
2. 이 요청은 Forward Proxy 서버로 전달된다.
3. Forward Proxy 서버에서 차단 정책을 확인한 결과, 프록시 서버는 해당 요청을 외부 인터넷으로 전달을 금지한다.
4. 클라이언트는 www.google.com 사이트 접속에 차단된다.

## 3. Reverse Proxy
Reverse Proxy는 서버 측에서 동작하는 프록시 서버이다. 클라이언트가 직접 백엔드 서버로 요청을 보내지 않고 Reverse Proxy를 통해 요청을 전달하는 방식이다.

- 프록시 사용 예시
<img width="729" alt="스크린샷 2025-01-31 오후 10 17 47" src="https://github.com/user-attachments/assets/96d650de-1a72-4936-87d9-b735b464e8d0" />

클라이언트는 웹 서비스에 접근할 때 백엔드 서버로 직접 요청을 보내는 것이 아닌 Reverse Proxy 서버로 요청한다. 이 프록시 서버가 백엔드 서버로 요청을 전달하고 응답을 받아 클라이언트에 전달한다.

### 3-1. Reverse Proxy 사례
Nginx를 사용하여 클라이언트가 서버로 보내는 요청을 먼저 받아 대신 서버에 전달한다. 클라이언트가 HTTPS 요청을 보내면 Nginx에서 443 포트로 오는 요청을 대기하였다가 서버로 전달한다.
1. 클라이언트가 https://google.com 으로 요청
2. Nginx(Reverse Proxy)가 요청을 받아 백엔드 서버(Http://localhost:8080)로 전달
3. 백엔드 서버가 요청을 처리하고 응답을 생성
4. Nginx가 백엔드 서버의 응답을 받아 클라이언트에게 반환

즉, Nginx가 443 포트(HTTPS)에서 요청을 받아 8080 포트(Spring Boot 서버)로 전달하는 역할을 수행하는 것이 Reverse Proxy이다.
