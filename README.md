# 🔧 ESG 프로젝트 서비스별 환경변수 설정 가이드

> 새로운 개발자가 프로젝트를 클론 후 각 서비스별로 설정해야 하는 환경변수들을 정리했습니다.

## 📁 설정 파일 구조

```
backend/config-yml/
├── auth-service.yml       # 인증 서비스 (8081)
├── gateway-service.yml    # API 게이트웨이 (8080)
├── discovery-service.yml  # 서비스 디스커버리 (8761)
└── csddd-service.yml      # CSDDD 공시 서비스 (8083)
```

---

## 🔐 Auth Service (포트: 8081)

**역할**: 본사/협력사 인증, JWT 토큰 관리, 계층적 권한 시스템

### 필수 환경변수

```bash
# 데이터베이스 연결
DB_URL=jdbc:mysql://localhost:3306/esg_auth
DB_USERNAME=esg_user
DB_PASSWORD=esg_password

# JWT 인증 (256비트 이상 권장)
JWT_SECRET=mySecretKeyForJWTTokenGenerationAndValidationPurposeOnly123456789
```

### 선택적 환경변수

```bash
# JPA 설정
JPA_DDL_AUTO=update                    # create | update | validate | none
JPA_SHOW_SQL=true                      # 개발: true, 운영: false

# JWT 쿠키 보안
JWT_COOKIE_SECURE=false                # 개발: false, 운영: true (HTTPS 필수)

# Eureka 서버
EUREKA_SERVER=http://localhost:8761/eureka/

# 애플리케이션 버전
spring.application.version=1.0.0
```

### 개발환경 권장 설정

```bash
export DB_URL=jdbc:mysql://localhost:3306/esg_auth
export DB_USERNAME=esg_user
export DB_PASSWORD=esg_password
export JWT_SECRET=dev-secret-key-for-jwt-auth-service
export JPA_SHOW_SQL=true
export JWT_COOKIE_SECURE=false
```

---

## 🚪 Gateway Service (포트: 8080)

**역할**: API 게이트웨이, JWT 인증, 라우팅, CORS 처리

### 필수 환경변수

```bash
# JWT 검증 (Auth Service와 동일해야 함)
JWT_SECRET=mySecretKeyForJWTTokenGenerationAndValidationPurposeOnly123456789
```

### 선택적 환경변수

```bash
# Eureka 서버
EUREKA_SERVER=http://localhost:8761/eureka/

# 애플리케이션 버전
spring.application.version=1.0.0
```

### 개발환경 권장 설정

```bash
export JWT_SECRET=dev-secret-key-for-jwt-auth-service  # Auth Service와 동일
export EUREKA_SERVER=http://localhost:8761/eureka/
```

### 운영환경 추가 설정

```bash
# CORS 설정 (운영환경에서 프론트엔드 도메인 명시)
export CORS_ALLOWED_ORIGINS=https://esg.yourcompany.com,https://admin.yourcompany.com
```

---

## 🌐 Discovery Service (포트: 8761)

**역할**: 마이크로서비스 등록/발견, 로드밸런싱, 서비스 헬스체크

### 환경변수 없음

> **특징**: Discovery Service는 기본 설정만으로 동작하므로 **환경변수 설정이 불필요**합니다.

### 선택적 환경변수 (알림용)

```bash
# 서비스 등록 알림 (선택사항)
DISCORD_WEBHOOK=https://discord.com/api/webhooks/your-webhook-url
```

### 실행 방법

```bash
cd backend/discovery-service
./gradlew bootRun
```

---

## 📊 CSDDD Service (포트: 8083)

**역할**: CSDDD 공시 데이터 관리, ESG 리포팅

### 필수 환경변수

```bash
# 데이터베이스 연결 (Auth Service와 동일 DB 사용)
DB_URL=jdbc:mysql://localhost:3306/esg_auth
DB_USERNAME=esg_user
DB_PASSWORD=esg_password
```

### 선택적 환경변수

```bash
# JPA 설정 (대용량 데이터 고려)
JPA_DDL_AUTO=update                    # create | update | validate | none
JPA_SHOW_SQL=false                     # 대용량 처리시 false 권장
LOG_SQL=false                          # SQL 로깅 제어

# Eureka 서버
EUREKA_SERVER=http://localhost:8761/eureka/

# 외부 API 연동 (미래 확장용)
CARBON_API_ENABLED=false               # 탄소 배출 계수 API
CARBON_API_URL=https://carbon-api.example.com
CARBON_API_KEY=your-api-key

GOV_API_ENABLED=false                  # 정부 공시 시스템
GOV_API_URL=https://gov-api.example.com

# 애플리케이션 버전
spring.application.version=1.0.0
```

### 개발환경 권장 설정

```bash
export DB_URL=jdbc:mysql://localhost:3306/esg_auth
export DB_USERNAME=esg_user
export DB_PASSWORD=esg_password
export JPA_SHOW_SQL=false              # 대용량 데이터로 인한 성능 고려
export LOG_SQL=false
```

---

## 🚀 전체 서비스 실행 가이드

### 1. 최소 환경변수 설정 (개발환경)

```bash
# 1. 데이터베이스 설정 (Auth Service + CSDDD Service 공통)
export DB_URL=jdbc:mysql://localhost:3306/esg_auth
export DB_USERNAME=esg_user
export DB_PASSWORD=esg_password

# 2. JWT 설정 (Auth Service + Gateway Service 공통)
export JWT_SECRET=dev-secret-key-for-development

# 3. 개발 편의 설정
export JPA_SHOW_SQL=true
export JWT_COOKIE_SECURE=false
```

### 2. 서비스 실행 순서

```bash
# 1단계: Discovery Service (환경변수 불필요)
cd backend/discovery-service && ./gradlew bootRun &

# 2단계: Auth Service (DB + JWT 설정 필요)
cd backend/auth-service && ./gradlew bootRun &

# 3단계: CSDDD Service (DB 설정 필요)
cd backend/csddd-service && ./gradlew bootRun &

# 4단계: Gateway Service (JWT 설정 필요)
cd backend/gateway-service && ./gradlew bootRun &
```

### 3. 설정 확인 방법

```bash
# 각 서비스 상태 확인
curl http://localhost:8761/eureka/apps    # Discovery: 등록된 서비스 목록
curl http://localhost:8081/actuator/env   # Auth: 환경변수 확인
curl http://localhost:8083/actuator/env   # CSDDD: 환경변수 확인
curl http://localhost:8080/actuator/env   # Gateway: 환경변수 확인
```

---

## ⚠️ 환경별 주의사항

### 개발환경

- `JWT_COOKIE_SECURE=false` (HTTP 허용)
- `JPA_SHOW_SQL=true` (SQL 로그 확인)
- `JPA_DDL_AUTO=update` (스키마 자동 업데이트)

### 운영환경

- `JWT_COOKIE_SECURE=true` (HTTPS 필수)
- `JPA_SHOW_SQL=false` (성능 고려)
- `JPA_DDL_AUTO=validate` (스키마 검증만)
- `JWT_SECRET`은 256비트 이상 안전한 키 사용

---

## 🔍 트러블슈팅

### 자주 발생하는 문제

1. **Auth Service가 시작되지 않는 경우**

   ```bash
   # DB 연결 확인
   mysql -u esg_user -p -h localhost -P 3306 esg_auth
   ```

2. **Gateway에서 401 에러 발생**

   ```bash
   # JWT_SECRET이 Auth Service와 동일한지 확인
   echo $JWT_SECRET
   ```

3. **서비스간 통신 불가**
   ```bash
   # Discovery Service 먼저 실행했는지 확인
   curl http://localhost:8761
   ```

### 환경변수 확인 스크립트

```bash
#!/bin/bash
echo "=== 필수 환경변수 확인 ==="
echo "DB_URL: $DB_URL"
echo "DB_USERNAME: $DB_USERNAME"
echo "DB_PASSWORD: [설정됨: $(test -n "$DB_PASSWORD" && echo "YES" || echo "NO")]"
echo "JWT_SECRET: [설정됨: $(test -n "$JWT_SECRET" && echo "YES" || echo "NO")]"
```

---

## 📝 환경변수 템플릿 파일

### `.env.development` 예시

```bash
# 개발환경 설정
DB_URL=jdbc:mysql://localhost:3306/esg_auth
DB_USERNAME=esg_user
DB_PASSWORD=esg_password
JWT_SECRET=dev-secret-key-for-jwt-auth-service-development
JPA_SHOW_SQL=true
JWT_COOKIE_SECURE=false
JPA_DDL_AUTO=update
EUREKA_SERVER=http://localhost:8761/eureka/
spring.application.version=1.0.0-dev
```

### `.env.production` 예시

```bash
# 운영환경 설정
DB_URL=jdbc:mysql://prod-server:3306/esg_production
DB_USERNAME=prod_user
DB_PASSWORD=super-secure-production-password
JWT_SECRET=production-super-secure-jwt-key-256-bits-minimum
JPA_SHOW_SQL=false
JWT_COOKIE_SECURE=true
JPA_DDL_AUTO=validate
EUREKA_SERVER=http://prod-eureka:8761/eureka/
CORS_ALLOWED_ORIGINS=https://esg.yourcompany.com
spring.application.version=1.0.0
```
