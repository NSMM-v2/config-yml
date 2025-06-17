## ⚙️ 서비스별 상세 설정

### **🔐 auth-service.yml**

- **포트**: 8081
- **데이터베이스**: MySQL (esg_auth)
- **JWT 설정**: 15분 액세스토큰, 7일 리프레시토큰
- **보안**: 비밀번호 정책, 세션 관리, 계층적 권한
- **특별기능**: 본사/협력사 계정 생성, 트리 구조 권한

### **🌐 discovery-service.yml**

- **포트**: 8761 (Eureka Server)
- **역할**: 마이크로서비스 등록/발견
- **개발환경**: 빠른 하트비트 (5초), 자체보호모드 비활성화
- **모니터링**: 서비스 상태 추적, 자동 장애 감지

### **🚪 gateway-service.yml**

- **포트**: 8080 (클라이언트 진입점)
- **라우팅**: Auth(8081), CSDDD(8083) 등 서비스 라우팅
- **인증**: JWT 쿠키 → 헤더 변환
- **보안**: Rate Limiting, Circuit Breaker, CORS
- **모니터링**: Gateway 상태, 라우팅 정보

### **📊 csddd-service.yml**

- **포트**: 8083
- **역할**: CSDDD 공시 데이터 관리
- **데이터검증**: Scope 1,2,3 배출량 검증
- **배치처리**: 대용량 데이터 청크 처리
- **외부연동**: 탄소배출계수 API, 정부공시시스템

## 🔧 업데이트된 환경변수 섹션

기존 README.md의 "🔧 환경 변수 설정" 부분을 다음과 같이 교체해주세요:

````markdown
## 🔧 환경변수 설정

### **🌐 공통 환경변수 (모든 서비스)**

```bash
# 데이터베이스 설정
DB_URL=jdbc:mysql://localhost:3306/esg_auth
DB_USERNAME=esg_user
DB_PASSWORD=esg_password

# JPA 설정
JPA_DDL_AUTO=update              # create | update | validate | none
JPA_SHOW_SQL=true                # 개발: true, 운영: false

# JWT 인증 (모든 인증 서비스에서 동일해야 함)
JWT_SECRET=mySecretKeyForJWTTokenGenerationAndValidationPurposeOnly123456789

# Eureka 서버
EUREKA_SERVER=http://localhost:8761/eureka/
```
````

### **🔐 Auth Service (8081) 환경변수**

**필수:**

- `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` (공통)
- `JWT_SECRET` (공통)

**선택:**

```bash
JWT_COOKIE_SECURE=false          # 개발: false, 운영: true
spring.application.version=1.0.0
```

### **🚪 Gateway Service (8080) 환경변수**

**필수:**

- `JWT_SECRET` (Auth Service와 동일해야 함)

**선택:**

```bash
CORS_ALLOWED_ORIGINS=https://your-frontend.com,https://admin.yoursite.com
spring.application.version=1.0.0
```

### **🌍 Discovery Service (8761) 환경변수**

**모두 선택사항:**

```bash
DISCORD_WEBHOOK=https://discord.com/api/webhooks/your-webhook-url  # 서비스 등록 알림용
```

> **특징**: 거의 모든 설정이 기본값으로 동작하므로 환경변수가 거의 불필요

### **📊 CSDDD Service (8083) 환경변수**

**필수:**

- `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` (공통)

**선택:**

```bash
# JPA 설정 (대용량 데이터 고려)
JPA_SHOW_SQL=false               # 대용량 처리시 false 권장
LOG_SQL=false                    # SQL 로깅

# 외부 API 연동
CARBON_API_ENABLED=false         # 탄소 배출 계수 API
CARBON_API_URL=https://carbon-api.example.com
CARBON_API_KEY=your-api-key

GOV_API_ENABLED=false            # 정부 공시 시스템
GOV_API_URL=https://gov-api.example.com

spring.application.version=1.0.0
```

## 🌟 환경별 설정 예시

### **개발환경 (최소 설정)**

```bash
# 이 4개만 설정하면 모든 서비스 실행 가능
DB_URL=jdbc:mysql://localhost:3306/esg_auth
DB_USERNAME=esg_user
DB_PASSWORD=esg_password
JWT_SECRET=dev-jwt-secret-key-for-development

# 추가 개발 편의 설정
JWT_COOKIE_SECURE=false
JPA_SHOW_SQL=true
CORS_ALLOWED_ORIGINS=http://localhost:3000
```

### **운영환경 (보안 강화)**

```bash
# 보안 강화 설정
DB_URL=jdbc:mysql://prod-server:3306/esg_production
DB_USERNAME=prod_user
DB_PASSWORD=super-secure-production-password
JWT_SECRET=production-super-secure-jwt-key-256-bits-minimum
JWT_COOKIE_SECURE=true
JPA_DDL_AUTO=validate
JPA_SHOW_SQL=false
CORS_ALLOWED_ORIGINS=https://esg.yourcompany.com

# 외부 API 활성화
CARBON_API_ENABLED=true
CARBON_API_URL=https://api.carbonfactor.org/v1
CARBON_API_KEY=cf_live_production_key
```

## 🚀 빠른 시작 가이드

### **1. 최소 환경변수 설정**

```bash
export DB_URL=jdbc:mysql://localhost:3306/esg_auth
export DB_USERNAME=esg_user
export DB_PASSWORD=esg_password
export JWT_SECRET=your-secret-key
```

### **2. 환경변수 확인**

```bash
# Actuator로 현재 설정 확인
curl http://localhost:8081/actuator/env
curl http://localhost:8080/actuator/env
```
