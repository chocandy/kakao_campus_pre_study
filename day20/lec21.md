## 🗓 lec 21 - E2E 검증 & Docker/AWS 개요

### 1. E2E(End-to-End) 검증

E2E 검증은 사용자의 실제 사용 흐름을 처음부터 끝까지 따라가며 시스템 전체가 정상 동작하는지 확인하는 과정이다.

배포 후 구조

```text
사용자
→ Vercel (Next.js)
→ Railway (FastAPI)
→ SQLite
```

로컬에서는 모든 요소가 한 컴퓨터에 있었지만, 배포 후에는 프론트엔드와 백엔드가 서로 다른 서버에서 동작한다.

따라서 각 계층이 정상적으로 연결되어 있는지 확인하는 과정이 필요하다.

---

### 2. E2E 검증 순서

배포된 서비스에서 CRUD 전체 기능을 순서대로 확인한다.

#### 게시글 생성 (POST)

```text
게시글 작성
→ 작성 버튼 클릭
→ FastAPI POST 요청
→ SQLite 저장
→ 목록 페이지 이동
→ 새 게시글 표시 확인
```

확인 내용

- 작성 후 목록 페이지 이동 여부
- 생성한 게시글 표시 여부

---

#### 게시글 조회 (GET)

```text
목록 페이지
→ 게시글 선택
→ 상세 페이지 이동
→ 내용 확인
```

추가 확인

```text
Railway Swagger UI
→ GET /posts
→ 실제 저장 데이터 확인
```

---

#### 검색 기능 확인

```text
검색 페이지 접속
→ 검색어 입력
→ 결과 필터링 확인
→ 새로고침 후 결과 유지 확인
```

확인 내용

- Query Parameter 정상 동작 여부
- URL 기반 검색 결과 유지 여부

---

#### 게시글 수정 (PUT)

```text
게시글 선택
→ 수정 페이지 이동
→ 제목 또는 내용 수정
→ 저장
→ 변경 내용 반영 확인
```

---

#### 게시글 삭제 (DELETE)

```text
삭제 버튼 클릭
→ 삭제 승인
→ 목록 페이지 이동
→ 게시글 제거 확인
```

---

### 3. Railway Swagger UI 확인

Swagger UI는 실제 백엔드 상태를 확인하는 도구이다.

접속

```text
https://railway-url/docs
```

활용

```text
GET /posts
→ 현재 저장된 게시글 확인

POST /posts
→ 데이터 생성 테스트

PUT /posts
→ 수정 테스트

DELETE /posts
→ 삭제 테스트
```

목적

```text
프론트엔드 화면 ≠ 실제 DB 상태

직접 확인
```

---

### 4. Vercel + Railway 구조 복습

현재까지 사용한 구조

```text
사용자 → Vercel → Railway → SQLite
```

장점

- 빠른 배포
- 간단한 설정
- 무료 플랜 제공
- GitHub 자동 연동

적합한 경우

```text
개인 프로젝트

포트폴리오

MVP 개발
```

---

### 5. Docker + AWS와의 차이

다음 주부터는 Docker와 AWS를 이용한 배포를 학습한다.

구조

```text
사용자 → AWS EC2 → Docker Container → Backend → Database
```

---

### 6. Vercel과 Docker/AWS 비교

| 항목 | Vercel + Railway | Docker + AWS |
|--------|--------|--------|
| 난이도 | 낮음 | 높음 |
| 배포 속도 | 빠름 | 상대적으로 복잡 |
| 서버 제어 | 제한적 | 완전 제어 |
| 비용 | 무료 플랜 존재 | 비용 발생 |
| 확장성 | 플랫폼 제공 | 직접 구성 |
| 운영 난이도 | 낮음 | 높음 |

---

### 7. Vercel의 한계

Vercel은 편리하지만 제약도 존재한다.

#### 실행 시간 제한

```text
Serverless Function

≈ 10초 제한
```

문제

```text
장시간 연산

실시간 처리

스트리밍 작업

부적합
```

---

#### 실행 환경 제한

```text
Node.js

Python 일부
```

문제

```text
AI 모델

대규모 데이터 처리

특수 라이브러리

제약 존재
```

---

#### 서버 제어 불가

```text
서버 접근 불가

OS 설정 불가

성능 튜닝 제한
```

---

### 8. Docker + AWS가 필요한 상황

실무 환경에서는 다음과 같은 이유로 Docker + AWS를 사용한다.

예시

```text
대규모 SaaS 서비스

실시간 채팅

WebSocket 서버

AI 모델 서빙

기업 내부 시스템

고트래픽 서비스
```

필요 기능

```text
서버 직접 제어

확장성

모니터링

보안 정책 적용
```

---

### 9. SQLite의 한계

현재 프로젝트는 SQLite를 사용한다.

특징

```text
파일 기반 DB
```

장점

```text
설치 필요 없음

간단한 프로젝트 적합
```

---

한계

```text
동시 접속 증가 시 충돌 가능

여러 서버가 DB 공유 불가

대규모 서비스 부적합
```

실무에서는

```text
PostgreSQL

MySQL
```

같은 네트워크 기반 DB를 사용한다.

---

### 10. 지금까지의 학습 흐름

#### 1~3주차

```text
Python

FastAPI

SQLite

SQLAlchemy
```

↓

백엔드 API 구축

---

#### 4주차

```text
Next.js

Server Action

Full Stack 통합
```

↓

프론트엔드 구축


---

### 핵심 정리

```text
E2E 검증

→ 사용자의 실제 사용 흐름 검증

→ CRUD 전체 기능 확인

→ 프론트와 백엔드 연결 상태 확인
```

```text
현재 구조

사용자
→ Vercel
→ Railway
→ SQLite
```

```text
Vercel + Railway

장점
→ 쉬운 배포
→ 무료 사용 가능

단점
→ 서버 제어 제한
→ 실행 환경 제약
```

```text
다음 단계

Docker → 애플리케이션 컨테이너화

AWS → 직접 서버 운영

실무 환경에 가까운 배포 경험
```