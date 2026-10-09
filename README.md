# 박건태 | Database Administrator

Oracle과 Exadata를 중심으로 약 10년간 DB 운영, 구축과 전환 업무를 해왔습니다. 현재 AXA손해보험의 DB 파트 리드로 Oracle 약 25대, AWS RDS PostgreSQL 약 30대, MSSQL 17대의 운영과 변경 관리를 담당하고 있습니다.

DB 변경 작업에서는 업무 영향과 복구 방법을 먼저 확인합니다. 이관이나 업그레이드도 DB 기동에서 끝내지 않고, 데이터 정합성·연계 서비스·백업과 모니터링이 정상 동작하는지까지 확인해 왔습니다.

## 주요 경험

- **ExaCC 전환 PM / Main DBA**: 기간계·정보계·DR·개발을 포함한 Oracle 25개 DB 전체의 전환과 안정화. 이관 방식 선정, 성능·가용성·복구 검증, 절체와 원복 절차 관리.
- **Oracle 12.2→19c 업그레이드 실무 리딩**: 운영 백업으로 테스트 환경을 구성하고 AutoUpgrade 오류와 패치 충돌을 사전 확인. DR 선행 검증 후 운영 전환과 연계 시스템 정상화.
- **Oracle→RDS PostgreSQL 전환 고객사 DBA**: 운영·보안 기준 제시, 수행사 기술안 검토와 이행 관리, DB 영역 전환 완료 판단. PostgreSQL→Oracle Reverse CDC PoC 직접 참여.
- **장애 분석과 이관 자동화**: DBLink 지연 조건 재현, 후속 OGG 복구 위험 관리, Tibero→Oracle 이관과 Python·Shell 기반 반복 작업 구성.
- **DB 보안 및 감사 대응**: 계정·권한 분리, DB 접근제어 운영, TDE 적용 환경의 전환 작업 검토, 금융감독원·외부회계법인·AXA 프랑스 그룹 감사 대응.

- **전사 개인정보 컬럼 암호화(진행 중)**: 2026.03부터 고객사 DBA·DB 파트 리드로 참여. 전체 대상 합산 약 500억 건의 대상 조사, 클론 DB 구성과 보안 설정, 암호화 데이터의 OGG CDC 검증 직접 수행.

## 프로젝트별 상세 내용

| 프로젝트 | 맡은 역할 | 확인할 내용 |
| --- | --- | --- |
| [Exadata Cloud@Customer X10M 전환](01-exacc-migration.md) | PM / Main DBA | 이관 방식 선정, CDB/PDB와 파라미터 검토, 복구 검증, Reverse OGG 원복, AS-IS / TO-BE 구성도 |
| [Oracle 12.2→19c 업그레이드](02-oracle-upgrade.md) | DBA 실무 리딩 | 운영 백업 기반 테스트, AutoUpgrade 오류 대응, GI/DB 패치 검토, 단계별 원복 |
| [Oracle→AWS RDS PostgreSQL 전환](03-rds-postgresql-migration.md) | 고객사 DBA | 수행사와의 역할 구분, 계정·권한 기준, CDC와 WAL Sender 연결 안정화 검증 |
| [DBLink 지연 분석 및 OGG 복구 안정화](04-dblink-ogg-recovery.md) | 원인 분석·조치 제안·후속 대응 | 약 950초 지연 재현, Health Check 검증, 함수 변경의 한계, 로그 보존과 Bounded Recovery |
| [AnyLink Tibero→Oracle 전환](05-tibero-oracle-migration.md) | 대상 DB 구성·이관·오류 대응 | AI 코딩 지원을 활용한 작업 제어, CSV·LOB 처리, Gateway, 개발 검증과 운영 이관 완료 |
| [SQL 튜닝 접근과 수행 경험](06-sql-tuning.md) | SQL 성능 분석·튜닝 | 업무 성격과 실제 처리량 판단, 실행계획 검토, 약 1.06억 건 테이블 반복 조회 구조 개선 |
| [반복 작업 자동화 및 클라우드 운영](07-automation-cloud-operations.md) | 운영 작업 구성·수행 | PostgreSQL 약 800개 테이블의 배치 이관, Terraform 설정 변경, MSSQL RDS 이관 |
| [DB 암호화·접근통제·감사 대응](08-security-audit.md) | 고객사 DBA / 현재 DB 파트 리드 | TDE 수행사 작업 검토, 계정관리 개선, 감사 대응, 통합 솔루션 PoC와 추진 범위 |
| [전사 개인정보 컬럼 암호화와 데이터 전환](10-column-encryption.md) | 고객사 DBA / DB 파트 리드 | 전체 대상 합산 약 500억 건, 클론 구성과 Wallet 작업, 암복호화 함수·권한·감사 설정, OGG 정상 반영·복호화·DML 검증 |
| [이전 경력과 KB국민은행 THE K](09-previous-experience.md) | 구축·운영·전환 DBA | Db2 pureScale 설치, Oracle SQL 호환성 검증, 가용성 테스트 참여와 사용 가이드 작성 |

## 기술 경험

| 영역 | 사용 기술과 업무 |
| --- | --- |
| DBMS | Oracle, PostgreSQL, Microsoft SQL Server, Tibero, IBM Db2 |
| Oracle 플랫폼 | Exadata / Exadata Cloud@Customer, RAC, ASM, CDB/PDB |
| 이관·복제·복구 | RMAN, NetBackup, GoldenGate CDC, Restore/Recover, DR, 절체·원복 |
| 성능 분석 | AWR, ASH, SQL Monitor, 실행계획, SQL 튜닝, Exadata Smart Scan |
| 클라우드 | AWS RDS PostgreSQL / Oracle / MSSQL, CloudWatch |
| 반복 작업 | Shell, Python, JDBC, SQL*Loader, pg_dump / pg_restore |
| IaC | 기존 Terraform 코드의 RDS PostgreSQL 파라미터 값 수정과 apply 적용 |
| DB 보안 | TDE 적용 환경 운영·전환 작업 검토, 컬럼 암호화·OGG PoC, 계정·권한·접근제어, 감사 대응 |

## 경력

| 기간 | 소속 / 프로젝트 |
| --- | --- |
| 2022.03 ~ 현재 | AXA손해보험 DBA / 현재 DB 파트 리드 |
| 2021.11 ~ 2022.03 | 방위사업청 국방통합원가시스템 고도화 |
| 2021.01 ~ 2021.04 | KT ds 대국민 서비스 |
| 2019.04 ~ 2021.04 | KB국민은행 THE K |
| 2018.10 ~ 2019.03 | 비즈폼 신규 서비스 |
| 2016.06 ~ 2018.09 | 트리콜 Infra / DBA |

각 프로젝트 문서에는 직접 수행한 작업, 수행사 작업을 검토·판단한 범위, 업무담당자와 분담한 검증을 구분해 적었습니다.

