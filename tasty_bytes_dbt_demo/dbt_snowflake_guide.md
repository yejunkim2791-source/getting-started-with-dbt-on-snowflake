# dbt on Snowflake 가이드

## 1. profiles.yml (프로필 설정)

### 개요

`profiles.yml`은 dbt가 Snowflake에 연결하기 위한 설정 파일입니다. Snowflake 네이티브 dbt 환경에서는 세션 인증을 사용하므로 `account`와 `user` 값은 실제로 사용되지 않습니다.

### 구조

```yaml
tasty_bytes:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: 'not needed'
      user: 'not needed'
      role: accountadmin
      database: tasty_bytes_dbt_db
      schema: dev
      warehouse: tasty_bytes_dbt_wh
      threads: 8
    prod:
      type: snowflake
      account: 'not needed'
      user: 'not needed'
      role: accountadmin
      database: tasty_bytes_dbt_db
      schema: prod
      warehouse: tasty_bytes_dbt_wh
      threads: 8
```

### 각 필드 설명

| 필드 | 설명 |
|------|------|
| `target` | 기본 실행 환경. `dbt run` 시 별도 지정 없으면 이 타겟 사용 |
| `type` | 연결 대상 데이터베이스 종류 (`snowflake`) |
| `account` | Snowflake 계정 식별자. 네이티브 dbt에서는 세션 인증 사용으로 불필요 |
| `user` | Snowflake 사용자명. 네이티브 dbt에서는 세션 인증 사용으로 불필요 |
| `role` | SQL 실행 시 사용할 Snowflake 역할 |
| `database` | 모델이 생성될 대상 데이터베이스 |
| `schema` | 모델이 생성될 대상 스키마 |
| `warehouse` | 쿼리 실행에 사용할 웨어하우스 |
| `threads` | 동시 병렬 실행할 모델 수 |

### 환경 분리 (dev / prod)

- **dev**: 개발 환경. `TASTY_BYTES_DBT_DB.DEV` 스키마에 모델 배포
- **prod**: 운영 환경. `TASTY_BYTES_DBT_DB.PROD` 스키마에 모델 배포

### 사용 방법

```bash
# 기본 타겟(dev)으로 실행
dbt run

# prod 타겟으로 실행
dbt run --target prod
```

### 주의사항

- Snowflake 네이티브 dbt에서는 `password`, `authenticator`, `env_var()` 사용 불가
- 인증은 Snowflake 세션에서 자동 처리됨
- `target` 스키마는 dbt 실행 전에 미리 생성되어 있어야 함

### Snowflake 네이티브 dbt 인증 방식

#### 왜 `account: 'not needed'`, `user: 'not needed'`인가?

Snowflake 네이티브 dbt는 **Snowflake Workspace 내부**에서 실행됩니다. 이미 Snowsight에 로그인한 상태이므로:

1. 사용자 인증이 완료된 세션 안에서 dbt가 동작
2. 별도의 계정/사용자/비밀번호 정보가 불필요
3. `profiles.yml`의 YAML 구문 요구사항을 충족하기 위해 placeholder 값을 넣어둔 것

#### 네이티브 dbt vs 로컬 dbt 비교

| 항목 | 네이티브 dbt (Workspace) | 로컬 dbt (PC) |
|------|--------------------------|---------------|
| 인증 방식 | Snowflake 세션 자동 사용 | account/user/password 직접 입력 |
| `account` | 불필요 (`'not needed'`) | 필수 (예: `vhb13071`) |
| `user` | 불필요 (`'not needed'`) | 필수 (예: `YEJUNING`) |
| `password` | 사용 불가 | 필수 또는 SSO/키페어 사용 |
| `env_var()` | 사용 불가 | 사용 가능 |
| 외부 패키지 | External Access Integration 필요 | 자유롭게 다운로드 |

#### 로컬 환경에서의 profiles.yml 예시

로컬 PC에서 dbt를 실행할 경우에는 실제 인증 정보가 필요합니다:

```yaml
tasty_bytes:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: vhb13071
      user: YEJUNING
      password: '{{ env_var("SNOWFLAKE_PASSWORD") }}'
      role: accountadmin
      database: tasty_bytes_dbt_db
      schema: dev
      warehouse: tasty_bytes_dbt_wh
      threads: 8
```
