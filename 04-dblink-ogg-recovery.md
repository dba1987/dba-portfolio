# DBLink 지연 장애 재현 및 OGG 복구 안정화

[전체 목록으로 돌아가기](README.md)

ExaCC 전환 이후 자동차보험 콜센터에서 발생한 약 900초의 DBLink 조회 지연에 대응. 장애 조건 재현과 Health Check 검증을 진행하고, 이후 정보계·DR·AWS RDS로 이어지는 OGG 복제의 복구 위험까지 후속 조치.

## 업무 영향과 초기 판단 수정

자동차보험 콜센터의 TODO 조회 과정에서 원격 DB 연결 확인 SQL(select 1 from dual@dblink)이 약 900초 지연된 뒤 오류 발생. 운영 분석 기록에서 같은 업무·SQL·WAS 컨테이너의 지연 요청 19건 확인.

기존·신규 환경의 유휴시간 관련 설정을 비교하며 방화벽 세션 만료를 우선 의심했으나, 비교값과 설정 의미를 잘못 해석했음을 확인. 네트워크 담당자와 실제 방화벽 설정을 확인해 기존과 동일한 정책이 유지되고 있음을 파악하고 초기 판단 수정. 서버·네트워크 담당자와 다른 가능성을 함께 검토하고, 테스트 환경에서 유휴 상태와 연결 재사용 조건을 재현하도록 제안하며 협의 진행.

## 장애 조건 재현과 개선 검증

방화벽 세션 만료 시간을 10분으로 설정한 테스트 환경에서 WAS 재기동 후 최초 조회는 1초 이내 완료. 20분 이상 유휴 상태 후 재호출 시 약 950초 지연과 오류 재현. 유휴 상태 이후 기존 연결을 다시 사용하는 조건에서 지연이 발생함을 확인.

OS TCP Keepalive, Oracle Net의 SQLNET.EXPIRE_TIME, WAS Check Query, 호출마다 물리 연결을 새로 생성하는 방안을 비교. 공통 설정의 다른 서비스 영향과 애플리케이션 변경 범위를 고려해 해당 WAS 연결 풀에서 실제 원격 DBLink 경로를 주기적으로 조회하는 방안 제안.

방화벽 만료 시간 10분을 유지한 채 Health Check 주기를 3분으로 설정. 30분 유휴 후 테스트 API 응답 682ms, 추가 약 1시간 50분 유휴 후 93ms 확인. 운영 적용 이후 동일한 장시간 조회 지연은 재발하지 않음. 다만 ExaCC 전환 전후의 동작 차이를 발생시킨 최종 원인은 미확정.

## Health Check 적용 이후 OGG 복구 위험 대응

Health Check 적용 이후 DBLink 조회 관련 트랜잭션이 OGG Long Transaction으로 남으면서 Extract의 Recovery Checkpoint가 과거 시점에 머무는 문제 확인. 정보계·DR·AWS RDS PostgreSQL로 데이터를 전달하는 세 개 Extract에 영향.

Extract 재기동 시 오래된 시점부터 복구해야 하는 범위가 커져 동기화 재개가 지연될 수 있었고, 복구에 필요한 Archive Log가 보관 정책에 따라 먼저 삭제될 위험도 확인. 조회 지연 개선 이후 발생한 CDC 재기동과 로그 보존 문제에 추가 대응.

### 업무 트랜잭션과 Health Check 분리

WAS 담당자·기술지원과 DBLink 조회 후 트랜잭션을 종료하는 방안 검토. 데이터소스 전체 Auto-Commit 변경은 업무 트랜잭션의 커밋 경계에 영향을 줄 수 있어 적용하지 않음. Health Check에만 별도 COMMIT을 수행하기 어려운 WAS 제약을 확인하고 DB 함수에서 독립 트랜잭션을 처리하는 방안 선택.

PRAGMA AUTONOMOUS_TRANSACTION을 사용한 FN_DBLINK_ALIVE_CHECK 함수를 직접 작성·제공. 함수 내부에서 로컬 DB와 두 개 DBLink를 조회하고, 성공 시 COMMIT, 예외 시 ROLLBACK하도록 구성. WAS Check Query를 해당 함수 호출 방식으로 변경.

### 함수 적용의 한계 확인

개발 및 운영 테스트를 거쳐 적용했으나, 후속 운영 확인에서 함수 변경만으로 Recovery Checkpoint 정체가 해소되지 않음을 확인. OGG 재기동 시 복구 처리와 필요한 로그 보존을 위한 후속 조치 진행.

### Archive Log 보존과 Bounded Recovery 적용

Recovery Checkpoint 시점까지 복구에 필요한 Archive Log가 남도록 보관 정책 재설정. 주 1회 WAS 야간 재기동에 따른 관련 세션 종료로 Checkpoint를 전진시키는 임시 대응 병행.

장기 트랜잭션이 지속되는 상황에서 Extract 재기동 시 복구 부담을 줄이기 위해 Bounded Recovery 적용 추진. 정보계와 AWS RDS PostgreSQL 방향 Extract에 정상 적용 확인.

BR 적용 후 Extract 재기동과 동기화 재개까지 확인하고 후속 조치 완료.

## 결과

- 테스트에서 약 950초 지연을 재현하고 Health Check 적용 후 1초 이내 응답 확인. 운영 적용 이후 동일 지연 재발 없음.
- 업무 트랜잭션의 커밋 경계를 유지하도록 Health Check를 분리하는 함수를 직접 작성하고, 운영에서 확인된 조치의 한계를 후속 대응에 반영.
- 정보계·DR·AWS RDS 복제에 필요한 Archive Log의 보관 정책을 조정하고 임시 대응을 병행해 복구에 필요한 로그 보존.
- 정보계·AWS RDS 방향 Extract의 BR 적용 후 재기동과 동기화 재개까지 검증.

[전체 목록으로 돌아가기](README.md)
