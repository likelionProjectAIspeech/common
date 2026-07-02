# AI Speech Common Docs

AI 기반 교수 발표 준비 도우미 프로젝트의 공통 문서 저장소입니다.

이 저장소는 프론트엔드, 백엔드, AI 분석, 데이터베이스 설계를 함께 참고하기 위한 협업 문서를 관리합니다.

## 문서 목록

| 문서 | 설명 |
| --- | --- |
| [협업 전략 및 개발 방향](docs/collaboration-strategy.md) | 역할 분담, 기술 스택, GitHub 협업 규칙, 개발 흐름 |
| [PostgreSQL 데이터베이스 설계](docs/database-postgresql.md) | 핵심 테이블, 관계 구조, 저장 대상 |
| [Docker PostgreSQL 사용 가이드](docs/docker-postgresql.md) | 로컬 개발용 PostgreSQL 실행 방법 |

## 원본 PDF

| 파일 | 설명 |
| --- | --- |
| `cooperationStrategy.pdf` | 협업 전략 및 개발 방향 원본 문서 |
| `AI_PostgreSQL_DB.pdf` | PostgreSQL 데이터베이스 설계 원본 문서 |

## 프로젝트 저장소 구성

| 저장소 | 역할 |
| --- | --- |
| `presentation-coach-frontend` | React Native 모바일 앱 |
| `presentation-coach-backend` | FastAPI 서버, AI 분석, 문서 분석, 비동기 작업 |
| `presentation-coach-common` | ERD, API 명세서, 회의록, DB 설계, 협업 문서 |

## 기본 개발 흐름

```text
React Native App
-> FastAPI Backend
-> Redis + RQ Worker
-> AI 분석 모듈
-> PostgreSQL
-> React Native 결과 조회
```

## GitHub 협업 원칙

- `main` 브랜치에 직접 push하지 않습니다.
- 기능 개발은 `feat/...` 브랜치에서 진행합니다.
- 버그 수정은 `fix/...` 브랜치에서 진행합니다.
- 작업 완료 후 `dev` 브랜치 대상으로 Pull Request를 생성합니다.
- 검증된 변경사항만 `dev`에서 `main`으로 병합합니다.
