## lec 16 - Next.js 기초 & 데이터 처리 복습

### 1. Next.js와 React의 차이

- React는 UI를 만들기 위한 라이브러리로, 라우팅이나 데이터 관리 구조를 개발자가 직접 구성해야 한다.
- Next.js는 React 기반 프레임워크로, 프로젝트 구조와 동작 방식이 정해져 있어 보다 체계적으로 개발할 수 있다.
- 파일과 폴더 구조만으로 라우팅이 자동 생성되며, 서버 렌더링과 데이터 페칭 기능을 기본 제공한다.

---

### 2. App Router와 File-based Routing

- Next.js의 App Router는 폴더 구조를 URL 경로로 사용하는 방식이다.
- `page.tsx`는 해당 경로의 화면을 담당한다.
- `layout.tsx`는 여러 페이지에서 공통으로 사용하는 레이아웃을 구성한다.
- `loading.tsx`는 데이터 로딩 중 사용자에게 보여줄 UI를 정의한다.
- 동적 라우팅은 `[id]` 형태의 폴더를 사용하여 URL 파라미터를 처리한다.

예시

| 파일 경로 | URL |
|------------|------|
| app/page.tsx | / |
| app/posts/page.tsx | /posts |
| app/posts/[postId]/page.tsx | /posts/1 |

---

### 3. Server Component에서 데이터 가져오기

- Next.js의 Server Component는 서버에서 실행된다.
- 컴포넌트 자체를 `async` 함수로 작성할 수 있다.
- 서버에서 직접 API 호출 및 데이터 조회가 가능하다.
- 데이터 조회 결과를 서버에서 HTML로 렌더링한 뒤 브라우저에 전달한다.
- 클라이언트로 전송되는 JavaScript 양이 줄어 초기 로딩 속도가 빨라진다.

```tsx
export default async function PostsPage() {
  const res = await fetch("http://localhost:8000/posts");
  const posts = await res.json();

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

---

### 4. Streaming과 loading.tsx

- 기존 SSR은 서버가 모든 데이터를 준비할 때까지 사용자가 빈 화면을 보게 되는 문제가 있다.
- Next.js는 Streaming 기능을 통해 준비된 화면부터 먼저 전송할 수 있다.
- `loading.tsx`를 활용하면 스켈레톤 UI나 로딩 화면을 먼저 보여줄 수 있다.
- 데이터가 준비되는 즉시 나머지 화면을 순차적으로 렌더링한다.

동작 흐름

1. 브라우저 요청
2. 레이아웃 + 로딩 UI 전송
3. 데이터 조회 진행
4. 완성된 컴포넌트 전송
5. 화면 업데이트

---

### 5. Promise.all을 이용한 병렬 데이터 요청

- 여러 API 요청이 서로 의존하지 않는다면 동시에 실행하는 것이 효율적이다.
- 순차 실행 시 각 요청 시간이 누적된다.
- `Promise.all()`을 사용하면 여러 요청을 병렬로 처리할 수 있다.
- 전체 응답 시간은 가장 오래 걸리는 요청 시간 수준으로 감소한다.

```tsx
const [user, posts] = await Promise.all([
  fetchUser(),
  fetchPosts(),
]);
```

---

### 6. Server Component와 Client Component

#### Server Component

- Next.js 기본 컴포넌트 형태
- 서버에서 실행
- async/await 사용 가능
- DB 접근 가능
- 환경변수 접근 가능
- useState, useEffect 사용 불가
- 이벤트 핸들러 사용 불가

#### Client Component

- `"use client"` 선언 필요
- 브라우저에서 실행
- 사용자 입력 및 인터랙션 처리 가능
- useState, useEffect 사용 가능
- 이벤트 핸들러 사용 가능
- 서버 환경변수 및 DB 직접 접근 불가

```tsx
"use client";

import { useState } from "react";

export default function SearchPage() {
  const [query, setQuery] = useState("");

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
    />
  );
}
```

---

### 7. Server Actions

- `"use server"`를 선언한 서버 전용 함수이다.
- 별도의 API Route 없이 클라이언트에서 서버 로직을 실행할 수 있다.
- 주로 데이터 생성(Create), 수정(Update), 삭제(Delete) 작업에 사용한다.
- 환경변수를 안전하게 사용할 수 있다.
- 데이터 변경 후 캐시를 갱신하고 페이지 이동도 수행할 수 있다.

```tsx
"use server";

export async function createPost(formData: FormData) {
  const title = formData.get("title");

  await fetch(`${process.env.FASTAPI_URL}/posts`, {
    method: "POST",
    body: JSON.stringify({ title }),
  });
}
```

```tsx
<form action={createPost}>
  <input name="title" />
  <button type="submit">작성</button>
</form>
```

---

### 8. SSR 한계와 해결 전략

#### SSR의 장점

- 초기 로딩 속도가 빠르다.
- SEO에 유리하다.
- 서버에서 HTML을 완성해 전달한다.

#### SSR의 한계

- 사용자 인터랙션 처리에 제약이 있다.
- 실시간 상태 변경에 불편함이 있다.

#### 해결 방법

- 데이터 조회 및 렌더링 → Server Component
- 입력창, 버튼, 상태관리 → Client Component
- 데이터 변경 작업 → Server Actions

이 세 가지를 적절히 조합하여 SSR의 성능과 CSR의 인터랙션 장점을 모두 활용할 수 있다.

