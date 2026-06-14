## 🗓 lec 20 - Vercel & Railway 배포 실습

### 1. 실습 목표

이번 실습은 로컬 환경에서만 동작하던 Full Stack 프로젝트를 실제 인터넷에 배포하는 과정이다.

배포 구조

```text
Frontend (Next.js) → Vercel 배포

Backend (FastAPI) → Railway 배포
```

최종 목표

```text
사용자 → Vercel(Next.js) → Railway(FastAPI) → SQLite
```

---

### 2. Vercel과 Railway 이해

#### Vercel

- Next.js 제작사가 운영하는 배포 플랫폼
- GitHub와 연동하여 자동 배포 가능
- Preview 배포 제공
- Next.js 프로젝트 배포에 최적화

배포 결과

```text
https://프로젝트명.vercel.app
```

---

#### Railway

- 컨테이너 기반 서버 호스팅 서비스
- Python, Node.js 등 다양한 백엔드 배포 가능
- FastAPI 서버를 항상 실행 상태로 유지
- 공개 URL 제공

배포 결과

```text
https://프로젝트명.up.railway.app
```

---

### 3. 프로젝트 배포 구조

프로젝트 구조

```text
fullstack-practice

├── frontend
│   └── Vercel 배포

└── backend
    └── Railway 배포
```

배포 순서

```text
1. GitHub 업로드

→ 2. Railway에 FastAPI 배포

→ 3. Railway URL 생성

→ 4. Vercel에 Next.js 배포

→ 5. 환경변수 연결

→ 6. 배포 테스트
```

---

### 4. GitHub 업로드

배포 전 반드시 GitHub에 코드가 올라가 있어야 한다.

진행 순서

```text
프로젝트 Git 초기화
→ Commit 생성
→ GitHub Repository 생성
→ Push
```

배포 플랫폼은 GitHub 저장소를 기반으로 자동 빌드를 수행한다.

---

### 5. Railway 배포 실습

#### Step 1. Railway 프로젝트 생성

```text
Railway 로그인

→ GitHub 연동

→ Repository 선택
```

---

#### Step 2. Backend 연결

설정

```text
Root Directory
→ backend
```

실행 명령어

```text
fastapi run main.py --host 0.0.0.0 --port $PORT
```

역할

```text
Railway 서버 실행
→ 외부 접속 허용
→ Railway 포트 사용
```

---

#### Step 3. Railway URL 생성

설정

```text
Settings
→ Networking
→ Generate Domain
```

결과

```text
https://xxx.up.railway.app
```

생성된 URL은 이후 Vercel 환경변수에 사용한다.

---

### 6. CORS 설정 수정

배포 후에는 localhost가 아닌 실제 Vercel 주소를 허용해야 한다.

기존

```text
http://localhost:3000
```

배포 후

```text
http://localhost:3000

+

https://프로젝트명.vercel.app
```

구성 방식

```python
FRONTEND_URL
↓
origins 리스트 생성
↓
allow_origins 설정
```

목적

```text
배포된 프론트엔드의 API 접근 허용
```

---

### 7. Vercel 배포 실습

#### Step 1. GitHub 연동

```text
Vercel 로그인 → Import Project → GitHub Repository 선택
```

---

#### Step 2. Frontend 지정

설정

```text
Root Directory → frontend
```

Vercel은 frontend 폴더만 빌드한다.

---

#### Step 3. 환경변수 등록

필수 설정

```text
FASTAPI_URL = Railway URL
```

예시

```text
FASTAPI_URL = https://xxx.up.railway.app
```

특징

```text
서버 전용 환경변수

NEXT_PUBLIC_ 불필요
```

---

#### Step 4. Deploy

진행

```text
Deploy 클릭 → Build → URL 생성
```

결과

```text
https://프로젝트명.vercel.app
```

---

### 8. Railway 환경변수 수정

Vercel URL 생성 후 Railway에 다시 등록한다.

설정

```text
FRONTEND_URL = https://프로젝트명.vercel.app
```

또는

```text
CORS_ORIGINS

=

https://프로젝트명.vercel.app
```

수정 후

```text
Redeploy
```

---

### 9. 배포 확인

확인 순서

```text
1. Railway URL 접속 → Swagger UI 확인

2. Vercel URL 접속 → 화면 로딩 확인

3. 게시글 목록 확인 → API 정상 연결 확인

4. 검색 기능 테스트 → 프론트/백엔드 연동 확인
```

---

### 10. 트러블슈팅

#### 500 에러 발생

확인

```text
Railway 로그 확인

requirements.txt 확인

실행 명령어 확인
```

---

#### CORS 에러 발생

확인

```text
Railway 환경변수

FRONTEND_URL 또는 ORS_ORIGINS
```

주의

```text
http
https

구분
```

---

#### 데이터가 안 보임

확인

```text
Vercel 환경변수

FASTAPI_URL
```

수정 후

```text
Redeploy 필요
```

---

### 11. 전체 배포 흐름 정리

```text
코드 작성
→ GitHub Push
→ Railway Backend 배포
→ Railway URL 생성
→ FastAPI CORS 설정
→ Vercel Frontend 배포
→ FASTAPI_URL 설정
→ Railway에 Vercel URL 등록
→ 재배포
→ 서비스 테스트
```

---

### 핵심 정리

```text
Vercel
→ Next.js 배포

Railway
→ FastAPI 배포

GitHub
→ 배포 소스 관리

FASTAPI_URL
→ Railway 주소 저장

FRONTEND_URL
→ Vercel 주소 저장

배포 순서

GitHub
→ Railway
→ Railway URL 생성
→ Vercel
→ 환경변수 연결
→ CORS 설정
→ 테스트
```

```text
최종 구조

사용자
→ Vercel (Next.js)
→ Railway (FastAPI)
→ SQLite
```