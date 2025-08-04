# README - DB Repository

담당자 : 류성열   
초기 세팅 날짜 : 2025-08-04

## 개요

이 저장소는 프로젝트에서 사용하는 데이터베이스 인프라를 도커 컨테이너로 구성하기 위한 설정을 포함합니다.
PostgreSQL, MongoDB, 그리고 Valkey의 세션 및 캐시용 인스턴스를 분리하여 띄우도록 설계되어 있습니다.

---

## 구성 서비스

| 서비스명           | 이미지                   | 설명                     | 포트 매핑            |
| -------------- | --------------------- | ---------------------- | ---------------- |
| postgres       | postgres:17           | 관계형 데이터베이스(PostgreSQL) | 5432 (호스트:컨테이너)  |
| mongo          | mongo:7.0             | NoSQL 데이터베이스(MongoDB)  | 27017 (호스트:컨테이너) |
| valkey-session | valkey/valkey\:latest | Valkey 세션 저장소 인스턴스     | 6379 (호스트:컨테이너)  |
| valkey-cache   | valkey/valkey\:latest | Valkey 캐시 저장소 인스턴스     | 6380 (호스트 6379)  |

---

## 사용 방법

1. 저장소를 클론합니다.

   ```bash
   git clone https://github.com/DungeonTalk/dungeontalk-db.git
   cd db-repo
   ```

2. 도커 컴포즈로 모든 DB 서비스를 백그라운드에서 실행합니다.

   ```bash
   docker compose -f docker-compose-db.yml up -d
   ```

3. 각 서비스가 정상적으로 실행되었는지 확인합니다.

   ```bash
   docker ps
   ```

---

## 각 서비스 상세 설정

### PostgreSQL

* 버전: 17
* 기본 유저: `root`
* 비밀번호: `1234`
* 초기화 스크립트: `./postgres/init.sql` 경로의 SQL 파일이 컨테이너 시작 시 자동 실행됩니다.

---

### MongoDB

* 버전: 7.0
* 기본 설정으로 실행되며, 추가 초기화 스크립트는 포함되어 있지 않습니다.

---

### Valkey 세션 저장소 (`valkey-session`)

* Valkey 최신 이미지 사용
* 포트: 6379
* 설정 파일: `./valkey/session.conf`

#### `session.conf` 주요 내용

```conf
port 6379
timeout 0
save 900 1
```

* `port 6379` : 기본 포트
* `timeout 0` : 클라이언트 연결 제한 없음 (영속 연결)
* `save 900 1` : 900초(15분) 동안 최소 1회 변경 발생 시 RDB 스냅샷 저장

---

### Valkey 캐시 저장소 (`valkey-cache`)

* Valkey 최신 이미지 사용
* 포트: 호스트 6380 → 컨테이너 6379 포트 연결
* 설정 파일: `./valkey/cache.conf`

#### `cache.conf` 주요 내용

```conf
port 6379
timeout 300
maxmemory 256mb
maxmemory-policy allkeys-lru
```

* `port 6379` : 컨테이너 내부 기본 포트
* `timeout 300` : 300초(5분) 이후 연결 타임아웃
* `maxmemory 256mb` : 최대 256MB 메모리 사용 제한
* `maxmemory-policy allkeys-lru` : 메모리 초과 시 가장 적게 사용된 키 우선 제거 (LRU 정책)

---

## 참고 사항

* `valkey-session`과 `valkey-cache`는 서로 독립적으로 운영되며, 각기 다른 역할과 메모리 정책을 갖습니다.
* Spring Boot 등 애플리케이션에서 이 두 인스턴스를 각각 세션 저장소와 캐시 저장소로 분리해 사용할 수 있습니다.
* PostgreSQL과 MongoDB는 각각 로컬 개발 환경에서 기본 포트로 접근 가능합니다.

---

## 컨테이너 정지 및 삭제

```bash
docker compose -f docker-compose-db.yml down
```

---

### Etc

1. 아직 MongoDB와 PostGre에 필요한 정확한 테이블 생성 쿼리문은 설계가 되지 않아서 비어놨습니다.

---

