## lec 17 - SQLAlchemy 소개 및 CRUD 실습

### 1. ORM(Object-Relational Mapping)의 필요성

- 기존 sqlite3 방식은 SQL 문자열을 직접 작성하여 데이터베이스를 조작했다.
- 프로젝트 규모가 커질수록 SQL 문 관리가 어려워지고 오타나 문법 오류를 실행 전 발견하기 어렵다.
- ORM은 Python 객체와 데이터베이스 테이블을 연결해주는 역할을 한다.
- 개발자는 SQL 대신 Python 코드로 데이터를 다룰 수 있다.
- IDE 자동완성, 타입 검사 등의 도움을 받을 수 있어 생산성과 유지보수성이 향상된다.

비교

| 작업 | sqlite3 | SQLAlchemy |
|--------|--------|--------|
| 등록 | INSERT 문 직접 작성 | db.add() |
| 조회 | SELECT 문 직접 작성 | select() |
| 삭제 | DELETE 문 직접 작성 | db.delete() |
| 수정 | UPDATE 문 직접 작성 | 객체 속성 변경 후 commit |
| 개발 편의성 | 낮음 | 높음 |

---

### 2. SQLAlchemy 설치 및 개발 환경 구성

```bash
pip install SQLAlchemy
```

#### 백엔드 실행

```bash
cd fullstack-practice/backend

uv venv

uv pip install -r requirements.txt

uv run fastapi dev main.py
```

#### 가상환경 재생성

```bash
rm -rf .venv
```

#### 프론트엔드 실행

```bash
cd fullstack-practice/frontend

npm install

npm run dev
```

---

### 3. SQLAlchemy 핵심 구성 요소

#### Engine

- 실제 데이터베이스와 연결을 담당
- DB 파일 위치 및 연결 설정 관리

#### SessionLocal

- 조회, 생성, 수정, 삭제 작업을 수행하는 세션 생성기
- 요청마다 새로운 세션을 생성하여 사용

#### Base

- 모든 ORM 모델이 상속받는 기본 클래스
- 테이블과 클래스 매핑 규칙 제공

#### Model

- 실제 데이터베이스 테이블을 표현하는 Python 클래스
- 컬럼 정보를 정의

---

### 4. 데이터베이스 연결 설정

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

DATABASE_URL = "sqlite:///./blog.db"

engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False}
)

SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
)

class Base(DeclarativeBase):
    pass
```

구성 요소 역할

- DATABASE_URL → 연결할 DB 경로
- engine → DB 연결 관리
- SessionLocal → 세션 생성
- Base → 모델 정의용 부모 클래스

---

### 5. Model 정의하기

- Python 클래스를 DB 테이블과 연결한다.
- 각 속성이 데이터베이스 컬럼이 된다.
- `__tablename__` 으로 실제 테이블명을 지정한다.

```python
class Post(Base):
    __tablename__ = "posts"

    id = mapped_column(Integer, primary_key=True)
    title = mapped_column(String(200), nullable=False)
    content = mapped_column(Text, nullable=False)
    created_at = mapped_column(
        DateTime,
        default=lambda: datetime.now(timezone.utc)
    )
```

컬럼 설명

| 컬럼 | 역할 |
|--------|--------|
| id | 기본키(PK) |
| title | 게시글 제목 |
| content | 게시글 내용 |
| created_at | 작성 시간 |

테이블 생성

```python
Base.metadata.create_all(bind=engine)
```

- 서버 시작 시 테이블이 없으면 자동 생성된다.

---

### 6. Session 관리와 Dependency Injection

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

역할

- 요청마다 세션 생성
- 요청 종료 시 자동 반환
- DB 연결 누수 방지
- 중복 코드 제거

---

### 7. 데이터 조회(Read)

#### 전체 조회

```python
db.scalars(select(Post)).all()
```

실행 과정

1. select(Post)
2. DB 실행
3. 결과를 객체 리스트로 반환

```python
@app.get("/posts")
def get_posts(db: Session = Depends(get_db)):
    return db.scalars(select(Post)).all()
```

---

#### 단건 조회

```python
stmt = select(Post).where(Post.id == 1)

post = db.scalar(stmt)
```

- 조건에 맞는 객체 한 건 반환
- 없으면 None 반환

헬퍼 함수

```python
def _get_post_or_none(
    db: Session,
    post_id: int
) -> Post | None:
    return db.scalar(
        select(Post).where(Post.id == post_id)
    )
```

---

### 8. 트랜잭션(Transaction)

트랜잭션은 더 이상 나눌 수 없는 작업 단위이다.

예시

1. 계좌 A 출금
2. 계좌 B 입금

입금 실패 시 출금도 취소되어야 한다.

이를 위해 사용하는 것이 Rollback이다.

---

### 9. 데이터 생성(Create)

생성 과정

#### 1단계. 객체 생성

```python
post = Post(
    title="제목",
    content="내용"
)
```

#### 2단계. 세션 등록

```python
db.add(post)
```

- 아직 DB 저장 전 상태

#### 3단계. DB 반영

```python
db.commit()
```

- 실제 데이터 저장

#### 4단계. 최신 데이터 동기화

```python
db.refresh(post)
```

- 자동 생성된 id
- created_at

값을 다시 가져온다.

#### 예외 발생 시

```python
db.rollback()
```

- 작업 전체 취소
- 데이터 무결성 보장

---

### 10. 게시글 생성 API

```python
@app.post("/posts")
def create_post(
    data: PostCreate,
    db: Session = Depends(get_db)
):
    try:
        post = Post(
            title=data.title,
            content=data.content
        )

        db.add(post)
        db.commit()
        db.refresh(post)

        return post

    except Exception as e:
        db.rollback()
        raise HTTPException(
            status_code=500,
            detail=str(e)
        )
```

트랜잭션 흐름

```text
객체 생성 → add() → commit() → refresh() → 응답 반환
```

---

### 11. 데이터 수정(Update)

ORM에서는 객체를 수정한 후 commit 하면 자동으로 UPDATE SQL이 생성된다.

```python
post = db.scalar(
    select(Post).where(Post.id == 1)
)

post.title = "새 제목"

db.commit()
```

동작 과정

1. 게시글 조회
2. 객체 속성 수정
3. commit
4. SQLAlchemy가 UPDATE SQL 자동 생성

---

### 12. 게시글 수정 API

```python
@app.put("/posts/{post_id}")
def update_post(
    post_id: int,
    data: PostUpdate,
    db: Session = Depends(get_db)
):
    post = _get_post_or_none(
        db,
        post_id
    )

    if not post:
        raise HTTPException(
            status_code=404
        )

    try:
        if data.title is not None:
            post.title = data.title

        if data.content is not None:
            post.content = data.content

        db.commit()
        db.refresh(post)

        return post

    except Exception as e:
        db.rollback()
```

---

### 13. 데이터 삭제(Delete)

삭제 과정

1. 삭제 대상 조회
2. 삭제 예약
3. commit

```python
post = db.scalar(
    select(Post).where(Post.id == 1)
)

db.delete(post)

db.commit()
```

---

### 14. 게시글 삭제 API

```python
@app.delete("/posts/{post_id}")
def delete_post(
    post_id: int,
    db: Session = Depends(get_db)
):
    post = _get_post_or_none(
        db,
        post_id
    )

    if not post:
        raise HTTPException(
            status_code=404
        )

    try:
        db.delete(post)
        db.commit()

    except Exception as e:
        db.rollback()
```

---

### 15. CRUD 전체 흐름 정리

#### Create

```python
db.add()
db.commit()
db.refresh()
```

#### Read

```python
db.scalar()
db.scalars()
```

#### Update

```python
객체 조회 → 속성 수정 → db.commit()
```

#### Delete

```python
db.delete()
db.commit()
```

---

### 핵심 정리

- SQLAlchemy는 ORM 라이브러리로 Python 객체와 DB 테이블을 연결한다.
- Engine → DB 연결 관리
- Session → CRUD 작업 수행
- Base → 모델 정의 기준 클래스
- Model → 실제 테이블 구조 정의
- 조회는 select() 사용
- 생성은 add → commit → refresh
- 수정은 객체 값 변경 후 commit
- 삭제는 delete 후 commit
- 데이터 변경 작업은 항상 Transaction으로 관리하며 오류 발생 시 rollback으로 복구한다.
```**````