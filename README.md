<p align="center">
    <img align="top" width="30%" src="/images/BNGdrasil.png" alt="BNGdrasil"/>
</p>

<br>

# BNGdrasil (BNbong + ygGdrasil)

bnbong 클라우드 허브 프로젝트

- `bnbong.xyz` : bnbong의 개인 포트폴리오 사이트 -> [Bantheon](https://github.com/BNGdrasil/Bantheon)
- `playground.bnbong.xyz` : bnbong이 개발한 웹 게임 모음 플랫폼 -> [Blysium](https://github.com/BNGdrasil/Blysium)
- `api.bnbong.xyz` : bnbong이 개발한 backend API 게이트웨이, Bifrost (하위 API 서버 프록시) -> [Bifrost](https://github.com/BNGdrasil/Bifrost)
- `auth.bnbong.xyz` : backend Auth Server, Bidar (사용자 인증 서버) -> [Bidar](https://github.com/BNGdrasil/Bidar)

## Architecture

(MSA 구조도 추가 예정)

## Stack

> [!TIP]
 > 세부 Stack은 수정사항 있을 시, 반영

- **Cloud**: OCI + Terraform IaC
- **Deployment**: Docker Compose
- **API Gateway (Bifrost)**: FastAPI + Python 3.12+
- **Auth Server**: FastAPI + Python 3.12+ + JWT + PostgreSQL
- **Client (Portfolio + Admin)**: React + TypeScript + Vite + Tailwind CSS
- **Playground**: React + TypeScript + Vite + Framer Motion
- **Database**: PostgreSQL + Redis
- **Reverse Proxy**: Nginx
- **SSL**: Cloudflare (자동 SSL 인증서 관리)
- **CI/CD**: GitHub Actions (예정)
- **Monitoring**: Prometheus + Grafana (예정)

## 주요 기능

### Bifrost (API Gateway)

- ✅ 동적 서비스 라우팅
- ✅ 요청/응답 로깅
- ✅ 레이트 리미팅
- ✅ 헬스 체크
- ✅ 관리자 API (서비스 등록/제거)

### Bidar (Auth Server)

- ✅ JWT 기반 인증
- ✅ 사용자 등록/로그인
- ✅ 토큰 갱신
- ✅ API 키 관리
- ✅ 슈퍼유저 권한 관리

### Bantheon (Portfolio + Admin client)

- ✅ 포트폴리오 사이트
- ✅ 관리자 패널 (예정)
- ✅ 반응형 디자인
- ✅ SEO 최적화

### Blysium (Playground)

- ✅ 게임 컬렉션 플랫폼
- ✅ 사용자 인증
- ✅ 게임 점수 관리 (예정)
- ✅ 실시간 게임 (예정)

## 프로덕션 배포

자세한 배포 가이드는 [DEPLOYMENT.md](./DEPLOYMENT.md)를 참조하세요.

```bash
# 1. 인프라 배포
cd baedalus
terraform init
terraform apply

# 2. 애플리케이션 배포 (자동화된 스크립트 사용)
chmod +x baedalus/scripts/deploy.sh
./baedalus/scripts/deploy.sh [서버_IP] ubuntu

# 또는 수동 배포
rsync -avz --exclude='.git' --exclude='node_modules' . ubuntu@[서버_IP]:/opt/bnbong/
ssh ubuntu@[서버_IP] 'cd /opt/bnbong && docker-compose up -d --build' # 예시
```

## API 엔드포인트

> [!TIP]
 > API endpoint에 수정사항 있을 시, 반영

### Bifrost (api.bnbong.xyz)

- `GET /health` - 서비스 상태 확인
- `GET /services` - 등록된 서비스 목록
- `GET /services/{service_name}/health` - 서비스 헬스 체크
- `/{service_name}/{path}` - 서비스 프록시

### Bidar (auth.bnbong.xyz)

#### 인증 (Authentication)

- `POST /auth/register` - 회원가입
- `POST /auth/login` - 로그인
- `POST /auth/refresh` - 토큰 갱신
- `POST /auth/logout` - 로그아웃

#### 사용자 관리 (Users)

- `GET /users/me` - 현재 사용자 정보 조회
- `PUT /users/me` - 사용자 정보 수정
- `DELETE /users/me` - 계정 삭제

#### 헬스 체크 (Health Check)

- `GET /health` - 서비스 상태 확인

(추가 & 수정 예정)

## 개발 가이드

- [UV 패키지 매니저 사용법](./UV_GUIDE.md) - FastAPI 서비스 개발 환경 설정
- [배포 가이드](./DEPLOYMENT.md) - 프로덕션 배포 방법

## 개발 로드맵

### Phase 1: 기본 인프라

- [x] 프로젝트 구조 설계
- [x] Docker Compose 설정
- [x] Terraform 인프라 코드
- [x] API Gateway 구현
- [x] Auth Server 구현

### Phase 2: 프론트엔드 개발

- [ ] React 클라이언트 구현
- [ ] 포트폴리오 사이트
- [ ] 관리자 패널
- [ ] Playground 플랫폼

### Phase 3: 게임 통합

- [ ] Pygame 웹 변환 ([선새임 몰래 춤추기](https://github.com/bnbong/rickTcal_Game) 우선)
- [ ] 게임 실행 엔진
- [ ] 점수 시스템
- [ ] 리더보드

### Phase 4: 고급 기능

- [ ] 모니터링 시스템
- [ ] CI/CD 파이프라인
- [ ] 백업 시스템
- [ ] 성능 최적화
