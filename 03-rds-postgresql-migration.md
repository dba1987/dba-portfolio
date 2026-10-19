# 차세대1차 : Oracle → AWS RDS PostgreSQL 전환

[전체 목록으로 돌아가기](README.md)

| 항목 | 내용 |
| --- | --- |
| 기간 | 2023.07 ~ 2025.07 |
| 역할 | 고객사 DBA |
| 수행사 | SK AX |
| 원장 운영 | Oracle Exadata Cloud@Customer의 GALAXY 원장 유지 |
| 전환 범위 | GALAXY의 A&H·일반·보상 관련 스키마 분리 및 RDS PostgreSQL 전환 |
| 운영 규모 | AWS RDS PostgreSQL 약 30대 |

차세대 1차에서는 Oracle Exadata Cloud@Customer의 GALAXY 원장을 유지하면서 A&H·일반·보상 관련 스키마를 분리해 AWS RDS PostgreSQL로 전환했다.

RDS PostgreSQL의 Target 구성, DB Parameter Group 설정, 데이터베이스 인코딩(캐릭터셋) 선정과 초기 데이터 이관 방식을 검토했다. Oracle과 PostgreSQL의 데이터 타입, 숫자 정밀도·스케일, 문자 길이, 날짜·시간 표현, 기본값·NULL 처리와 PK·Unique Index의 차이를 확인하고 DDL 변환 기준을 검토했다.

초기 적재 이후의 OGG CDC 연계와 변경분 동기화, Source/Target 데이터 정합성 검증과 절체 절차를 함께 확인했다. 이를 바탕으로 수행사의 구성·이관안이 업무 데이터와 운영 조건에 맞는지 판단했다.

기존 Oracle에서 적용하던 솔루션 사용 정책, 개인정보 마스킹, 접근제어, 계정관리와 백업 기준을 신규 환경에 반영하도록 요구사항을 제시했다. 수행사 DBA의 구성·작업안을 검토하고 필요한 보완을 요청했으며 적용 여부를 확인했다.

실제 절체는 수행사 DBA가 수행했다. 나는 DB 영역의 전환 완료 여부를 판단하고 운영 인수인계로 연결했다. PostgreSQL→Oracle 정보계 Reverse CDC PoC는 프로젝트 DBA와 공동 수행했다. Oracle SR을 직접 진행하고 설정 변경을 제안·검증했으며, 당시 확인한 장애 대응 제약을 내부에 보고했다.

## 1. 고객사와 수행사의 역할

| 작업 | 맡은 범위 |
| --- | --- |
| 운영·보안 기준 | 고객사 DBA로 요구사항 제시와 적용 기준 판단 |
| 상세 구성·작업안·산출물 | 수행사 DBA가 작성·제시, 나는 검토와 보완 요구 |
| 작업 적용 | 수행사 실행과 이행 관리·감독 |
| DB 전환 완료 | 고객사 DBA로 결과 판단 |
| Reverse CDC PoC | 프로젝트 DBA와 공동 검증, Oracle SR 진행, 설정 변경 제안 및 운영상 위험 보고 |

## 2. 계정·권한과 운영 기준

Owner, 애플리케이션·배치, 조회, 연계, 모니터링과 이관 계정의 용도를 구분했다. Role과 Default Privileges를 이용한 권한 전달 기준, 신규 Table/Sequence의 권한 부여 범위를 검토하고 확정했다.

이관팀의 적재, 개발자의 데이터 보정, 정상 운영 접근권한을 분리했다. 작업 전에 임시 권한을 부여하고 완료 후 회수하는 시점을 인수인계 절차에 포함했다.

WAL·운영 로그, Autovacuum과 유지보수, 통계, 가용성, DB 크기와 인스턴스 변경 등 수행사 운영안을 검토했다. RDS의 관리단위와 제약에 맞춰 기존 Oracle 정책을 현행화하고 관련 준법·보안 문서에도 반영했다.

## 3. DDL 변환과 CDC 운영 절차

Oracle→PostgreSQL CDC 대상 101개 테이블의 데이터 타입, Default, NULL 허용 여부, PK·Unique Index와 Comment 변환 기준을 확인했다. 숫자 Precision/Scale, 문자 길이와 날짜·시간 표현 등 이기종 차이를 검토했다.

CDC는 OGG Mapping, DDL 변경 순서, 프로세스 상태, Lag와 처리건수, Replication Slot을 함께 확인하도록 운영 기준을 정리했다. 신규 테이블이나 컬럼 변경에서는 Source/Target DDL과 Mapping·Trandata 변경, GAP 확인, 프로세스 재기동을 하나의 변경 절차로 관리하도록 했다.

## 4. Reverse CDC PoC와 장애 대응 제약

### 공동 PoC에서 확인한 문제

프로젝트 DBA와 PostgreSQL→Oracle 정보계 Reverse CDC PoC를 함께 진행했다. 대량 데이터 처리 중 WAL Sender 연결 종료, OGG 재접속과 Extract ABEND가 발생했고, WAL 누적과 Storage 증가도 관찰했다.

PostgreSQL 로그에는 replication timeout으로 WAL Sender를 종료한 기록이 남아 있었다. OGG 로그에서는 연결 종료·재접속에 이어 Positioning sequence ID is out of order 오류를 확인했다.

프로젝트 DBA는 Extract 재기동 후 Lag이 없는 상태에서도 데이터를 비교했다. 동일 조건의 조회 건수가 PostgreSQL 17,465건, Oracle 612,083건으로 달랐다. 이 값은 테이블 전체 건수가 아닌 특정 조건의 조회 결과다. 프로세스가 실행 중인지와 데이터가 일치하는지를 별도로 확인해야 하는 상황이었다.

### Oracle SR과 설정 변경 검증

나는 Oracle SR을 직접 진행하고 오류와 데이터 불일치에 대한 기술지원 회신을 확인했다. 당시 Oracle은 보고된 제품 결함이나 유사 사례가 없으며, 연결 종료·Invalid socket 등의 오류가 확인된다고 설명했다. `wal_sender_timeout=0`은 유효한 Workaround로 안내받았다.

이를 근거로 파라미터 변경을 제안했다. OGG 전용 계정의 `wal_sender_timeout=0` 설정을 대량 동기화 테스트에 적용하고 프로젝트 DBA·벤더와 공동 검증했다. 적용 후 PoC에서 연결 종료 관련 경고와 Extract ABEND가 재발하지 않는 것을 확인했다.

설정 적용 후 발생한 `OGG-25546`은 Timeout 비활성화를 알리는 정상(expected) 메시지라는 Oracle 답변도 확인했다. 이를 연결 종료 관련 오류와 구분해 결과를 판단했다.

### Failover 대응 제약과 운영상 판단

SR 과정에서 당시 PostgreSQL Capture의 Failover 대응 제약을 추가로 확인했다. 대량 트랜잭션 처리 중 비계획 Failover가 발생하면 일부 변경분의 추출이 정상적으로 이어지지 않을 수 있다는 기술지원 안내였다.

업체는 사전에 프로세스를 정상 종료한 뒤 전환하는 절차를 안내했다. 나는 이 방법을 예고 없이 발생하는 장애에 적용하기는 어렵다고 판단했다. 장애 시 데이터 정합성 검증과 재적재가 필요할 수 있다는 점, Long Transaction을 줄이는 운영 관리가 필요하다는 의견을 내부에 보고했다.

### 수행 결과

공동 PoC에서 연결 안정화 방안을 검증했다. SR에서 확인한 당시의 Failover 대응 제약과 운영상 위험은 내부에 보고했다.

## 5. 백업·가용성·보안과 전환 완료 판단

백업 보관·삭제, 복구 단위, 가용성, 암호화와 접근통제 요구사항을 검토했다. RDS의 인스턴스 복구와 DB/Table 단위 백업 필요성을 구분하고, 기존 정책을 신규 환경에서 적용할 수 있는 범위로 정리했다.

구성, 계정·권한, CDC, 정기 점검과 정책 변경을 운영 인수인계에 반영했다. 수행사 작업의 결과와 기준 충족 여부를 확인한 뒤 DB 영역의 전환 완료를 판단했다.

## 현재 RDS PostgreSQL 운영 범위

AXA의 AWS RDS PostgreSQL 약 30대에서 계정·권한, 오브젝트 변경, 성능 모니터링과 운영 변경을 수행하고 있다. pg_stat_activity, Vacuum/Analyze 상태, Replication Slot, Extension과 SSL 등 운영 조회 항목을 확인한다.

[전체 목록으로 돌아가기](README.md)
