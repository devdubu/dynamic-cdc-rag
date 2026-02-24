# 🚀 Dynamic CDC-RAG Infrastructure

본 프로젝트는 레거시 데이터베이스(PostgreSQL)의 변경 사항을 실시간으로 감지(CDC)하여 지능형 RAG(Retrieval-Augmented Generation) 시스템에 즉각적으로 반영하기 위한 **실시간 데이터 파이프라인**입니다.

## 🌟 Key Features

- **Real-time Synchronization**: DB의 `INSERT`, `UPDATE`, `DELETE` 발생 시 1초 미만의 지연 시간으로 Kafka 토픽에 반영됩니다.
- **Zero-ETL Approach**: 복잡한 배치 작업 없이 DB 로그(WAL)를 직접 읽어 성능 부하를 최소화합니다.
- **Unified Management**: **Kafka UI**를 통해 토픽 모니터링부터 Debezium 커넥터 관리까지 하나의 대시보드에서 제어합니다.
- **ARM64 Optimized**: Apple Silicon(M1/M2/M3) 환경에서 완벽하게 동작하도록 컨테이너 스택을 최적화했습니다.

## 🛠 Tech Stack

- **Database**: PostgreSQL 15 (Logical Decoding enabled)
- **Streaming**: Apache Kafka & Zookeeper (Confluent Platform)
- **CDC Engine**: Debezium Connect 2.3
- **Management**: Kafka UI (Provectus Labs)

---

## 🚀 Getting Started

### 1. 인프라 가동

모든 서비스는 Docker Compose를 통해 원클릭으로 실행됩니다.

```bash
docker-compose up -d

```

### 2. 서비스 접속 정보

| 서비스            | URL                     | 비고                                    |
| ----------------- | ----------------------- | --------------------------------------- |
| **Kafka UI**      | `http://localhost:8080` | **메인 대시보드** (커넥터 및 토픽 관리) |
| **PostgreSQL**    | `localhost:5432`        | User: `myuser` / DB: `mydb`             |
| **Kafka Connect** | `localhost:8083`        | Debezium REST API 엔드포인트            |

---

## ⚙️ Connector Configuration (Kafka UI)

1. `http://localhost:8080` 접속 후 왼쪽 메뉴의 **[Kafka Connect]** 클릭.
2. **[Create Connector]** 버튼 클릭 후 아래 설정을 입력합니다.

- **Connector Name**: `product-connector`
- **Config (JSON)**:

```json
{
  "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
  "database.hostname": "postgres",
  "database.port": "5432",
  "database.user": "myuser",
  "database.password": "mypassword",
  "database.dbname": "mydb",
  "topic.prefix": "dbserver1",
  "table.include.list": "public.products",
  "plugin.name": "pgoutput"
}
```

---

## 🧪 Verification & Testing

### Step 1: 데이터 이벤트 발생 (SQL)

```bash
docker exec -it postgres psql -U myuser -d mydb

```

```sql
-- 실시간 데이터 입력 테스트
INSERT INTO products (title, content, price)
VALUES ('MacBook Pro M3', 'Real-time CDC Test', 3500000);

```

### Step 2: 실시간 로그 확인

1. **Kafka UI** -> **Topics** -> `dbserver1.public.products` 접속.
2. **Messages** 탭에서 실시간으로 생성된 JSON 페이로드를 확인합니다.
3. `op: c` (Create) 신호와 함께 데이터가 포함되어 있다면 성공입니다.

---

## 📂 Next Roadmap

- [ ] **AI Adapter (Python)**: Kafka 메시지를 구독하여 텍스트 임베딩 생성.
- [ ] **Vector Sync**: 생성된 벡터를 Qdrant/Milvus에 실시간 `Upsert`.
- [ ] **Context Scoper**: 사용자 세션 및 페이지 권한(`pcd`) 기반의 휘발성 컨텍스트 필터링 구현.

---

## 🧹 Cleanup

```bash
docker-compose down -v

```
