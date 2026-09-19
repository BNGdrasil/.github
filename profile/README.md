<p align="center">
    <img align="top" width="30%" src="https://raw.githubusercontent.com/BNGdrasil/.github/main/images/BNGdrasil.png" alt="BNGdrasil"/>
</p>

<div align="center">

# BNGdrasil (BNbong + ygGdrasil)

**개인 클라우드 인프라 프로젝트**

[![Python](https://img.shields.io/badge/Python-3.12+-3776ab?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=white)](https://react.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org)
[![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white)](https://vitejs.dev)
[![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat-square&logo=terraform&logoColor=white)](https://www.terraform.io)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white)](https://www.postgresql.org)
[![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)](https://redis.io)
[![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat-square&logo=nginx&logoColor=white)](https://nginx.org)

*[bnbong](https://github.com/bnbong)의 개인 클라우드 인프라 프로젝트입니다*

</div>

---

## 소개

BNGdrasil은 개인 클라우드 인프라 프로젝트입니다. Wegis와 Overlock, ambiw 같은 개인 서비스를 한 인프라 위에서 운영합니다. 인증과 게이트웨이, 데이터베이스, 관측 도구를 모든 서비스가 공유합니다. 구성 요소는 독립된 저장소로 나뉘며, 이름은 북유럽 신화와 그리스 신화에서 따왔습니다.

## 프로젝트

| | 이름 | 역할 |
|---|---|---|
| <img width="48" src="https://raw.githubusercontent.com/BNGdrasil/.github/main/images/Bidar.png" alt="Bidar"/> | [Bidar](https://github.com/BNGdrasil/Bidar) | 인증 서버 |
| <img width="48" src="https://raw.githubusercontent.com/BNGdrasil/.github/main/images/Bifrost.png" alt="Bifrost"/> | [Bifrost](https://github.com/BNGdrasil/Bifrost) | API 게이트웨이 |
| <img width="48" src="https://raw.githubusercontent.com/BNGdrasil/.github/main/images/Bantheon.png" alt="Bantheon"/> | [Bantheon](https://github.com/BNGdrasil/Bantheon) | 웹 클라이언트와 VM1 Nginx 설정 |
| <img width="48" src="https://raw.githubusercontent.com/BNGdrasil/.github/main/images/Baedalus.png" alt="Baedalus"/> | [Baedalus](https://github.com/BNGdrasil/Baedalus) | 인프라 코드와 운영 도구 |
| <img width="48" src="https://raw.githubusercontent.com/BNGdrasil/.github/main/images/Bsgard.png" alt="Bsgard"/> | [Bsgard](https://github.com/BNGdrasil/Bsgard) | 자체 VPC 네트워크 구상 (계획 단계) |

## 구조

외부 요청은 Cloudflare를 거쳐 VM1의 Nginx에 도착합니다. Nginx는 정적 사이트를 응답하고 API 요청은 VM2로 넘깁니다. VM2에서는 Bifrost와 Bidar, 개별 서비스가 동작합니다. 데이터는 사설 서브넷 VM3의 PostgreSQL에 저장합니다.

```mermaid
graph LR
    CF[Cloudflare] --> VM1[VM1: Nginx, 정적 사이트]
    VM1 --> VM2[VM2: Bifrost, Bidar, 서비스]
    VM2 --> VM3[(VM3: PostgreSQL)]
```

## 현재 상태

주요 서비스는 운영 중입니다. 2026년 9월부터 전면 보수를 진행하고 있습니다. 인증 강화와 게이트웨이 안정화, 백업과 관측과 CI/CD 정비가 보수의 범위입니다.

## 기술 스택

| 영역 | 사용 기술 |
|---|---|
| 백엔드 | Python 3.12, FastAPI, uv |
| 프런트엔드 | React, TypeScript, Vite |
| 인프라 | Terraform, Docker Compose, Nginx, Cloudflare |
| 데이터 | PostgreSQL, Redis |
| 관측 | Prometheus, Grafana, Loki |
