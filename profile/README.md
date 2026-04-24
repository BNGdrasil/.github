<p align="center">
    <img align="top" width="30%" src="/images/BNGdrasil.png" alt="BNGdrasil"/>
</p>

<div align="center">

# BNGdrasil (BNbong + ygGdrasil)

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

BNGdrasil is a comprehensive cloud ecosystem that integrates a personal portfolio, API services, authentication systems, and infrastructure automation. All infrastructure is managed through **Infrastructure as Code (IaC)** principles, designed to operate seamlessly across global cloud environments (Oracle Cloud, AWS, Azure) and future home lab (OpenStack-based) environments.

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

## Architecture

![BNGdrasil Infrastructure](../images/bngdrasil-infra.png)

### Multi-Region Architecture

**Chuncheon Region (Main Services):**
```mermaid
graph TB
    subgraph "Chuncheon - Public Subnet"
        VM1[VM1: Client<br/>Nginx Reverse Proxy<br/>Static Files]
        VM2[VM2: Core APIs<br/>Bifrost Gateway + Bidar Auth<br/>2 OCPU, 12GB]
    end
    
    subgraph "Chuncheon - Private Subnet"
        VM3[VM3: Database<br/>PostgreSQL + Redis + MongoDB<br/>1 OCPU, 6GB, 80GB]
    end
    
    Cloudflare[Cloudflare DNS & WAF] --> VM1
    VM1 --> VM2
    VM2 --> VM3
```

**Osaka Region (Monitoring & Backup):**
```mermaid
graph TB
    subgraph "Osaka - Private Subnet"
        VM4[VM4: Monitoring<br/>Prometheus + Grafana + Loki<br/>1 OCPU, 6GB, 80GB]
        VM5[VM5: Backup<br/>Long-term Storage<br/>2 OCPU, 12GB, 70GB]
        VM6[VM6: Sandbox<br/>Development Environment<br/>1 OCPU, 6GB]
    end
    
    VM4 -.Monitor.-> Chuncheon
    VM5 -.Backup.-> Chuncheon
```

**Cross-Region Communication:**
- VCN Remote Peering Connection (RPC) between Chuncheon and Osaka
- Zero data transfer costs (within same OCI tenancy)
- Main services in Chuncheon minimize cross-region traffic

## Technology Stack

- **Backend (API Gateway, Auth Server)**: Python 3.12+ (FastAPI)
- **Frontend (Portfolio, Admin UI)**: React (Vite-based)
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

### Phase 3: Advanced Features
- [ ] Monitoring system
- [ ] CI/CD pipeline
- [ ] Backup system
- [ ] Performance optimization

## Future Expansion Plans

- **API Gateway (Bifrost)**: Service registration automation, API Key issuance, Rate Limiting
- **Auth Server (Bidar)**: OIDC integration, Role-based access control
- **Bantheon**: Project showcase additions, admin dashboard expansion
- **Baedalus**: Multi-CSP support (easy migration to AWS, Azure)
- **Bsgard**: OpenStack Neutron-based API wrapper completion for CSP-like VPC functionality

---

*BNGdrasil - Building a personal cloud nation, one service at a time.*
