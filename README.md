<p align="center">
    <img align="top" width="30%" src="/images/BNGdrasil.png" alt="BNGdrasil"/>
</p>

<div align="center">

# BNGdrasil (BNbong + Yggdrasil)

**A comprehensive cloud infrastructure project**

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

*A personal cloud nation infrastructure project by [bnbong](https://github.com/bnbong) - A comprehensive cloud infrastructure project*

</div>

---

## Overview

BNGdrasil is a comprehensive cloud ecosystem that integrates personal portfolio, game platform, API services, and authentication systems. All infrastructure is managed through **Infrastructure as Code (IaC)** principles, designed to operate seamlessly across global cloud environments (Oracle Cloud, AWS, Azure) and future home lab (OpenStack-based) environments.

## Project Naming Convention

Each sub-project in BNGdrasil combines **bnbong's name + Norse mythology/concepts**:

- **BNGdrasil (Main Project)**: The overarching project name (bnbong + Yggdrasil, the World Tree)

### Sub-projects

1. **🏗️ [Baedalus (IaC)](https://github.com/BNGdrasil/Baedalus)**
   - Terraform-based infrastructure code project
   - Declarative management of CSP environments (Oracle Cloud, etc.) and home lab (OpenStack) infrastructure
   - (bnbong + Daedalus, architect and craftsman of Greek mythology)

2. **🌐 [Bsgard (Custom VPC)](https://github.com/BNGdrasil/Bsgard)**
   - Custom network project wrapping OpenStack Neutron functionality
   - Provides VPC-like features for CSP environments
   - Manages VM resource placement in home lab environment with Public/Private Subnet architecture
   - (bnbong + Asgard, gods' location in Nordic mythology)

3. **🌉 [Bifrost (API Gateway)](https://github.com/BNGdrasil/Bifrost)**
   - FastAPI-based API Gateway service
   - API service routing, logging, authentication/authorization (JWT, API Key)
   - Admin UI integration for service registration and management
   - (bnbong + Bifrost, bridge between the gods and humans in Norse mythology)

4. **🔐 [Bidar (Auth Server)](https://github.com/BNGdrasil/Bidar)**
   - FastAPI-based authentication server
   - JWT-based authentication/authorization, Superuser management
   - PostgreSQL/Redis integration for user and session management
   - (bnbong + Vidar, god of vengeance and guardian of silence)

5. **🎨 [Bantheon (Web Client + Portfolio)](https://github.com/BNGdrasil/Bantheon)**
   - React-based static frontend
   - Portfolio pages and Admin Client functionality
   - Integration with API Gateway and Auth Server for administrative operations
   - (bnbong + Pantheon, temple of the gods from ancient Greek)

6. **🎮 [Blysium (Game Platform)](https://github.com/BNGdrasil/Blysium)**
   - React-based static frontend
   - Platform for browser-executable games collection
   - Minimal user management, focused on game execution and selection
   - (bnbong + Elysium, paradise of the blessed of Greek religious cult)

## Architecture

![BNGdrasil Infrastructure](./images/bngdrasil-infra.png)

### Network and VM Layout

```mermaid
graph TB
    subgraph "Public Subnet"
        VM1[Nginx Proxy Manager<br/>Cloudflare Integration]
        VM2[Bifrost API Gateway<br/>Bidar Auth Server]
        VM3[Bantheon Portfolio/Admin<br/>Blysium Game Platform]
    end
    
    subgraph "Private Subnet"
        VM4[PostgreSQL<br/>Redis]
        VM5[Prometheus<br/>Grafana<br/>Loki]
        VM6[Backend API Services<br/>qshing-server, hello, etc.]
    end
    
    Cloudflare[Cloudflare DNS & Proxy + WAF] --> VM1
    VM1 --> VM2
    VM1 --> VM3
    VM2 --> VM4
    VM2 --> VM6
    VM3 --> VM2
    VM5 --> VM4
    VM5 --> VM6
```

## Technology Stack

- **Backend (API Gateway, Auth Server)**: Python 3.12+ (FastAPI)
- **Frontend (Portfolio, Game Platform, Admin UI)**: React (Vite-based)
- **Infrastructure as Code**: Terraform
- **Containerization**: Docker, Docker Compose (→ Kubernetes scalable)
- **Database & Cache**: PostgreSQL, Redis
- **Monitoring**: Prometheus, Grafana, Loki
- **DNS & Proxy**: Cloudflare + Nginx Proxy Manager
- **Cloud / Virtualization**: Oracle Cloud → OpenStack (Home Lab)

## Security & Access Control

- **Public Services Protection**: Cloudflare DNS & Proxy + WAF
- **Private Subnet Isolation**: External access restricted (VPN/Bastion Host only)
- **Service Deployment**: Docker Compose-based VM deployment (Kubernetes expansion planned)

## Development Roadmap

### Phase 1: Core Infrastructure ✅
- [x] Project structure design
- [x] Docker Compose configuration
- [x] Terraform infrastructure code
- [x] API Gateway implementation
- [x] Auth Server implementation

### Phase 2: Frontend Development
- [ ] React client implementation
- [ ] Portfolio website
- [ ] Admin panel
- [ ] Playground platform

### Phase 3: Game Integration
- [ ] Pygame web conversion (Priority: [선새임 몰래 춤추기](https://github.com/bnbong/rickTcal_Game))
- [ ] Game execution engine
- [ ] Score system
- [ ] Leaderboard

### Phase 4: Advanced Features
- [ ] Monitoring system
- [ ] CI/CD pipeline
- [ ] Backup system
- [ ] Performance optimization

## Future Expansion Plans

- **API Gateway (Bifrost)**: Service registration automation, API Key issuance, Rate Limiting
- **Auth Server (Bidar)**: OIDC integration, Role-based access control
- **Bantheon**: Project showcase additions, admin dashboard expansion
- **Blysium**: Simple ranking/score system implementation
- **Baedalus**: Multi-CSP support (easy migration to AWS, Azure)
- **Bsgard**: OpenStack Neutron-based API wrapper completion for CSP-like VPC functionality

## Development Guide

- [UV Package Manager Guide](./UV_GUIDE.md) - FastAPI service development environment setup
- [Deployment Guide](./DEPLOYMENT.md) - Production deployment methods

---

*BNGdrasil - Building a personal cloud nation, one service at a time.*
