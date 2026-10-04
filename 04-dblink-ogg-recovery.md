# DBLink 지연 장애 분석 및 OGG 복구 안정화

[전체 목록으로 돌아가기](README.md)

IDC 간 DBLink 호출에서 약 900초 동안 응답이 지연된 뒤 오류가 발생했다. 특정 SQL과 연결 사용 조건에서만 반복됐고, 같은 업무의 다른 호출은 정상이었다.

SQL 실행시간뿐 아니라 방화벽의 유휴 세션 만료와 WAS 연결 재사용 조건을 함께 확인했다. 이후 Health Check로 연결을 유지하는 방안을 검증하고, 해당 조회와 관련된 OGG Long Transaction 문제까지 후속 대응했다.

## 1. 장애 조건 재현

방화벽 유휴 세션 만료 시간을 10분으로 설정한 테스트에서 최초 조회는 1초 이내에 끝났다. 20분 이상 유휴 상태 후 같은 연결을 재사용하면 약 950초 지연과 ORA-03150·ORA-02063 오류가 재현됐다.

이 전후 비교로 방화벽에서 만료된 연결과 WAS에 남은 연결의 재사용을 문제 발생 조건으로 좁혔다.

## 2. 대안 비교와 개선 확인

OS TCP Keepalive, Oracle Net의 SQLNET.EXPIRE_TIME, WAS Check Query와 물리 연결 재생성을 비교했다. 공통 설정의 다른 서비스 영향과 애플리케이션 변경 범위를 고려해 해당 WAS 연결 풀에서 실제 DBLink 경로를 주기적으로 조회하는 방안을 제안했다.

로컬 DB만 확인하는 쿼리로는 원격 DBLink 경로의 유휴 상태를 해소할 수 없어서 원격 조회를 포함했다.

방화벽 조건을 유지한 채 Health Check 주기를 3분으로 설정했다. 이후 30분 유휴와 추가 약 1시간 50분 유휴 후에도 재조회가 1초 이내에 끝나는 것을 확인했다.

## 3. 후속 OGG 문제

Health Check 적용 이후 관련 트랜잭션이 OGG Long Transaction으로 남고 Extract의 Recovery Checkpoint가 과거 시점에 머무는 문제를 확인했다. 정보계·DR·AWS RDS PostgreSQL 방향 CDC의 복구 처리와 Archive Log 보존에 영향을 줄 수 있었다.

### 트랜잭션 분리 시도

데이터소스 전체 Auto-Commit은 업무 트랜잭션의 커밋 경계를 바꿀 수 있어 적용하지 않았다. Health Check만 별도 COMMIT하기 어려운 WAS 제약을 확인하고, PRAGMA AUTONOMOUS_TRANSACTION을 사용한 DB 함수 SQL을 작성·제공했다.

함수에서 DBLink를 조회하고 성공 시 COMMIT, 예외 시 ROLLBACK하도록 구성했다. 개발과 운영 테스트 후 적용을 진행했지만, 운영에서는 함수 변경만으로 Checkpoint 정체가 해소되지 않았다.

### 로그 보존과 Bounded Recovery

Recovery Checkpoint 시점까지 복구에 필요한 Archive Log가 남도록 보관 정책을 재설정했다. 관련 세션 재기동으로 Checkpoint를 전진시키는 임시 대응도 유지했다.

장기 트랜잭션이 지속되는 상황에서 재기동 복구 부담을 관리하기 위해 Bounded Recovery 적용을 추진했다. 정보계와 AWS RDS 방향 Extract에 BR 정상 적용을 확인했다.

당시 운영 판단은 BR 간격을 4시간 기준으로 관리하는 것이었다. Source와 Target을 포함한 재기동 후 재처리 시간 약 30분은 예상치였으며, 실제 장애 복구시간을 측정한 성과는 아니다.

## 결과

동일한 방화벽·연결 재사용 조건에서 지연을 재현하고, 주기적 원격 조회의 개선 효과를 확인했다. 이후 OGG 영향도 계속 추적했다. 함수 변경으로 해결되지 않은 범위를 남겨둔 채 로그 보존과 BR 적용으로 복구 위험을 관리했다.

[전체 목록으로 돌아가기](README.md)
