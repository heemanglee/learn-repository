## 1. AWS EBS(Elastic Block Store)

### 1-1. 개념

- AWS EBS는 AWS에서 제공하는 Block Storage 서비스이다.
- AWS EBS는 AWS EC2에서 사용되는 저장 장치이다.
- AWS EC2는 OS와 Application을 실행하기 위한 서버 역할을 하므로 데이터 저장 공간이 필요하다. AWS EBS는 EC2 인스턴스에 연결(Attach)되어 EC2가 데이터를 읽고 쓰는 저장소 역할을 한다.

### 1-2. 사용 전후의 차이

|  | **EBS 사용 전 (EC2만 있음)** | **EBS 사용 후 (EC2 + EBS)** |
| --- | --- | --- |
| **스토리지 제공 방식** | EC2 인스턴스 내 임시 저장소(Instance Store)만 사용 가능 | EBS 볼륨을 추가하여 영구적인 저장 가능 |
| **데이터 보존 여부** | 인스턴스 종료 시 데이터 손실 | 인스턴스를 종료해도 EBS 데이터는 유지됨 |
| **백업 가능 여부** | 백업 어려움 (별도 데이터 이동 필요) | 스냅샷(Snapshot) 기능으로 백업 가능 |
| **크기 조정** | 변경 불가 (인스턴스 재설치 필요) | 필요에 따라 크기 조정 가능 |
| **다른 인스턴스에서 사용** | 불가능 | 가능 (다른 EC2에 재연결 가능) |

### 1-3. 주요 특징

- AWS EC2와 독립적으로 동작한다.
    - EC2를 종료하거나 다시 시작해도 EBS 볼륨 데이터는 유지된다.
    - 필요하면 다른 EC2 인스턴스에 EBS를 연결할 수 있다.
- 탄력적인 크기 조정
    - EBS를 생성할 때 초기 용량을 설정하지만, 이후에 필요하다면 크기를 확장할 수 있다.
- 스냅샷(Snapshot) 기능 제공
    - EBS 볼륨의 스냅샷을 AWS S3에 저장할 수 있다.
    - 따라서 데이터 백업 및 복원이 가능해진다.

### 1-4. AWS EBS가 필요한 이유

- EC2의 임시 저장소(Instance Store)는 인스턴스 종료 시에 휘발되므로 데이터가 사라진다.
- EBS는 스냅샷을 제공하므로 데이터를 백업할 수 있다.
- EBS는 초기 용량을 설정하고 이후에 크기를 확장할 수 있으므로 탄력적이다.

⇒ AWS EC2만 사용하면 임시 저장소를 사용하므로 데이터를 안전하게 보존할 수 없으나, EBS를 같이 사용하면 데이터를 영구적으로 저장하고 확장할 수 있으므로 애플리케이션을 안정적으로 운영할 수 있다.

### 1-5. EBS는 스냅샷을 사용한다.

> 스냅샷(Snapshot) : 특정 시점의 데이터를 그대로 복사하여 저장하는 것을 의미한다. 스냅샷을 기반으로 특정 시점의 EBS 볼륨을 그대로 저장하고, 이후에 해당 시점으로 볼륨을 복원할 수 있다.
> 
- EBS 스냅샷은 AWS S3에 저장된다. 하지만 일반적인 S3 버킷과 달리, AWS가 내부적으로 관리하는 비공개 S3 Storage에 저장되므로 사용자가 직접 접근할 수 없다.
- 사용자는 스냅샷을 기반을호 새로운 EBS 볼륨을 만들거나 기존 볼륨을 복원할 수 있다.

```sql
1. EC2에서 사용 중인 EBS 볼륨이 있음
    │
2. 특정 시점에서 EBS 스냅샷을 생성
    │
3. AWS 내부 S3 저장소에 저장됨
    │
4. 필요할 때 해당 스냅샷을 기반으로 새로운 EBS 볼륨 생성
    │
5. 새롭게 생성된 볼륨을 EC2에 연결하여 사용 가능
```

## 2. AWS Instance Store(EC2의 임시 저장소)

### 2-1. 개념

- Instance Store는 물리적으로 EC2 인스턴스에 연결된 디스크이므로 빠른 입출력이 가능하다.
- EC2 인스턴스가 재부팅되거나 종료될 때 데이터가 삭제되기 때문에 임시 데이터를 저장하는 용도로 사용한다.
- EBS 대비 추가 비용 없이 제공되는 저장소이다.

### 2-2. 주요 특징

| **특징** | **설명** |
| --- | --- |
| **고속 I/O 성능** | 물리적으로 EC2에 연결되어 있어 EBS보다 빠름 |
| **데이터 휘발성** | EC2 인스턴스가 종료되면 데이터가 사라짐 |
| **백업 불가** | Snapshot 같은 백업 기능이 없으며, 데이터 유실 방지를 위해 외부 저장 필요 |
| **추가 비용 없음** | EC2 인스턴스 유형에 따라 기본 제공되므로 별도 비용이 들지 않음 |
| **크기 변경 불가** | 기본 제공된 디스크 크기를 변경할 수 없음 |

### 2-3. Instance Store vs EBS 사용 시점

| **사용 목적** | **Instance Store** | **EBS** |
| --- | --- | --- |
| **고속 성능(I/O) 필요** | ✅ | ❌ |
| **데이터 영구 저장 필요** | ❌ | ✅ |
| **EC2 종료 후 데이터 유지 필요** | ❌ | ✅ |
| **백업 및 스냅샷 필요** | ❌ | ✅ |
| **임시 데이터 저장 (캐시, 로그, 중간 데이터)** | ✅ | ❌ |
| **스토리지 크기 조정 가능해야 함** | ❌ | ✅ |

### 2-4. Instance Store와 EBS 차이점

| **구분** | **EBS (Elastic Block Store)** | **Instance Store** |
| --- | --- | --- |
| **접근 경로** | 네트워크를 통해 연결 | 물리적으로 EC2에 직접 연결 |
| **속도** | 네트워크 지연 발생 가능 | 로컬 디스크라 매우 빠름 |
| **데이터 지속성** | EC2 종료 후에도 데이터 유지 | EC2 종료 시 데이터 삭제 |
| **백업 가능 여부** | 스냅샷(Snapshot) 기능 제공 | 백업 기능 없음 |
| **사용 사례** | 영구 데이터 저장 (DB, 앱 데이터) | 캐시, 임시 데이터 저장 |
- EBS를 사용할 때 EC2가 데이터를 읽거나 쓰는 과정
    
    ```sql
    EC2 인스턴스
        │
        ▼
    운영체제 (OS)
        │
        ▼
    EBS 드라이브 (네트워크를 통해 연결)
        │
        ▼
    EBS 볼륨 (AWS 내부 스토리지)
    ```
    
    - EC2는 네트워크 연결을 통해 EBS에서 데이터를 읽고 쓴다.
    - 네트워크 속도에 따라 데이터 접근 속도가 결정된다.
- Instance Store를 사용할 때 EC2가 데이터를 읽거나 쓰는 과정
    
    ```sql
    EC2 인스턴스
        │
        ▼
    운영체제 (OS)
        │
        ▼
    Instance Store (물리적 디스크)
    ```
    
    - Instance Store는 EC2에 직접 연결된 로컬 디스크라서 네트워크 속도에 영향을 받지 않아 접근 속도가 매우 빠르다.

---

참고

- [Amazon Elastic Block Store란 무엇인가요?](https://docs.aws.amazon.com/ko_kr/ebs/latest/userguide/what-is-ebs.html)
- [EC2 인스턴스용 인스턴스 스토어 임시 블록 스토리지](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/InstanceStorage.html)
