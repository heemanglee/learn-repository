## 1. CDN(Content Delivery Network)

![image](https://github.com/user-attachments/assets/325d1b36-9d89-4a99-92f8-0cdf3778bb01)

CDN(Content Delivery Network, 콘텐츠 전송 네트워크)는 웹 콘텐츠(CSS, HTML, JavaScript, 이미지, 동영상 등)를 사용자에게 빠르게 전송하기 위해서 사용되는 분산 네트워크 시스템이다. 클라이언트와 원본 서버 간의 물리적 거리가 멀 경우 서버에서 전송한 데이터가 클라이언트에 도착하는데 지연 시간이 증가하므로 사용자 경험이 좋지 않다. 예를 들어, 한국에서 인터넷을 사용하는 사용자가 CDN을 사용하지 않고 해외에서 호스팅하고 있는 웹 사이트에 접근하면 화면이 로딩되는 데 오랜 시간이 걸리곤 한다.

물리적 거리가 멀 수록 지연 시간이 증가하는 문제를 해결하기 위해 CDN을 사용할 수 있다. 전 세계 여러 지역에 CDN 엣지 서버를 분산 설치하여 사용자가 오리진 서버로부터 데이터를 전송받지 않고 엣지 서버에 캐시되어 있는 데이터를 빠르게 전송받을 수 있다. 
> - 엣지 서버(Edge Server) : 사용자 가까이에 위치한 CDN 서버
> - 오리진 서버(Origin Server) : 원본 데이터를 저장하는 서버

클라이언트는 여러 엣지 서버 중에서 자신과 가장 가까운 엣지 서버로부터 캐시되어 있는 데이터를 전송받는다. 먼 거리에 있는 오리진 서버로부터 데이터를 전송받지 않기 때문에 지연 시간을 줄일 수 있으며, 오리진 서버에 트래픽이 과부화되는 것을 방지할 수 있다. 
- CDN 사용 전 : 모든 요청이 오리진 서버로 전송되기 때문에 트래픽 과부하가 발생하므로 대역폭 사용량이 증가한다.
  - 대역폭 사용량이 증가하면 클라이언트가 응답받는 속도가 느려짐 => 지연 시간 증가
- CDN 사용 후 : 엣지 서버에서 캐싱된 데이터를 제공하여 오리진 서버의 트래픽이 감소하므로 대역푝 사용량이 감소한다.

## 2. CDN 캐싱 과정
1. 클라이언트가 웹 사이트에 접속하여 정적 웹 콘텐츠를 요청한다.
2. DCN 엣지 서버가 먼저 요청을 확인한다.
3. 엣지 서버에 캐시된 데이터가 있다면 즉시 응답한다.
4. 엣지 서버에 캐시된 데이터가 없으면 오리진 서버에서 콘텐츠를 받아와 클라이언트에 응답하고, 엣지 서버에 캐싱한다.
5. 이후 동일한 요청이 발생하면 엣지 서버에서 캐싱된 데이터를 제공하여 빠르게 응답한다.

## 3. CDN에 정적 콘텐츠(Static Content)와 동적 콘텐츠(Dynamic Content)를 저장한다.
- 정적 콘텐츠(Static Content)
  - 데이터가 자주 변하지 않는 정적 파일을 캐싱한다.
    - ex. CSS, JavaScript, 이미지(JPG, PNG, SVG), 웹 폰트, 동영상
  - 데이터가 자주 변경되지 않으므로 TTL(Time To Live)를 길게 설정해도 된다.
- 동적 콘텐츠(Dynamic Content)
  - 데이터가 자주 변하는 동적 콘텐츠를 캐싱한다.
    - ex. 현재 날씨, 상품 가격
  - 데이터가 자주 변경되므로 TTL(Time To Live)를 짧게 설정해야 한다.

사용자마다 다르게 전송되는 동적 콘텐츠는 엣지 서버에 캐싱하면 안 되겠지만, 이를 제외한 공통적으로 사용되는 동적 콘텐츠는 짧은 시간동안 캐싱하여 트래픽 과부하를 줄일 수 있다.
