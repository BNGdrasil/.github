# 구현 교차 리뷰 — 2026-09-18

## 결론

계획의 핵심 방향은 구현에 반영됐다. 공개 가입 차단, access/refresh 구분, DB 기반 현재 권한 검증, 구형 무인증 등록 API 제거, 실제 사용자 관리 중계, 모니터링 계측, Monad 계열 화면 개편을 확인했다. **운영 투입 완료로 판정하지 않는다.** 아래 미해결 배포·복구 문제를 먼저 해결하고 실제 환경 검증을 수행해야 한다.

검토한 HEAD: Baedalus `0091d1f`, Bantheon `8dddcdb`, Bidar `b9493d3`, Bifrost `f71f4f7`, 조직 문서 `23fc1cc`. 리뷰 작성 시 수정은 위 커밋 위의 미커밋 변경이었다. 기존 미추적 `baedalus/docs/deployment-inventory.md`와 개선 문서는 보존했다. 운영 SSH·배포·Terraform state 읽기/변경·새 의존성 설치·푸시는 수행하지 않았다.

## 직접 수정한 작은 결함

| 대상 | 문제와 수정 | 검증 |
|---|---|---|
| `baedalus/vm2-deployment/deploy-image.sh` | 실패 후 예전 `.env`만 복원하면 `main` 같은 가변 태그는 이미 새 이미지다. 기존 컨테이너의 image ID로 만든 rollback 태그를 `.env`에 명시하고 재기동하도록 수정 | Docker/curl 대역을 사용해 동일 `main` 배포 실패 → 두 번째 compose가 rollback 태그를 사용하고 다른 env 항목 보존 확인 |
| `baedalus/backup/ship.sh` | `--dry-run`도 EXIT trap에서 전송 성공 상태/metric을 갱신했다. dry-run은 성공/실패 기록을 바꾸지 않도록 수정 | 이전 성공 JSON 유지, 새 last-run/metric/SHIPPED 생성 없음 확인 |
| `bifrost/src/api/api.py` | Content-Length 없는 본문을 `request.body()`로 전부 메모리에 읽은 뒤 제한 검사. chunk마다 누적 크기를 검사하고 초과 즉시 413 | 수정 전 4 bytes 제한에서 15 bytes/3 chunks 소비. 수정 후 첫 초과 chunk에서 종료. 기존 바이너리/프록시 계약도 통과 |
| `bifrost/tests/test_config.py` | production 설정 테스트가 외부 ALLOWED_HOSTS/SECRET_KEY/DB 설정에 의존. 테스트 안에서 유효한 입력을 명시 | wildcard 환경에서 실패했던 2건을 포함해 전체 294 passed. production wildcard 거절 동작은 유지 |
| `baedalus/.github/workflows/deploy-backup.yml` | 목적지의 `ssh -i id_deploy`는 ProxyJump 자식 SSH에 그대로 적용되지 않음. 두 홉이 읽는 SSH config에 IdentityFile/IdentitiesOnly/BatchMode/StrictHostKeyChecking 명시 | 실제 추가 config를 추출해 두 호스트에 `ssh -G -F` 적용, 키·strict 설정 확인. 실제 Actions/2-hop 인증은 미실행 |

회귀 검사: `python3 -m unittest discover -s baedalus/tests -v`의 2개 검사, Bifrost 기존 테스트 파일에 추가한 chunked body 검사. 테스트용 Docker/SSH 대역은 운영 서비스에 연결하지 않는다.

## Claude에게 넘기는 미해결 리뷰

### R1 · P1 · Nginx 두 번째 배포부터 이전 설정으로 온전히 복원할 수 없음

- 위치: `bantheon/scripts/vm1-apply-nginx.sh:115`, `:161`, `:199`; 동일 흐름 `bngdrasil/deploy_vm1.sh:176`, `:238`.
- `nginx.conf`와 Compose만 백업하고 `snippets/`는 `rsync --delete`로 덮어쓴다. 첫 전환 이후 구 설정도 snippet을 include하므로 이전 nginx.conf만 복원하면 새 snippet과 섞인다. “이전 설정에는 include가 없다”는 복구 안내는 첫 배포에만 성립한다.
- 로컬 대역 재현: 기존 snippet=OLD, 새 snippet=NEW, 선검증 성공 후 compose 실패. config 백업 1개, snippet 백업 0개, live snippet은 NEW로 남는다.
- 요청: nginx.conf + snippets + Compose를 같은 release 단위로 보존하고 두 배포 진입점의 복구 절차를 맞춘다. 실제 반영된 image도 식별할 수 있게 기록한다.
- 완료: 연속 두 번 이상 배포한 뒤 snippet/Compose 적용 단계 실패를 주입하고, 이전 전체 설정 해시·nginx 문법·핵심 vhost 응답이 복구되는지 검증한다.

### R2 · P1 · 백업 보존·전송·경보 정책이 서로 맞지 않음

- 위치: `baedalus/backup/retention.sh:94`, `:117`; `backup/ship.sh:217`; `backup/systemd/bngdrasil-backup-ship.timer:6`; `monitoring/prometheus/rules/basic.yml`의 BackupStale.
- retention은 전송 완료 여부를 보지 않고 같은 날 최신 성공본만 남긴다. 00:10/06:10/12:10 성공본을 모두 미전송으로 준비한 로컬 dry-run에서 앞의 두 백업이 삭제 대상으로 나왔다. ship과 retention의 잠금도 별개라 전송 중 원본 정리와 경합할 수 있다.
- 기본 전송은 12시간 주기인데 BackupStale은 component 구분 없이 8시간 기준이다. ship metric에도 적용되므로 정상 전송 주기 중 경보가 발생한다. 호스트 손실에 대한 독립 사본 RPO 6시간이라는 기존 제안도 이 주기로 충족되지 않는다.
- 요청: 로컬/오프사이트 RPO를 구분해 전송 주기와 경보를 맞추고, 미전송·전송 중 백업의 보존 및 전송 후 검증 정책을 정한다. 무조건 무기한 보존으로 디스크가 차는 경우도 실패·용량 경보로 드러내야 한다.
- 완료: 전송 실패·지연·retention 동시 실행·호스트 손실 시나리오를 시험하고, 유효한 독립 복원본과 실제 달성한 RPO를 확인한다. 이 리뷰에서 고친 dry-run 기록 문제와 별개의 남은 작업이다.

### R3 · P1 · 릴리스가 동일 SHA의 전체 CI 성공을 요구하지 않음

- 위치: `bidar/.github/workflows/release.yml:116`, `:187`; `bifrost/.github/workflows/release.yml:118`, `:189`; 양쪽 `ci.yml`의 lint/security job.
- release는 자기 test→build→deploy만 기다린다. 별도 ci.yml의 lint·bandit·pip-audit 실패 여부는 의존성에 없으므로 release DAG 자체로 배포를 막지 않는다. 실제 branch protection/environment 설정은 이번에 조회하지 않았다. 사람이 승인 시 확인하는 것과 자동 gate는 구분해야 한다.
- 요청: 공유 validation workflow 또는 동일 commit의 전체 CI 결과에 의존하도록 release 경로를 묶는다. 기존 artifact 롤백을 위한 수동 경로는 새 코드 release와 예외 정책을 구분한다.
- 완료: 단위 테스트는 통과하지만 lint/security가 실패하는 SHA에서 이미지 운영 배포가 시작되지 않는 것을 Actions에서 검증한다. 단순히 다른 workflow의 “가장 최근 성공”을 조회하지 말고 동일 SHA를 확인한다.

### R4 · P2 · 한 앱 배포가 다른 앱의 현재 release 보관본을 삭제함

- 위치: `bantheon/scripts/vm1-release-static.sh:83`, `:103`, `:108`.
- 보존 기준이 공용 `releases/<sha>`이고 이번 앱의 SHA만 보호한다. admin만 여러 번 배포하면 `current-client`가 가리키는 오래된 SHA도 삭제된다.
- 로컬 대역 재현: KEEP_RELEASES=1, client는 old SHA, admin은 new SHA. admin 배포 exit 0인데 client 현재 release 디렉터리는 삭제되고 current-client 링크가 끊긴다. 복사된 client/dist는 남아 있어 즉시 서비스 중단은 아니지만 재배포·롤백 원본이 사라진다.
- 요청: 두 앱의 현재 release를 모두 보호하고 보존 정책을 앱별로 정의한다. 정적 파일 부분 동기화 실패와 되돌리기 경로도 함께 검증한다.
- 완료: 한 앱만 보존 세대 수 이상 배포해도 다른 앱의 현재 release·최소 롤백 지점이 보존되는지 확인한다.

## 계획 대비 아직 완료가 아닌 범위

| 묶음 | 판정과 남은 조건 |
|---|---|
| I00 | 문서·기준선 확보. deployment-inventory.md는 아직 미추적, 실제 환경/state 차이 확정 필요 |
| I01 | 핵심 코드 및 SQLite 회귀 검증됨. PostgreSQL 실제 동시 관리자 변경·운영 계정/키 전환·origin 제한은 별도 검증 |
| I02 | 백업 도구 구현. R2, VM 설치·외부 보관·알림 수신·앱/권한 복원 미검증. `/opt/bnbong` 설정·모델·인증서의 반복 백업은 run.sh의 PG/Redis/SQLite 경로에 포함되지 않음 |
| I03 | 로컬 빌드 기반 있음. 실제 Actions 실행, R1/R3/R4 해결과 불량 release 복귀 증거 필요 |
| I04 | PG17 사전 시험 경로. 운영 전환 및 달력 기한 관리는 별도 |
| I05 | 인증된 등록부와 프록시 개선 반영. 이번 body 제한 보완. buffered proxy·단일 worker라는 한계를 유지 |
| I06 | metric·exporter·rule 구성. Alloy/독립 외부 probe와 실제 경보 수신은 미완료; rules 파일만으로 통지 완료 아님 |
| I07 | 사용자 관리와 fake write/통계 정리 반영. 백업 현황·최신 관측 시각 등 08의 개요 데이터 계약 전체는 아직 미구현 |
| I08 | 소스 구조·팔레트·빌드 확인. 이번 리뷰에서 브라우저 전체 21화면/E2E를 재실행하지 않았고 기존 기록과 구분 |
| I09 | 코드 조건화·퇴역 문서. 실제 OCI/state 대조 및 퇴역 절차 미실행. 존재하는 자원을 무조건 state rm 하지 않음 |

## 실제 실행한 검증

- Bidar: **183 passed**, SQLite 메모리 DB, 2 dependency deprecation warnings.
- Bifrost: **294 passed**, SQLite 임시 DB. 최초 291 passed/2 failed는 production 설정 테스트의 외부 host 값 의존성이었고, 테스트 입력 격리 후 전부 통과.
- Bantheon: admin/client `npm run build` 성공.
- Baedalus: 신규 stdlib 회귀 2개 통과; 변경 shell bash -n·shellcheck 통과(공통 helper source 경로를 지정).
- Bifrost 변경 Python: black·flake8 통과.
- Nginx 실패/정적 release 보존: 임시 디렉터리와 Docker/rsync 대역으로 재현. 실제 nginx 컨테이너/운영 서버 장애 주입이 아님.
- SSH: 생성 config의 두 호스트 유효 옵션만 확인. 네트워크 접속 없음.
- **미실행:** 실제 GitHub Actions, 운영 배포, live API 공격, 전체 PostgreSQL 통합/동시성 시험, 새 DB 복원, 최신 dependency 보안 재감사, Terraform plan/apply.

기존 테스트 수·로그에 적힌 검증이 모든 운영 완료 조건을 충족했다는 의미로 확대되지 않도록 실행 로그의 구현/로컬 검증/운영 적용을 분리해 유지한다.
