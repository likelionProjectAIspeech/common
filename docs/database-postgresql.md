# PostgreSQL 데이터베이스 설계

## 설계 개요

AI Speech는 PostgreSQL을 기준으로 사용자, 발표 프로젝트, 발표 자료, 슬라이드 분석, 시간 배분, 대본, 리허설, 음성 분석, 자세 분석, 시선 분석, AI 평가, Q&A, 종합 리포트, 비동기 작업 데이터를 저장합니다.

## 기본 기준

| 항목 | 기준 |
| --- | --- |
| DBMS | PostgreSQL |
| ORM | SQLAlchemy 2.x |
| 마이그레이션 | Alembic |
| 일반 PK | `BIGINT GENERATED ALWAYS AS IDENTITY` |
| 비동기 작업 PK | `UUID DEFAULT gen_random_uuid()` |
| 시간 타입 | `TIMESTAMPTZ` |
| 가변 AI 결과 | `JSONB` |
| 삭제 전략 | `deleted_at` 기반 소프트 삭제 |

## 주요 테이블

| 테이블 | 역할 |
| --- | --- |
| `users` | 회원 계정과 기본 프로필 저장 |
| `refresh_tokens` | JWT Refresh Token 발급, 만료, 폐기 관리 |
| `presentations` | 발표 프로젝트 중심 정보와 발표 및 Q&A 시간 저장 |
| `presentation_files` | 업로드된 PPT, PPTX, PDF, DOCX 파일과 파싱 상태 관리 |
| `slides` | 파싱된 슬라이드 텍스트, 이미지, 순서 저장 |
| `presentation_analyses` | 발표 전체 AI 분석 결과와 버전 저장 |
| `slide_analyses` | 슬라이드별 중요도, 복잡도, 핵심 메시지, 예상 질문 저장 |
| `slide_timings` | 슬라이드별 배정 시간과 재분배 잠금 상태 저장 |
| `slide_scripts` | 슬라이드별 발표 대본과 수정 이력 저장 |
| `rehearsals` | 발표 프로젝트별 리허설 회차와 상태 저장 |
| `rehearsal_media` | 리허설 음성 및 영상 파일 저장 위치와 메타데이터 관리 |
| `audio_analyses` | STT, 말하기 속도, 누락 및 추가 내용 등 음성 분석 요약 저장 |
| `filler_word_events` | 추임새 종류, 횟수, 발생 시각 저장 |
| `speech_events` | 긴 침묵, 반복 표현, 불필요한 추가 설명 구간 저장 |
| `pose_analyses` | 리허설 영상의 자세 분석 요약 점수 저장 |
| `pose_events` | 몸 흔들림, 고개 숙임, 얼굴 만짐 등 자세 이벤트 저장 |
| `gaze_analyses` | 교수, 화면, 바닥 방향 시선 비율과 시간 저장 |
| `gaze_events` | 장시간 화면 응시나 바닥 응시 등 시선 이벤트 저장 |
| `rehearsal_slide_results` | 리허설에서 슬라이드별 계획 시간과 실제 시간 비교 |
| `agent_evaluations` | 교수, 학생, 최종 판단 에이전트 평가와 버전 저장 |
| `evaluation_criteria` | 에이전트 평가 세부 기준별 점수와 피드백 저장 |
| `evaluation_priorities` | 최종 판단 에이전트가 산출한 개선 우선순위 저장 |
| `evaluation_priority_slides` | 개선 우선순위와 관련 슬라이드 연결 |
| `qa_sessions` | 리허설별 Q&A 연습 세션과 진행 상태 저장 |
| `qa_questions` | 교수 예상 질문, 의도, 답변 키워드 저장 |
| `qa_question_slides` | 질문과 관련된 슬라이드 연결 |
| `qa_answers` | 사용자의 텍스트 또는 음성 답변 저장 |
| `qa_answer_evaluations` | 답변의 논리성, 정확성, 간결성 등 세부 평가 저장 |
| `final_reports` | 리허설 종합 리포트와 PDF 생성 상태 저장 |
| `report_scores` | 리포트 영역별 점수 저장 |
| `rehearsal_comparisons` | 두 리허설의 비교 대상과 전체 개선율 저장 |
| `comparison_metrics` | 추임새, 시선, 몸 흔들림 등 비교 지표별 변화량 저장 |
| `jobs` | 파싱, 분석, 대본, 리포트 등 비동기 작업 상태 관리 |
| `job_steps` | 비동기 작업의 단계별 진행률과 오류 저장 |

## 핵심 관계

- `users` 1:N `presentations`
- `users` 1:N `refresh_tokens`
- `users` 1:N `jobs`
- `presentations` 1:N `presentation_files`
- `presentations` 1:N `slides`
- `presentations` 1:N `presentation_analyses`
- `presentations` 1:N `rehearsals`
- `presentations` 1:N `jobs`
- `presentation_files` 1:N `slides`
- `presentation_analyses` 1:N `slide_analyses`
- `slides` 1:N `slide_analyses`
- `slides` 1:N `slide_timings`
- `slides` 1:N `slide_scripts`
- `rehearsals` 1:N `rehearsal_media`
- `rehearsals` 1:N `agent_evaluations`
- `rehearsals` 1:N `qa_sessions`
- `rehearsals` 1:N `final_reports`
- `rehearsals` 1:1 `audio_analyses`
- `rehearsals` 1:1 `pose_analyses`
- `rehearsals` 1:1 `gaze_analyses`
- `audio_analyses` 1:N `filler_word_events`
- `audio_analyses` 1:N `speech_events`
- `pose_analyses` 1:N `pose_events`
- `gaze_analyses` 1:N `gaze_events`
- `qa_questions` 1:N `qa_answers`
- `qa_answers` 1:1 `qa_answer_evaluations`
- `jobs` 1:N `job_steps`

## 핵심 도메인 흐름

### 1. 발표 프로젝트 생성

1. 사용자가 발표 제목, 전체 발표 시간, 질의응답 시간을 입력합니다.
2. `presentations`에 발표 프로젝트가 생성됩니다.
3. 실제 발표 가능 시간은 `전체 발표 시간 - 질의응답 시간`으로 계산합니다.

### 2. 발표 자료 업로드 및 분석

1. 업로드 파일 정보는 `presentation_files`에 저장합니다.
2. 파싱된 슬라이드는 `slides`에 저장합니다.
3. 발표 전체 분석은 `presentation_analyses`에 저장합니다.
4. 슬라이드별 분석은 `slide_analyses`에 저장합니다.

### 3. 시간 배분 및 대본 생성

1. 슬라이드별 배정 시간은 `slide_timings`에 저장합니다.
2. 슬라이드별 발표 대본은 `slide_scripts`에 저장합니다.
3. 재생성 또는 수정이 필요한 경우 버전 정보를 함께 관리합니다.

### 4. 리허설 분석

1. 리허설 회차는 `rehearsals`에 저장합니다.
2. 음성 및 영상 파일 정보는 `rehearsal_media`에 저장합니다.
3. 음성 분석은 `audio_analyses`, `filler_word_events`, `speech_events`에 저장합니다.
4. 자세 분석은 `pose_analyses`, `pose_events`에 저장합니다.
5. 시선 분석은 `gaze_analyses`, `gaze_events`에 저장합니다.
6. 슬라이드별 계획 시간과 실제 시간 비교는 `rehearsal_slide_results`에 저장합니다.

### 5. AI 평가 및 리포트

1. 교수, 학생, 최종 판단 에이전트 평가는 `agent_evaluations`에 저장합니다.
2. 평가 기준별 점수와 피드백은 `evaluation_criteria`에 저장합니다.
3. 개선 우선순위는 `evaluation_priorities`에 저장합니다.
4. 종합 리포트는 `final_reports`, 영역별 점수는 `report_scores`에 저장합니다.

### 6. Q&A 훈련

1. Q&A 세션은 `qa_sessions`에 저장합니다.
2. 예상 질문은 `qa_questions`에 저장합니다.
3. 질문과 관련된 슬라이드는 `qa_question_slides`로 연결합니다.
4. 사용자 답변은 `qa_answers`에 저장합니다.
5. 답변 평가는 `qa_answer_evaluations`에 저장합니다.

### 7. 비동기 작업

1. 오래 걸리는 작업은 `jobs`에 등록합니다.
2. 작업 단계별 진행률과 오류는 `job_steps`에 저장합니다.
3. API는 작업 ID를 반환하고, 프론트엔드는 작업 상태를 조회합니다.

## 개발 시 주의사항

- 모든 사용자 데이터는 `user_id`를 기준으로 접근 권한을 검증합니다.
- 발표 자료, 음성, 영상 파일은 DB에 직접 저장하지 않고 파일 저장소 경로만 저장합니다.
- AI 분석 결과처럼 구조가 변할 수 있는 데이터는 `JSONB` 사용을 허용합니다.
- 핵심 조회 조건에는 인덱스를 추가합니다.
- 삭제가 필요한 사용자 데이터는 우선 `deleted_at` 기반 소프트 삭제를 적용합니다.
- Alembic 마이그레이션 파일은 DB 구조 변경 시 반드시 함께 작성합니다.
