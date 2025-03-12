# 데이터베이스 구조
### 엔티티 관계도 (ERD)
<img src="img/gameReviewHubERD.png"/>
데이터베이스는 다음 4개의 테이블로 구성되어 있습니다.

```
users: 사용자 정보 저장

games: 게임 정보 저장

reviews: 사용자가 작성한 게임 리뷰 정보

review_likes: 리뷰에 대한 좋아요 정보
```

### 관계

```
사용자(users)는 여러 리뷰(reviews)를 작성할 수 있습니다 (1:N)

게임(games)은 여러 리뷰(reviews)를 가질 수 있습니다 (1:N)

사용자(users)는 여러 리뷰에 좋아요(review_likes)를 할 수 있습니다 (1:N)

리뷰(reviews)는 여러 좋아요(review_likes)를 받을 수 있습니다 (1:N)
```

## 테이블 구조
### users 테이블

사용자 계정 정보를 저장합니다.
```
id: 사용자 고유 식별자 (PK)

email: 사용자 이메일 (UNIQUE)

name: 사용자 이름

password: 암호화된 비밀번호

role: 사용자 권한 ('USER' 또는 'ADMIN')

created_at: 계정 생성 시간
```
### games 테이블

게임 정보를 저장합니다.

```
id: 게임 고유 식별자 (PK)

title: 게임 제목

genre: 게임 장르

release_date: 게임 출시일

created_at: 데이터 생성 시간
```

### reviews 테이블

사용자가 게임에 대해 작성한 리뷰를 저장합니다.
```
id: 리뷰 고유 식별자 (PK)

user_id: 리뷰 작성자 ID (FK)

game_id: 리뷰 대상 게임 ID (FK)

rating: 게임 평점 (1~5)

comment: 리뷰 내용

created_at: 리뷰 작성 시간
```

### review_likes 테이블

리뷰에 대한 '좋아요' 정보를 저장합니다.
```
id: 좋아요 고유 식별자 (PK)

user_id: 좋아요를 누른 사용자 ID (FK)

review_id: 좋아요 대상 리뷰 ID (FK)

created_at: 좋아요 생성 시간

한 사용자는 특정 리뷰에 한 번만 좋아요 가능 (UNIQUE 제약조건)
```

# 🎮 게임 리뷰 플랫폼 REST API 설계

## 📌 1. 사용자 (Users) API
| HTTP 메서드 | 엔드포인트 | 설명 | 인증 필요 |
|------------|-----------|-----|----------|
| `POST` | `/api/users/register` | 회원가입 | ❌ |
| `POST` | `/api/users/login` | 로그인 | ❌ |
| `GET` | `/api/users/me` | 내 정보 조회 | ✅ |
| `PUT` | `/api/users/me` | 내 정보 수정 | ✅ |
| `DELETE` | `/api/users/me` | 회원 탈퇴 | ✅ |
| `GET` | `/api/users/{userId}` | 특정 사용자 프로필 조회 | ❌ |

---

## 📌 2. 게임 (Games) API
| HTTP 메서드 | 엔드포인트 | 설명 | 인증 필요 |
|------------|-----------|------|----------|
| `POST` | `/api/games` | 게임 추가 (관리자만 가능) | ✅ (Admin) |
| `GET` | `/api/games` | 게임 목록 조회 | ❌ |
| `GET` | `/api/games/{gameId}` | 특정 게임 상세 조회 | ❌ |
| `PUT` | `/api/games/{gameId}` | 게임 정보 수정 (관리자만 가능) | ✅ (Admin) |
| `DELETE` | `/api/games/{gameId}` | 게임 삭제 (관리자만 가능) | ✅ (Admin) |

---

## 📌 3. 리뷰 (Reviews) API
| HTTP 메서드 | 엔드포인트 | 설명 | 인증 필요 |
|------------|-----------|------|----------|
| `POST` | `/api/games/{gameId}/reviews` | 특정 게임에 리뷰 작성 | ✅ |
| `GET` | `/api/games/{gameId}/reviews` | 특정 게임의 리뷰 목록 조회 | ❌ |
| `GET` | `/api/reviews/{reviewId}` | 특정 리뷰 상세 조회 | ❌ |
| `PUT` | `/api/reviews/{reviewId}` | 본인이 작성한 리뷰 수정 | ✅ (작성자) |
| `DELETE` | `/api/reviews/{reviewId}` | 본인이 작성한 리뷰 삭제 | ✅ (작성자) |

---

## 📌 4. 리뷰 좋아요 (Review Likes) API
| HTTP 메서드 | 엔드포인트 | 설명 | 인증 필요 |
|------------|-----------|------|----------|
| `POST` | `/api/reviews/{reviewId}/like` | 리뷰에 좋아요 추가 | ✅ |
| `DELETE` | `/api/reviews/{reviewId}/like` | 리뷰 좋아요 취소 | ✅ |

---

## 📌 5. 인증 (Auth) API
| HTTP 메서드 | 엔드포인트 | 설명 | 인증 필요 |
|------------|-----------|-----|----------|
| `POST` | `/api/auth/login` | 로그인 | ❌ |
| `POST` | `/api/auth/logout` | 로그아웃 | ✅ |

---

## 📌 REST API 설계 원칙
### 1️⃣ 자원 중심 (Resource-Oriented)
- URL에 동사를 사용하지 않고, **명사**를 사용하여 자원 표현.
- `/games/{gameId}/reviews` 같은 계층적 구조 사용.

### 2️⃣ HTTP 메서드 활용
- `GET` → 데이터 조회
- `POST` → 데이터 생성
- `PUT` → 데이터 수정
- `DELETE` → 데이터 삭제

### 3️⃣ 권한 부여
- `관리자(Admin)` → 게임 추가/수정/삭제 가능.
- `일반 사용자(User)` → 리뷰 작성, 수정, 삭제 가능.

