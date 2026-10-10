# Oracle 12.2→19c 업그레이드

[전체 목록으로 돌아가기](README.md)

| 항목 | 내용 |
| --- | --- |
| 기간 | 2022.03-2022.08 |
| 역할 | DBA 실무 리딩 / 공식 PM 별도 |
| 대상 | 운영·정보계·DR·개발/테스트 DB, Exadata Grid Infrastructure, GoldenGate 연계 |

공식 PM과 협업하며 DBA 테스트와 본 작업을 주도했다. 내부 실무진, 업무 담당자와 Oracle 기술인력의 작업 순서와 검증 항목을 조율했다. 당시에는 현재의 DB 파트 리드 역할을 맡기 전이었다.

## 1. 운영 백업으로 테스트 환경 구성

운영 DB에는 파라미터·권한·오브젝트·데이터 딕셔너리 상태가 누적돼 있었다. 신규 DB에서 업그레이드가 성공하는 것만으로는 운영 작업의 문제를 확인하기 어렵다고 판단했다.

실제 운영 백업을 RMAN으로 복구한 별도 검증 환경에서 업그레이드를 반복했다. 복구, 사전점검, AutoUpgrade Analyze·Fixups·Deploy, 사후점검을 수행하고 발견한 문제를 본 작업 절차에 반영했다.

## 2. GI·DB 패치와 사전점검

Exachk, preupgrade와 AutoUpgrade Analyze, CVU와 GI Dry-Run 결과를 확인했다. ASM Password File, HugePages, 디스크 공간과 권장 설정, 기존 파라미터와 Job 상태도 점검했다.

GI/DB의 RU·RUR, One-off Patch 사이의 충돌 여부를 확인하고, 필요한 선행 원복과 재적용 순서를 검토했다. 버전만 맞추는 것이 아니라 실제 적용할 패치 조합과 원복 경로를 함께 준비했다.

## 3. AutoUpgrade 테스트에서 확인한 문제

| 문제 | 처리한 내용 |
| --- | --- |
| Timezone 업그레이드와 병렬 설정 관련 오류 | 기존 파라미터가 업그레이드 단계에 영향을 주는 조건을 확인하고 사전조치에 반영 |
| Fixups의 UPG-1316 / ORA-20000 | DBMS_STATS 실행과 관련된 권한 상태를 확인하고 필요한 권한을 보완한 뒤 Fixups 재수행 |

Analyze 결과와 Fixup 항목을 확인하고, 오류를 조치한 뒤 다음 단계로 진행했다. Deploy 후에는 Registry, INVALID Object, Timezone, Wallet과 DB 상태를 확인했다.

## 4. DR 선행 검증과 본 작업

DR 3개 DB를 먼저 업그레이드하고 사후 검증한 뒤 운영 전환을 진행했다. 실제 운영계열 환경에서 작업 순서와 연계 시스템 확인 절차를 한 번 더 검증했다.

본 작업에는 다음 순서를 포함했다.

1. DB·OS 백업과 사전 상태 확인
2. Batch/WAS, OGG, 백업과 모니터링 프로세스 중단
3. GI·DB 업그레이드와 상태 점검
4. GoldenGate 업그레이드, 프로세스 기동과 동기화 확인
5. WAS·Batch 기동과 업무담당자 검증
6. NetBackup·MaxGauge를 신규 버전에 맞게 재연결하고 정상 동작 확인

DB가 OPEN된 시점과 서비스가 정상화된 시점을 구분해서 관리했다.

## 5. 실패 단계별 원복 준비

| 실패 범위 | 준비한 복구 경로 |
| --- | --- |
| Grid Infrastructure | GI 업그레이드 원복 절차 |
| Database 업그레이드 | AutoUpgrade restore_on_fail 기반 복구 경로 검토 |
| 데이터 복구 필요 | NetBackup 기반 RMAN Full Backup Restore/Recover |

GI 원복과 DB 원복을 별도로 준비하고, 원복 이후에도 데이터 상태 확인이 필요하면 백업 복구로 이어가도록 구분했다.

## 결과

운영·정보계·DR·개발/테스트 DB의 19c 전환과 연계 시스템 정상화를 완료했다. 실제 운영 백업을 이용한 반복 테스트에서 오류와 환경 차이를 사전 확인하고, 본 작업과 복구 절차에 반영했다.

[전체 목록으로 돌아가기](README.md)
