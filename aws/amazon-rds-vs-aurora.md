## 1. Amazon RDS

### 1-1. 개념

- AWS가 제공하는 관리형 데이터베이스 서비스
- MySQL, PostgreSQL, MariaDB, Oracle, SQL Server 같은 다양한 데이터베이스 지원
- 사용자가 직접 데이터베이스를 설치하는 것이 아닌, AWS가 인프라 관리를 자동으로 한다.

### 1-2. 동작 방식

- 기본적으로 싱글 인스턴스 운영된다. (다중 AZ 배포 가능)
    - 기본적으로 하나의 DB 인스턴스로 동작하며, 장애 발생 시 수동으로 복구해야 한다.
    - 다중 AZ(Multi-AZ) 배포를 설정하면 읽기 가능한 인스턴스(Primary)와 한 개의 대기 인스턴스(Standby)가 존재하고, 장애 발생 시 자동으로 Failover 된다.
        - 기본적으로 Application은 Primary 인스턴스와만 통신한다. Application은 RDS와 연결할 때 Primary 인스턴스의 Endpoint를 사용하여 DB와 통신한다.
        - Standby 인스턴스는 Primary 인스턴스에서 처리하는 트랜잭션을 동기화만 수행하고 대기 상태로 존재한다.
        - Primary 인스턴스에 장애가 발생하면 RDS가 자동으로 Standby 인스턴스를 Primary 인스턴스로 승격한다. Application이 사용하는 En dpoint는 변경되지 않지만 내부적으로 새로운 Primary 인스턴스를 가리키도록 자동으로 변경된다.
- RDS의 데이터는 Amazon EBS(Elastic Block Store)에 저장된다.
    - 다중 AZ 배포 시에 Primary와 Standby 인스턴스는 각자 다른 AZ에 존재하지만, 각 AZ의 EBS에 동기식 복제가 수행된다. 따라서 Standby 인스턴스도 Primary 인스턴스와 동일하게 최신 데이터를 유지한다.
    - Primary 인스턴스가 트랜잭션을 commit하기 전에 반드시 Standby 인스턴스에도 동일한 데이터가 저장되어 있는지 확인한다.
    - 따라서 Primary 인스턴스에 장애가 발생하여 Standby 인스턴스가 승격되더라도 데이터가 유실되지 않고 그대로 유지될 수 있다.

### 1-3. 멀티 AZ 배포 시에 Primary, Standby 인스턴스는 서로 독립적인 EBS를 사용한다.

- Primary 인스턴스와 Standby 인스턴스는 서로 다른 AZ에 위치하며, 각 AZ에 개별적인 EBS를 사용한다.
    - ex. Primary 인스턴스는 AZ-1, Standby 인스턴스는 AZ-2에 위치
- AWS는 동기식 복제(Synchronous Replication)를 수행하여 Primary EBS의 데이터가 Standby 인스턴스에 실시간으로 동일하게 저장된다.
- Primary와 Standby 인스턴스가 서로 다른 EBS를 사용하지만 데이터는 항상 동일한 상태로 유지된다.
- Application과 Primary 인스턴스 통신 과정에서 새로운 데이터가 Primary EBS에 저장되었을 때 Standby EBS에 동기화되는 과정
    1. Application이 Primary 인스턴스에 데이터 쓰기 요청
    2. Primary 인스턴스가 데이터를 Primary EBS에 저장
        1. Primary 인스턴스는 새로운 데이터를 Primary EBS에 먼저 저장한다.
        2. 멀티 AZ 환경에서는 Standby 인스턴스와 데이터 일관성을 보장하기 위해 트랜잭션을 commit하기 전에 Standby 인스턴스로 데이터를 동기화하는 과정이 필요하다.
    3. Primary 인스턴스가 Standby 인스턴스의 EBS로 데이터 동기화를 요청한다.
        1. Primary 인스턴스는 AWS의 동기식 복제(Synchronous Replication)을 수행한다.
        2. Primary 인스턴스에 저장되는 데이터가 Standby 인스턴스로 데이터가 전송되므로, Primary EBS에 변경된 사항이 Standby EBS에도 복제된다.
    4. Standby 인스턴스에 동기화 후, Primary 인스턴스에서 트랜잭션을 commit한다.
        1. Standby 인스턴스의 EBS에 데이터가 정상적으로 동기화되었는지 확인되면, Primary 인스턴스에서 트랜잭션을 commit한다.
        2. 이 시점에 Application이 트랜잭션이 성공적으로 완료되었다는 응답을 받게 된다.
    
    ### 1-4. Failover 발생 시 Standby 인스턴스가 Primary 인스턴스로 승격된다.
    
    1. Primary 인스턴스 장애 발생
        1. Application과 통신할 때 사용된 Primary 인스턴스에 장애가 발생한다.
        2. Application은 Primary 인스턴스와 통신할 수 없는 상태가 된다.
    2. RDS가 자동으로 Standby 인스턴스를 새로운 Primary 인스턴스로 승격시킨다.
        1. 기존 Standby 인스턴스가 새로운 Primary 인스턴스로 승격된다.
        2. 승격된 인스턴스는 기존의 RDS Endpoint를 그대로 사용한다.
        3. 따라서 Application은 RDS와 통신하기 위한 RDS Endpoint를 변경할 필요가 없다.
        4. Standby 인스턴스가 새로운 Primary 인스턴스로 승격될 때 Standby 인스턴스의 EBS를 그대로 사용한다.
    3. 기존의 Standby 인스턴스는 새로운 Primary 인스턴스로 승격되고, 새로운 Standby 인스턴스가 생성되어 새로운 Primary 인스턴스와 연결되어 동기식 복제(Synchronous Replication)를 수행한다.
        1. 멀티 AZ 구성이 계속 유지된다.

## 2. Aurora RDS

<img width="596" alt="스크린샷_2025-02-15_오후_1 07 59" src="https://github.com/user-attachments/assets/3ed4fc90-e1b9-4447-a696-c077c0a4f5b3" />

### 2-1. 개념

- AWS가 자체 개발한 고성능 관리형 관계형 데이터베이스 서비스
- MySQL 및 PostgreSQL과 호환된다.
- Amazon RDS(일반 RDS)보다 최대 5배(MySQL 기준), 3배(PostgreSQL 기준) 빠른 성능을 제공한다.
- 성능이 좋을 뿐만 아니라 고가용성을 갖춘 데이터베이스이다.

### 2-2. 동작 방식

1. 분산 Storage 아키텍처
    1. Aurora는 6개의 복제본(3개의 AZ에 2개씩 배포)이 자동으로 생성된다.
    2. 데이터가 6개의 노드에 분산되어 저장되기 때문에, 한 개의 노드에 장애가 발생해도 서비스가 지속된다. ⇒ 고가용성
    3. 기존 RDS와 달리 Storage 용량이 자동으로 확장된다.
2. 빠른 Failover
    1. Aurora는 기본적으로 다중 노드로 구성되므로, 하나의 노드에 장애가 발생해도 나머지 노드가 자동으로 트래픽을 처리한다.
    2. Amazon RDS보다 Failover 속도가 빠르다.
3. Read Replica를 사용하여 고성능 읽기를 제공
    1. Aurora는 최대 15개의 Read Replica를 지원하고 자동으로 데이터 동기화가 이루어진다.
    2. Read Replica를 사용하므로 읽기 성능이 매우 뛰어나다. 이는 읽기 요청 트래픽이 많이 발생하는 시스템에 유리하다.
4. Serverless 지원
    1. Aurora Serverless 옵션을 사용하면 트래픽에 따라 자동으로 확장 및 축소되며, 사용한 만큼만 비용이 부과된다.

### 2-3. Aurora RDS의 데이터 분산 저장 방식

Aurora RDS는 Amazon RDS와 달리 Shared Storage(스토리지 계층)에서 데이터를 저장하고 관리한다. 6개의 스토리지 노드가 3개의 AZ에 분산되어 배치된다. Primary 인스턴스와 Read Replica들은 동일한 Shared Storage를 사용한다.

```sql
      [ Aurora RDS - 컴퓨팅 계층 (Compute Layer) ]
        (Handles SQL Queries & Application Traffic)
                     +----------------------+
                     |    Primary DB        |  (Handles Write & Read)
                     +----------------------+
                               |
        ------------------------------------------------
        |                      |                      |
        v                      v                      v
+----------------+    +----------------+    +----------------+
| Read Replica 1 |    | Read Replica 2 |    | Read Replica 3 |
| (Read Only)    |    | (Read Only)    |    | (Read Only)    |
+----------------+    +----------------+    +----------------+
        |                      |                      |
        ------------------------------------------------
                               |
                               v
     =====================================================
      [ Aurora Shared Storage - 스토리지 계층 (Storage Layer) ]
       (6 Storage Nodes, Distributed across 3 AZs)
     =====================================================
        |                      |                      |
        v                      v                      v
  +------------+         +------------+         +------------+
  | Storage AZ-1 |       | Storage AZ-2 |       | Storage AZ-3 |
  |  Node 1, 2   |       |  Node 3, 4   |       |  Node 5, 6   |
  +------------+         +------------+         +------------+
```

### 2-4. Aurora RDS에서 쓰기 작업 처리 과정

Aurora RDS에서 쓰기 작업은 Primary 인스턴스에서만 수행되며, 데이터는 Shared Storage에 저장된다.

```sql
[쓰기 요청 - Primary 인스턴스 사용]
Application → Aurora Primary Instance → Shared Storage (6개 노드로 분산 저장)
                     |
                     ├── Step 1: 데이터를 6개 노드에 분산
                     ├── Step 2: 최소 4개 노드 저장 성공 시 Commit
                     ├── Step 3: 나머지 2개 노드에 비동기적으로 복제 완료
                     v
(Write Successful: Application Receives Acknowledgment)
```

1. Application이 Primary 인스턴스로 쓰기 요청을 보낸다.
    1. Application은 Aurora 클러스터의 Cluster Endpoint (Primary Endpoint)를 통해 Primary 인스턴스에 연결하여 쓰기 요청을 보낸다.
        1. Primary Endpoint: <cluster-name>.cluster-xxxxx.rds.amazonaws.com
    2. Aurora에서는 오직 Primary 인스턴스에서만 쓰기 연산이 가능하고, Read Replica에서는 쓰기 연산을 처리할 수 없다.
2. Primary 인스턴스가 데이터 변경을 Shared Storage에 요청
    1. Primary 인스턴스는 Shared Storage에 데이터를 직접 저장하는 것이 아닌, 변경된 데이터 블록을 Shared Storage로 보내는 역할을 한다.
    2. Aurora는 변경된 데이터를 10GB 단위 블록(Shard)로 관리하며, 이를 6개의 노드에 걸쳐 분산 저장한다. 6개의 노드는 3개의 AZ에 2개씩 분산되어 관리된다.
    3. 이때, Aurora는 트랜잭션을 commit하기 전에 데이터가 Shared Storage에 존재하는 6개의 노드 중 최소 4개 이상에 저장되었는지 확인한다.
3. Shared Storage가 데이터를 6개 노드에 복제 (Quorum Write)
    1. Aurora의 Shared Storage는 6개의 노드에 데이터를 복제한다.
    2. Aurora는 쓰기 연산을 Quorum Write 방식으로 처리하는데,
        1. 6개 노드 중 4개 이상의 노드에서 데이터 저장이 완료되면 트랜잭션을 commit한다.
        2. 이를 통해 고가용성을 보장할 수 있다.
4. 데이터 저장이 완료되면 Primary 인스턴스에서 트랜잭션을 commit한다.
    1. 6개 노드 중 4개 이상에서 데이터 저장이 완료되면, Primary 인스턴스는 트랜잭션을 commit한다.
    2. 이후 나머지 2개 노드에도 데이터를 복제하는 작업이 비동기적으로 진행된다.
    3. Primary 인스턴스는 쓰기 연산이 완료되었으므로 Application에 성공 응답을 반환한다.
5. 나머지 2개 노드에서도 비동기적으로 데이터 복제가 완료된다.
    1. Aurora는 남은 2개의 노드에도 데이터를 복제하여, 6개 노드 모두 동일한 데이터를 가지도록 동기화한다.
    2. 이 과정은 비동기적으로 실행되므로 트랜잭션 성능에 영향을 미치지 않는다.
    3. 덕분에 Read Replica는 Shared Storage에서 최신 데이터를 읽게 된다.

- INSERT : 기존의 Shared Storage에 존재하는 10GB 단위의 블록 중에서 공간이 남아있는 블록에 데이터를 추가한다. 모든 블록이 꽉 찼다면 새로운 10GB 블록을 생성하여 데이터를 저장한다.
- UPDATE : 기존 데이터를 덮어쓰지 않고, 새로운 데이터 블록에 변경 사항을 저장한다. 즉, 이전 데이터는 그대로 남아있고 새로운 블록에 새로운 데이터를 저장한다.
- DELETE : 데이터를 즉시 삭제하지 않고 Delete Marker를 기록한다. 즉, 기존 데이터를 그대로 두고, “이 데이터는 삭제됨”이라는 정보를 기록하므로 Aurora는 삭제된 데이터로 인식하여 조회 시 무시한다. 이후 Garbage Collection이 실행될 때 실제로 삭제된다.

Aurora RDS는 Quorum Write 방식으로 쓰기 작업을 수행한다. INSERT, UPDATE, DELETE 작업 모두 Shared Storage에 존재하는 6개의 노드 중에서 4개 이상에 먼저 저장한다. 4개 이상의 노드에 데이터 저장이 완료되면 Primary 인스턴스는 트랜잭션을 commit한다. 이후에 나머지 2개의 노드에 비동기적으로 데이터를 복제하여 동기화를 완료한다.

```sql
[INSERT 요청]
Application → Primary Instance → Shared Storage (10GB 블록 중 빈 공간이 있는 블록에 저장)
                 |
                 ├── Step 1: 기존 10GB 블록에 저장할 공간이 있으면 같은 블록 사용
                 ├── Step 2: 공간 부족 시 새로운 10GB 블록 할당 후 저장
                 ├── Step 3: 데이터가 6개 노드 중 4개 이상에 복제되면 Commit
                 v
(INSERT 성공)

[UPDATE 요청]
Application → Primary Instance → Shared Storage (새로운 블록에 변경된 데이터 저장)
                 |
                 ├── Step 1: 기존 데이터는 그대로 두고, 새로운 데이터 버전을 다른 블록에 저장 (MVCC)
                 ├── Step 2: 데이터가 6개 노드 중 4개 이상에 복제되면 Commit
                 v
(UPDATE 성공, 이전 데이터는 나중에 정리됨)

[DELETE 요청]
Application → Primary Instance → Shared Storage (삭제 마커 기록)
                 |
                 ├── Step 1: 데이터를 즉시 삭제하지 않고, "삭제됨" 마커를 남김
                 ├── Step 2: Garbage Collection이 실행될 때 불필요한 데이터를 제거
                 v
(DELETE 성공, 실제 데이터 삭제는 Garbage Collection 이후)
```

### 2-5. Aurora RDS에서 읽기 작업 처리 과정

> 참고 : [https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Reader.html](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Endpoints.Reader.html)
> 

Aurora RDS는 읽기 성능을 최적화하기 위해 여러 개의 Read Replica를 지원하며, 읽기 작업을 효과적으로 분산 처리할 수 있도록 Reader Endpoint를 제공한다.

Reader Endpoint는 Aurora 클러스터의 모든 Read Replica를 가리키는 공통 Endpoint이다.

Application이 직접 특정 Read Replica를 선택할 필요 없이, Reader Endpoint를 사용하면 최적의 Read Replica에서 데이터를 조회할 수 있다.

```sql
[읽기 요청 - Reader Endpoint 사용]
Application → Aurora Reader Endpoint → Load Balancing → Read Replica 1 (Selected)
                              |
                              ├─ Read Replica 2 (if needed)
                              ├─ Read Replica 3 (if needed)
                              ├─ Read Replica 4 (if needed)
                              |
                              v
                    (Returns Data to Application)
```

1. Application은 Aurora 클러스터의 Reader Endpoint를 사용하여 Aurora에 읽기 요청을 보낸다.
    1. Reader Endpoint: <cluster-name>.cluster-ro-xxxxx.rds.amazonaws.com
2. Aurora가 가장 적절할 Read Replica를 선택한다.
    1. Aurora는 클러스터 내에 존재하는 여러 Read Replica 중에서 부하 상태, 응답 속도, 네트워크 지연 시간 등을 고려하여 최적의 Read Replica를 자동으로 선택한다.
    2. Aurora는 읽기 트래픽이 많을 경우 부하를 분산시키기 위해 여러 Read Replica로 요청을 분산할 수 있다.
3. 선택된 Read Replica가 데이터를 조회한다.
    1. Aurora는 선택된 Read Replica에서 데이터를 조회한다.
    2. Aurora의 Shared Storage는 Primary 인스턴스 및 모든 Read Replica와 동일한 데이터를 공유하므로 데이터 동기화 문제 없이 최신 데이터를 제공한다.
4. Read Replica가 조회한 데이터를 Application에 반환한다.
