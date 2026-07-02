# Docker PostgreSQL 사용 가이드

로컬 개발 환경에서는 Docker로 PostgreSQL을 실행해 백엔드와 데이터베이스를 연결합니다.

## 사용 목적

- 팀원별 동일한 PostgreSQL 개발 환경 구성
- FastAPI 백엔드와 DB 연동 테스트
- SQLAlchemy 모델 및 Alembic 마이그레이션 검증
- 실제 서비스 DB에 영향을 주지 않는 로컬 테스트

## 권장 구성

```text
FastAPI Backend
-> SQLAlchemy 2.x
-> Alembic
-> PostgreSQL Docker Container
```

## docker-compose.yml 예시

백엔드 저장소 루트에 `docker-compose.yml` 파일을 만들고 아래 내용을 사용할 수 있습니다.

```yaml
services:
  postgres:
    image: postgres:16
    container_name: ai_speech_postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ai_speech
      POSTGRES_USER: ai_speech_user
      POSTGRES_PASSWORD: ai_speech_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## 실행

```bash
docker compose up -d
```

## 종료

```bash
docker compose down
```

## 데이터까지 삭제

주의: 아래 명령은 로컬 PostgreSQL 데이터를 모두 삭제합니다.

```bash
docker compose down -v
```

## 백엔드 환경 변수 예시

FastAPI 백엔드에서는 `.env`에 아래처럼 DB 접속 정보를 둘 수 있습니다.

```env
DATABASE_URL=postgresql+psycopg://ai_speech_user:ai_speech_password@localhost:5432/ai_speech
```

비동기 DB 드라이버를 사용할 경우에는 아래 형식을 사용할 수 있습니다.

```env
DATABASE_URL=postgresql+asyncpg://ai_speech_user:ai_speech_password@localhost:5432/ai_speech
```

## psql 접속

```bash
docker exec -it ai_speech_postgres psql -U ai_speech_user -d ai_speech
```

## Alembic 마이그레이션 흐름

```bash
alembic revision --autogenerate -m "create initial tables"
alembic upgrade head
```

## 개발 규칙

- DB 구조 변경은 SQLAlchemy 모델과 Alembic 마이그레이션을 함께 수정합니다.
- 로컬 DB 비밀번호는 개발용으로만 사용합니다.
- 운영 DB 접속 정보는 GitHub에 올리지 않습니다.
- `.env` 파일은 `.gitignore`에 포함합니다.
- 실제 발표 자료, 음성, 영상 파일은 PostgreSQL에 직접 저장하지 않고 Storage 경로만 저장합니다.
