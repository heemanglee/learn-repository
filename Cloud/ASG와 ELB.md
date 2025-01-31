## 1. 오토 스케일링(Auto Scaling)
오토 스케일링은 트래픽이 증가하였을 때 자동으로 서버를 증설하고, 반대로 트래픽이 감소한다면 불필요한 서버를 제거하여 비용을 절감하는 기술이다. 

## 2. ELB(Elastic Load Balancer)
ELB(Elastic Load Balancer)는 EC2의 로드 밸런서이다. 로드 밸런서는 트래픽이 발생하였을 때 이를 적절하게 생성되어 있는 서버들에게 분산시키는 방법이다. ELB는 ASG(Auto Scaling Group) 내의 EC2 인스턴스들에게 트래픽을 분산시켜 효과적으로 처리한다.

## 3. ASG(Auto Scaling Group)
ASG(Auto Scaling Group)는 오토 스케일링 정책에 따라 생성된 EC2 인스턴스의 모음이다. 트래픽이 증가하면 EC2 인스턴스를 추가로 증성하고, 반대로 트래픽이 감소한다면 불필요한 EC2 인스턴스를 제거한다.
- Scale-Out : ASG는 트래픽이 증가하면 자동으로 EC2 인스턴스를 생성
- Scale-In : ASG는 트래픽이 감소하면 불필요한 EC2 인스턴스 제거

ASG는 Cloud Watch와 연동하여 EC2 인스턴스의 CPU 사용률을 모니터링하여 추가로 EC2 인스턴스를 생성하거나 제거한다. 또한, 헬스 체크(Health Check)를 통해 주기적으로 EC2 인스턴스의 상태를 확인한다. 이때, 비정상적인 인스턴스가 있다면 새로운 인스턴스를 생성하여 교체하는 작업을 수행한다.
