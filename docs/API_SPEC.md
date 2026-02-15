# A&I Report Server API 명세서

기준 컨트롤러:
- `member/controller/MemberController.kt`
- `report/controller/ReportController.kt`

기준 DTO:
- `member/dtos/LoginDto.kt`
- `report/dtos/ReportDtos.kt`

## 인증 방식
- `Authorization: Bearer <accessToken>`
- 권한 정책은 `common/config/SecurityConfig.kt` 기준
  - 공개: 로그인, Swagger
  - 관리자 전용: 리포트 생성/수정/삭제, 멤버 생성/조회/삭제
  - 인증 사용자: 리포트 조회

## 1. Member API (`/api/member`)

| Method | URI | 설명 | Auth |
|---|---|---|---|
| POST | `/register/member` | 일반 멤버 생성 | ADMIN |
| POST | `/register/admin` | 관리자 생성 | None |
| POST | `/login` | 로그인, Access Token 발급 | None |
| GET | `` | 전체 멤버 조회 | ADMIN |
| DELETE | `/{userId}` | 멤버 삭제 | ADMIN |

### 1-1) POST `/api/member/register/member`

#### Request Body
| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `nickname` | String | Y | 멤버 닉네임 |

**응답 (200)**
```json
{
  "userId": "aandiUser03",
  "nickname": "hood",
  "role": "MEMBER",
  "rawPassword": "Abcd1234Xy"
}
```

### 1-2) POST `/api/member/register/admin`

#### Request Body
| 필드 | 타입 | 필수 |
|---|---|---|
| `userId` | String | Y |
| `nickname` | String | Y |
| `password` | String | Y |

**응답 (200)**
```json
{
  "userId": "admin01",
  "nickname": "관리자",
  "role": "ADMIN",
  "rawPassword": null
}
```

### 1-3) POST `/api/member/login`

#### Request Body
| 필드 | 타입 | 필수 |
|---|---|---|
| `userId` | String | Y |
| `password` | String | Y |

**응답 (200)**
```json
{
  "accessToken": "jwt-token"
}
```

## 2. Report API (`/api/report`)

| Method | URI | 설명 | Auth |
|---|---|---|---|
| POST | `` | 리포트 생성 | ADMIN |
| GET | `` | 진행 중 리포트 요약 조회 | USER |
| GET | `/{id}` | 리포트 상세 조회 | USER |
| PUT | `/{id}` | 리포트 수정 | ADMIN |
| DELETE | `/{id}` | 리포트 삭제 | ADMIN |
| GET | `/allReport` | 전체 리포트 조회 | USER |

### 2-1) POST/PUT 요청 바디 (`ReportRequestDTO`)

| 필드 | 타입 | 필수 | 제약/설명 |
|---|---|---|---|
| `week` | Int | Y | 주차 |
| `seq` | Int | Y | 회차 |
| `title` | String | Y | 제목 |
| `content` | String | Y | 본문 |
| `requirement` | List<SeqString> | Y | 요구사항 목록 |
| `objects` | List<SeqString> | Y | 학습 목표 목록 |
| `exampleIO` | List<ExampleIO> | Y | 입출력 예시 |
| `reportType` | String | Y | `CS` or `BASIC` |
| `startAt` | String | Y | `yyyy-MM-dd'T'HH:mm:ss` (UTC) |
| `endAt` | String | Y | `yyyy-MM-dd'T'HH:mm:ss` (UTC) |
| `level` | String | Y | `VERYHIGH` `HIGH` `MEDIUM` `LOW` |

**요청 예시**
```json
{
  "week": 3,
  "seq": 1,
  "title": "3주차 과제",
  "content": "문제 설명",
  "requirement": [{ "title": "요구사항", "content": "내용" }],
  "objects": [{ "title": "목표", "content": "내용" }],
  "exampleIO": [{ "input": "1 2", "output": "3" }],
  "reportType": "CS",
  "startAt": "2026-02-01T00:00:00",
  "endAt": "2026-02-07T23:59:59",
  "level": "MEDIUM"
}
```

### 2-2) GET `/api/report` 응답 (`ReportSummaryDTO`)

```json
[
  {
    "id": "67ad...",
    "week": 3,
    "seq": 1,
    "title": "3주차 과제",
    "level": "MEDIUM",
    "reportType": "CS",
    "endAt": "2026-02-07T23:59:59Z"
  }
]
```

### 2-3) GET `/api/report/{id}` 응답 (`ReportDetailDTO`)

```json
{
  "id": "67ad...",
  "week": 3,
  "title": "3주차 과제",
  "content": "문제 설명",
  "requirement": [{ "title": "요구사항", "content": "내용" }],
  "objects": [{ "title": "목표", "content": "내용" }],
  "exampleIo": [{ "input": "1 2", "output": "3" }],
  "reportType": "CS",
  "endAt": "2026-02-07T23:59:59Z",
  "level": "MEDIUM"
}
```

## 3. 에러 처리 참고
- 존재하지 않는 리소스: `ResponseStatusException(HttpStatus.NOT_FOUND, ...)`
- 로그인 실패/중복 사용자 등: `IllegalArgumentException` 기반 오류 반환
