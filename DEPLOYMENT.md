# BNGdrasil 배포 가이드

이 문서는 BNGdrasil 프로젝트를 OCI 클라우드에 배포하는 방법을 설명합니다.

## 사전 준비사항

### 1. OCI 계정 설정

- Oracle Cloud Infrastructure 계정 생성
- API 키 생성 및 다운로드
- Compartment 생성
- 사용자 OCID 및 Tenancy OCID 확인

### 2. 도메인 설정

- `bnbong.xyz` 도메인 소유
- DNS 설정 준비 (A 레코드, CNAME 레코드)

### 3. 로컬 개발 환경

- Terraform 설치 (v1.0+)
- Docker 설치
- Git 설치

## 배포 단계

### 1. Terraform 설정

```bash
cd baedalus

# terraform.tfvars 파일 생성
cp terraform.tfvars.example terraform.tfvars

# terraform.tfvars 파일 편집
# OCI 설정 정보 입력
```

`terraform.tfvars` 파일에 다음 정보를 입력하세요:

```hcl
tenancy_ocid     = "ocid1.tenancy.oc1..your-tenancy-ocid"
user_ocid        = "ocid1.user.oc1..your-user-ocid"
fingerprint      = "your-api-key-fingerprint"
private_key_path = "/path/to/your/private_key.pem"
region           = "us-ashburn-1"
compartment_id   = "ocid1.compartment.oc1..your-compartment-ocid"
ssh_public_key   = "your-ssh-public-key"
domain_name      = "bnbong.xyz"
```

### 2. 인프라 배포

```bash
# Terraform 초기화
terraform init

# 배포 계획 확인
terraform plan

# 인프라 배포
terraform apply
```

### 3. DNS 설정

배포 완료 후 출력된 공용 IP 주소를 사용하여 DNS 레코드를 설정하세요:

```
A     bnbong.xyz          -> [공용 IP 주소]
A     api.bnbong.xyz      -> [공용 IP 주소]
A     playground.bnbong.xyz -> [공용 IP 주소]
A     auth.bnbong.xyz      -> [공용 IP 주소]
CNAME www.bnbong.xyz      -> bnbong.xyz
```

### 4. Cloudflare 설정

Cloudflare에서 도메인 설정을 완료하세요:

1. **DNS 레코드 설정**

   ```
   A     bnbong.xyz          -> [공용 IP 주소]
   A     api.bnbong.xyz      -> [공용 IP 주소]
   A     playground.bnbong.xyz -> [공용 IP 주소]
   CNAME www.bnbong.xyz      -> bnbong.xyz
   ```

2. **SSL/TLS 설정**

   - SSL/TLS 암호화 모드: "Full (strict)" 또는 "Full" 선택
   - Edge Certificates: "Always Use HTTPS" 활성화
   - HSTS: 필요시 활성화

3. **프록시 상태**

   - 모든 A 레코드에 대해 프록시 활성화 (주황색 구름 아이콘)
   - CNAME 레코드는 프록시 비활성화 (회색 구름 아이콘)

### 5. 애플리케이션 배포

#### 방법 1: 자동화된 배포 스크립트 사용 (권장)

```bash
# 프로젝트 루트에서 배포 스크립트 실행
chmod +x baedalus/scripts/deploy.sh
./baedalus/scripts/deploy.sh [공용 IP 주소] ubuntu
```

#### 방법 2: 수동 배포

```bash
# 프로젝트 루트로 이동
cd /path/to/bngdrasil

# 서버에 코드 복사 (rsync 사용 권장)
rsync -avz --exclude='.git' --exclude='node_modules' --exclude='__pycache__' \
    --exclude='.env' --exclude='.DS_Store' \
    . ubuntu@[공용 IP 주소]:/opt/bnbong/

# 서버에 SSH 접속
ssh ubuntu@[공용 IP 주소]

# 애플리케이션 빌드 및 실행
cd /opt/bnbong
# 빌더 스크립트 (docker-compose) 혹은 오케스트레이션 (k8s) 사용.
# 추가 예정
```

## 서비스 구성

### 1. API Gateway (Bifrost)

- **URL**: `https://api.bnbong.xyz`
- **포트**: 8000
- **기능**: API 라우팅, 로깅, 모니터링

### 2. Auth Server (Bidar)

- **URL**: `https://auth.bnbong.xyz`
- **포트**: 8000
- **기능**: JWT 인증, 사용자 관리, API 키 관리

### 3. Client (Bantheon)

- **URL**: `https://bnbong.xyz`
- **포트**: 3000
- **기능**: 포트폴리오 사이트, 관리자 패널

### 4. Playground (Blysium)

- **URL**: `https://playground.bnbong.xyz`
- **포트**: 3001
- **기능**: 게임 컬렉션 플랫폼

## 백업 및 복구 (docker-compose 사용 시)

### 1. 데이터베이스 백업

```bash
# PostgreSQL 백업
docker-compose exec postgres pg_dump -U bnbong bnbong > backup.sql

# Redis 백업
docker-compose exec redis redis-cli BGSAVE
```

### 2. 설정 파일 백업

```bash
# 중요 설정 파일 백업
tar -czf config-backup.tar.gz .env docker-compose.yml nginx/
```

## 보안 고려사항

### 1. 환경 변수 보안

- `.env` 파일의 민감한 정보 변경
- JWT 시크릿 키 변경
- 데이터베이스 비밀번호 변경

### 2. 방화벽 설정

- 필요한 포트만 열기 (22, 80, 443)
- SSH 키 기반 인증 사용
- 불필요한 서비스 비활성화

### 3. 정기 업데이트

- 시스템 패키지 정기 업데이트
- Docker 이미지 정기 업데이트
- 보안 패치 적용

## 문제 해결

### 1. Cloudflare 설정 문제

```bash
# DNS 확인
nslookup bnbong.xyz
dig bnbong.xyz

# Cloudflare 프록시 상태 확인
# Cloudflare 대시보드에서 프록시 상태 확인
```

### 2. 도메인 연결 문제

```bash
# DNS 확인
nslookup bnbong.xyz
dig bnbong.xyz

# 포트 확인
netstat -tlnp | grep :80
netstat -tlnp | grep :443
```

## 확장 및 마이그레이션

### 1. 다른 클라우드로 마이그레이션

- Terraform 코드를 다른 프로바이더용으로 수정
- Docker Compose 설정은 동일하게 사용 가능
- 데이터베이스 마이그레이션 계획 수립

### 2. 스케일링

- 로드 밸런서 추가 (CSP 과금 플랜 필히 확인)
- 데이터베이스 클러스터링 (여유가 된다면)
- 캐시 레이어 확장
