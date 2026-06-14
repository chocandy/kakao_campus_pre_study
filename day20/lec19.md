## lec 19 - 풀스택 통합 실습 복습

### 1. 실습 목표

이번 실습은 프론트엔드와 백엔드가 연결된 상태에서 게시글 검색과 삭제 기능을 구현하고, 기존 `fetch` 코드를 `axios`로 리팩토링하는 흐름으로 진행되었다.

핵심 흐름

```text
Next.js 화면 → API 요청 → FastAPI → DB 데이터 조회/삭제 → 화면 반영
```

---

### 2. 실습 1 - Fetch 기반 검색 기능 구현

검색 페이지에서 전체 게시글을 먼저 불러온 뒤, 사용자가 입력한 검색어를 기준으로 화면에서 게시글 목록을 필터링하는 기능을 구현했다.

진행 순서

```text
백엔드 실행 → 프론트엔드 실행 → 환경 변수 설정 → 게시글 목록 요청 → 검색어 기반 필터링
```

---

### 3. Direct Fetch 방식

Direct Fetch는 브라우저가 FastAPI 서버를 직접 호출하는 방식이다.

```text
브라우저 → FastAPI
```

특징

- 구조가 단순하다.
- 클라이언트 컴포넌트에서 바로 데이터를 요청한다.
- FastAPI 주소가 브라우저에 노출된다.
- 브라우저가 직접 다른 서버에 요청하므로 CORS 설정이 필요하다.
- 클라이언트에서 사용할 환경 변수는 `NEXT_PUBLIC_` 접두사를 붙여야 한다.

예시

```text
NEXT_PUBLIC_FASTAPI_URL=http://localhost:8000
```

---

### 4. Route Handler 방식

Route Handler는 Next.js 서버가 FastAPI 요청을 대신 처리하는 방식이다.

```text
브라우저 → Next.js Route Handler → FastAPI
```

특징

- 브라우저는 `/api/search`만 호출한다.
- 실제 FastAPI 주소는 서버에서만 사용된다.
- 백엔드 주소가 브라우저에 노출되지 않는다.
- 서버 간 통신이므로 CORS 문제를 피할 수 있다.
- `app/api/search/route.ts` 파일에서 API 요청을 처리한다.

---

### 5. 환경 변수 구분

```text
FASTAPI_URL
→ 서버 전용 환경 변수
→ Server Action, Route Handler에서 사용

NEXT_PUBLIC_FASTAPI_URL
→ 브라우저 공개용 환경 변수
→ Client Component에서 사용
```

정리

```text
서버에서만 쓰는 값 → FASTAPI_URL
브라우저에서도 필요한 값 → NEXT_PUBLIC_FASTAPI_URL
```

---

### 6. 검색 기능 구현 흐름

검색 기능은 다음 순서로 동작한다.

```text
검색 페이지 접속
→ useEffect 실행
→ 게시글 전체 목록 요청
→ 결과를 results 상태에 저장
→ 사용자가 검색어 입력
→ title 또는 content에 검색어가 포함된 게시글만 필터링
→ 필터링된 목록 렌더링
```

필터링 핵심 로직

```tsx
const filtered = results.filter(
  (post) =>
    post.title.includes(query) ||
    post.content.includes(query)
);
```

---

### 7. fetch 사용 흐름

`fetch`를 사용할 때는 요청 성공 여부와 JSON 변환을 직접 처리해야 한다.

```text
fetch 요청
→ 응답 성공 여부 확인
→ res.json()으로 데이터 변환
→ state에 저장
→ 에러 처리
→ 로딩 종료
```

기억할 점

- `fetch`는 404, 500 같은 응답도 자동으로 에러 처리하지 않는다.
- 그래서 `if (!res.ok)` 검사가 필요하다.
- 응답 데이터는 직접 `res.json()`으로 변환해야 한다.

---

### 8. 실습 2 - axios로 리팩토링

실습 2에서는 기존 `fetch` 기반 코드를 `axios`로 교체했다.

대상 파일

```text
search/page.tsx
→ 게시글 목록 조회 요청

DeleteButton.tsx
→ 게시글 삭제 요청
```

진행 순서

```text
axios 설치
→ import axios
→ fetch 요청을 axios.get / axios.delete로 교체
→ try-catch-finally 구조로 정리
→ Axios 에러 처리 추가
```

설치

```bash
npm install axios
```

---

### 9. axios를 사용하는 이유

axios는 fetch보다 HTTP 요청 코드를 더 간결하게 작성할 수 있다.

장점

- JSON 변환을 자동으로 처리한다.
- 응답 데이터는 `response.data`로 바로 접근한다.
- 2xx가 아닌 응답은 자동으로 catch로 이동한다.
- `axios.get()`, `axios.post()`, `axios.delete()`처럼 메서드가 직관적이다.
- `axios.isAxiosError()`로 에러를 구분할 수 있다.

---

### 10. fetch와 axios 비교

| 구분 | fetch | axios |
|---|---|---|
| JSON 변환 | `res.json()` 필요 | 자동 변환 |
| 에러 처리 | `if (!res.ok)` 직접 확인 | 2xx 외 자동 에러 |
| 데이터 접근 | 변환 후 data 사용 | `res.data` |
| 삭제 요청 | `fetch(url, { method: "DELETE" })` | `axios.delete(url)` |
| 에러 메시지 | 직접 응답 body 파싱 | `err.response.data.detail` |

---

### 11. axios 기반 검색 요청 흐름

```text
useEffect 실행
→ async 함수 선언
→ axios.get("/api/search")
→ 성공 시 setResults(res.data)
→ 실패 시 setError(...)
→ finally에서 setLoading(false)
```

핵심 구조

```tsx
try {
  const res = await axios.get<Post[]>(`${BASE_PATH}/api/search`);
  setResults(res.data);
} catch (err) {
  if (axios.isAxiosError(err)) {
    setError(err.response?.data?.detail ?? "게시글을 불러오는 데 실패했습니다");
  }
} finally {
  setLoading(false);
}
```

---

### 12. axios 기반 삭제 요청 흐름

```text
삭제 버튼 클릭
→ confirm으로 삭제 여부 확인
→ axios.delete 요청
→ 성공 시 목록 페이지로 이동
→ 실패 시 alert로 에러 표시
```

핵심 구조

```tsx
try {
  await axios.delete(`${BASE_PATH}/api/posts/${postId}`);
  window.location.href = `${BASE_PATH}/posts`;
} catch (err) {
  if (axios.isAxiosError(err)) {
    alert(err.response?.data?.detail ?? "게시글 삭제에 실패했습니다");
  }
}
```

---

### 13. 전체 실습 흐름 정리

```text
1. 백엔드 실행
2. 프론트엔드 실행
3. 환경 변수 설정
4. 검색 페이지 구현
5. Direct Fetch 방식 확인
6. Route Handler 방식 확인
7. fetch 기반 요청 동작 확인
8. axios 설치
9. fetch 코드를 axios로 리팩토링
10. 검색과 삭제 기능 재확인
```

---

### 핵심 정리

```text
Direct Fetch
→ 브라우저가 FastAPI 직접 호출
→ NEXT_PUBLIC_ 환경 변수 필요
→ CORS 설정 필요

Route Handler
→ 브라우저가 Next.js 서버 호출
→ Next.js 서버가 FastAPI 호출
→ 백엔드 주소 숨김
→ CORS 문제 없음

fetch
→ 기본 내장 함수
→ 직접 JSON 변환
→ 직접 에러 확인 필요

axios
→ HTTP 요청 라이브러리
→ JSON 자동 변환
→ 에러 처리 편리
→ 코드가 더 간결함
```