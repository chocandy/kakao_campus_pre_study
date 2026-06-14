## lec 18 - End-to-End 흐름 확인

### 1. Full Stack 구조 이해

이번 실습에서는 프론트엔드부터 데이터베이스까지 하나의 요청이 어떻게 전달되는지 전체 흐름을 확인했다.

구성

```text
Next.js → Server Action → FastAPI → SQLAlchemy → SQLite
```

각 계층은 역할이 분리되어 있으며, 하나의 요청이 여러 계층을 거쳐 처리된다.

---

### 2. 계층별 역할

| 계층 | 역할 |
|--------|--------|
| Next.js | 사용자 화면 제공 |
| Server Action | 서버에서 API 호출 |
| FastAPI | 요청 처리 및 비즈니스 로직 수행 |
| SQLAlchemy | Python 객체와 DB 연결 |
| SQLite | 데이터 영구 저장 |

핵심은 각 계층이 자신의 역할만 담당한다는 점이다.

---

### 3. 게시글 생성 요청 흐름

사용자가 게시글을 작성하면 다음 순서로 처리된다.

```text
1. 사용자가 작성 버튼 클릭

2. Server Action 실행

3. FastAPI API 호출

4. SQLAlchemy가 Post 객체 생성

5. SQLite에 데이터 저장

6. 응답 반환

7. 게시글 목록 갱신
```

---

### 4. Server Action의 역할

Server Action은 Next.js 서버에서 실행되는 함수이다.

```text
브라우저 → Server Action → FastAPI
```

역할

- 폼 데이터 처리
- API 호출
- 캐시 갱신
- 페이지 이동

기존처럼 API Route를 별도로 만들지 않아도 서버 로직을 실행할 수 있다.

---

### 5. CORS가 발생하지 않는 이유

일반적인 구조

```text
브라우저 → 다른 서버 요청
```

브라우저가 직접 다른 출처에 요청하기 때문에 CORS 문제가 발생할 수 있다.

Server Action 사용

```text
브라우저 → Next.js 서버 → FastAPI 서버
```

브라우저가 아닌 서버가 API를 호출한다.

따라서 서버 간 통신이 되어 CORS 설정 없이도 동작할 수 있다.

---

### 6. 환경 변수 사용

```env
FASTAPI_URL=http://localhost:8000
```

환경 변수 사용 이유

- API 주소 하드코딩 방지
- 개발/운영 환경 분리
- 보안성 향상

Server Component와 Server Action에서는 환경변수를 직접 사용할 수 있다.

---

### 7. 지금까지의 전체 흐름 정리

#### Frontend

```text
사용자 입력 → 화면 렌더링
```

기술

```text
Next.js
```

---

#### Server Action

```text
폼 데이터 처리 → API 호출 → 캐시 갱신
```

기술

```text
Next.js
```

---

#### Backend

```text
요청 수신 → 유효성 검증 → 비즈니스 로직 수행
```

기술

```text
FastAPI
```

---

#### ORM

```text
Python 객체 ↔ SQL 변환
```

기술

```text
SQLAlchemy
```

---

#### Database

```text
데이터 영구 저장
```

기술

```text
SQLite
```

---

### 핵심 정리 ⭐

```text
사용자 → Next.js → Server Action → FastAPI → SQLAlchemy → SQLite
```

- Next.js는 화면을 담당한다.
- Server Action은 서버에서 API를 호출한다.
- FastAPI는 요청을 처리한다.
- SQLAlchemy는 Python 객체를 SQL로 변환한다.
- SQLite는 데이터를 저장한다.
- 하나의 요청은 위 계층들을 순서대로 거쳐 처리된다.
- Server Action을 사용하면 서버 간 통신이 되어 CORS 문제를 피할 수 있다.