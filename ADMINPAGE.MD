## 어드민 페이지 개발 요구사항 정리

메뉴 촬영 서비스의 가이드 관리를 위한 웹 기반 어드민 페이지 개발

### 핵심 기능 요구사항

### 1. 가이드 업로드 기능

- **방식**: AWS S3 Presigned URL을 통한 직접 업로드
- **프로세스**:
  1. 클라이언트에서 업로드 요청 시 서버로부터 Presigned URL 받기
     1. **클라이언트 처리 순서:**
        1. 이 API를 호출하여 `presignedUrl`과 `s3Key`를 받습니다.
        2. 받은 `presignedUrl`을 목적지로, 업로드할 XML 파일의 원본 데이터를 body에 담아 **HTTP PUT** 요청을 보냅니다.
           - **주의:** `Content-Type` 헤더에 반드시 `application/xml`을 포함해야 합니다.
        3. S3 업로드가 성공(HTTP 200 OK)하면, `/api/guides/register` API를 호출하여 업로드 완료 사실을 서버에 알립니다.
           - 이때 응답으로 받았던 `s3Key`와 파일의 원본 이름 등 필요한 메타데이터를 함께 전송합니다.
- **지원 파일**: XML 파일만

### 2. 가이드 삭제 기능

- **방식**: 서버 API 호출을 통한 삭제 요청
- **프로세스**:
  1. 클라이언트에서 삭제 요청 전송
  2. 서버에서 DB와 S3 파일 모두 삭제 처리

### 3. 가이드 조회 기능

- **방식**: 서버 API를 통한 가이드 목록 조회
- **표시 형태**: 그리드 형태로 가이드 파일 정보 표시
- **페이지네이션**: 페이지 단위로 목록 조회 지원

### 기술적 요구사항

### 인프라 구조

- **스토리지**: AWS S3
- **인증**: Presigned URL 방식
- **백엔드**: Spring Boot 서버

### 필수 API 엔드포인트

1. `GET /api/guides` - 가이드 목록 조회 (페이지네이션)
2. `POST /api/guides/presigned-url` - Presigned URL 생성
3. `POST /api/guides/register` - 서버에 업로드 완료 사실 알림
4. `DELETE /api/guides/{guideId}` - 가이드 삭제
5. `GET /api/guides/{guideId}` - 가이드 상세 조회

### UI/UX 요구사항

- **인증**: 어드민 권한이 필요한 API 접근
- **직관적 인터페이스**: 드래그 앤 드롭 업로드 지원
- **진행 상태 표시**: 업로드/삭제 진행률 표시
- **파일 관리**: XML 가이드 파일 목록 및 관리 기능

# Chalpu Web - 가이드 관리 어드민 페이지

메뉴 촬영 서비스의 가이드 관리를 위한 웹 기반 어드민 페이지입니다.

## 기능

### 핵심 기능

- **가이드 업로드**: AWS S3 Presigned URL을 통한 직접 업로드
- **가이드 삭제**: 서버 API를 통한 가이드 삭제 (DB + S3 파일)
- **가이드 조회**: 서버에서 관리되는 가이드 목록 조회
- **드래그 앤 드롭**: 직관적인 파일 업로드 인터페이스
- **진행률 표시**: 업로드/삭제 작업 진행률 실시간 표시
- **페이지네이션**: 대량의 가이드 파일 효율적 관리

### 지원 파일 형식

- XML 파일만 지원

### 기술 스택

- **Frontend**: Next.js 14, React, TypeScript, Tailwind CSS
- **Backend**: Spring Boot Server
- **Storage**: AWS S3
- **API**: RESTful API

## 설치 및 실행

```bash
# 의존성 설치
npm install

# 개발 서버 실행
npm run dev

# 프로덕션 빌드
npm run build

# 프로덕션 서버 실행
npm start
```

## 어드민 페이지 사용법

### 1. 접근 방법

어드민 페이지는 `/admin` 경로로 접근할 수 있습니다.

```
http://localhost:3001/admin
```

### 2. 인증 토큰 설정

- 어드민 페이지에는 **토큰 설정 UI**가 제공됩니다.
- 페이지 상단의 "🔐 인증 토큰 설정" 섹션에서 Bearer 토큰을 입력할 수 있습니다.
- 토큰은 브라우저의 로컬 스토리지에 안전하게 저장됩니다.
- 토큰 상태를 실시간으로 확인할 수 있습니다:
  - ✅ 유효한 토큰: API 요청 가능
  - ❌ 유효하지 않은 토큰: API 요청 실패
  - ⚠️ 토큰 없음: API 기능 비활성화

### 3. 토큰 사용 방법

1. **토큰 입력**: 페이지 상단의 입력 필드에 Bearer 토큰을 입력합니다.
2. **토큰 저장**: "토큰 저장" 버튼을 클릭하여 로컬 스토리지에 저장합니다.
3. **토큰 테스트**: "토큰 테스트" 버튼으로 토큰의 유효성을 검증합니다.
4. **토큰 삭제**: "토큰 삭제" 버튼으로 저장된 토큰을 제거합니다.
5. **자동 인증**: 저장된 토큰은 모든 API 요청에 자동으로 포함됩니다.

### 4. 파일 업로드

1. **드래그 앤 드롭**: XML 파일을 업로드 영역으로 드래그하여 놓기
2. **클릭 업로드**: 업로드 영역을 클릭하여 파일 선택
3. **다중 파일**: 여러 XML 파일을 동시에 업로드 가능
4. **진행률 확인**: 업로드 진행률을 실시간으로 확인

### 5. 가이드 관리

- **목록 조회**: 업로드된 가이드 파일 목록 확인
- **삭제**: 각 가이드의 삭제 버튼으로 개별 삭제 가능
- **상세정보**: 파일명, ID, 업로드 날짜 확인
- **페이지네이션**: 페이지 단위로 목록 탐색

## API 엔드포인트

### 가이드 목록 조회

```
GET /api/guides?page=0&size=20&sort=createdAt,desc
```

**응답 예시:**

```json
{
  "code": 0,
  "message": "string",
  "result": {
    "content": [
      {
        "id": 1,
        "s3Key": "guides/c1f8f3a3-3b1b-4b8b-8b5b-4b8b8b5b4b8b.xml",
        "fileName": "guide_v1.xml",
        "createdAt": "2025-07-04T12:49:00.518Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 100,
    "totalPages": 5,
    "hasNext": true,
    "hasPrevious": false
  }
}
```

### Presigned URL 생성

```
POST /api/guides/presigned-url
Content-Type: application/json

{
  "fileName": "guide_v1.xml"
}
```

**응답 예시:**

```json
{
  "code": 0,
  "message": "string",
  "result": {
    "presignedUrl": "https://s3.amazonaws.com/...",
    "s3Key": "guides/c1f8f3a3-3b1b-4b8b-8b5b-4b8b8b5b4b8b.xml"
  }
}
```

### 업로드 완료 알림

```
POST /api/guides/register
Content-Type: application/json

{
  "s3Key": "guides/c1f8f3a3-3b1b-4b8b-8b5b-4b8b8b5b4b8b.xml",
  "fileName": "guide_v1.xml"
}
```

**응답 예시:**

```json
{
  "code": 0,
  "message": "string",
  "result": {
    "id": 1,
    "s3Key": "guides/c1f8f3a3-3b1b-4b8b-8b5b-4b8b8b5b4b8b.xml",
    "fileName": "guide_v1.xml",
    "createdAt": "2025-07-04T12:49:00.518Z"
  }
}
```

### 가이드 상세 조회

```
GET /api/guides/{guideId}
```

### 가이드 삭제

```
DELETE /api/guides/{guideId}
```

**응답 예시:**

```json
{
  "code": 0,
  "message": "string",
  "result": {}
}
```

## 환경 변수

```env
# API 기본 URL
NEXT_PUBLIC_API_BASE_URL=http://localhost:8080

# 개발 서버 포트
PORT=3001
```

## 개발 시 주의사항

1. **Spring 서버 연동**: 백엔드 Spring Boot 서버가 실행 중이어야 합니다.
2. **어드민 권한**: 모든 가이드 API는 어드민 권한이 필요합니다.
3. **토큰 관리**: 토큰 없이는 API 기능이 비활성화됩니다.
4. **토큰 보안**: 토큰은 로컬 스토리지에 저장되므로 보안에 주의해야 합니다.
5. **XML 파일만 지원**: SVG 파일은 더 이상 지원하지 않습니다.
6. **페이지네이션**: 대량의 데이터 처리를 위해 페이지네이션이 적용되어 있습니다.

## 폴더 구조

```
src/
├── app/
│   ├── admin/                 # 어드민 페이지
│   │   ├── page.tsx          # 어드민 메인 페이지
│   │   └── layout.tsx        # 어드민 레이아웃
│   └── services/             # 서비스 로직
│       ├── apis/             # API 클라이언트
│       │   ├── guide.ts      # Guide API
│       │   └── base.ts       # 기본 API 클라이언트
│       └── types/            # TypeScript 타입
│           └── guide.ts      # Guide 타입
└── components/               # UI 컴포넌트
    └── landing/
        └── ui/               # 재사용 가능한 UI 컴포넌트
```

## 주요 변경사항 (v2.0)

### API 변경

- **엔드포인트**: `/api/photos` → `/api/guides`
- **파일 타입**: XML 파일만 지원 (SVG 제거)
- **페이지네이션**: Spring Boot 표준 페이지네이션 적용
- **응답 구조**: 공통 응답 형태 (`code`, `message`, `result`) 적용

### 기능 개선

- **어드민 권한**: 모든 API에 어드민 권한 검증 추가
- **토큰 관리**: 스웨거 스타일의 토큰 설정 UI 제공
- **인증 시스템**: Bearer 토큰 기반 인증 시스템 구축
- **파일 관리**: DB와 S3 파일 동시 삭제 지원
- **UI/UX**: 가이드 관리에 특화된 인터페이스

### 기술 스택 변경

- **백엔드**: Next.js API Routes → Spring Boot Server
- **타입 안정성**: Guide 전용 타입 시스템 구축
- **API 클라이언트**: 새로운 API 명세서에 맞춘 클라이언트 재구성
