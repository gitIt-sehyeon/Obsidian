# Spring @Profile 어노테이션

## 개요

특정 환경(프로파일)에서만 Bean을 활성화하도록 조건을 거는 어노테이션

---

## prod 환경이란?

**Production(운영) 환경** — 실제 사용자가 사용하는 라이브 서버 환경

|환경|설명|
|---|---|
|`dev`|개발자 로컬 개발 환경|
|`test`|테스트/QA 환경|
|`prod`|실제 서비스 운영 환경|

---

## 기본 사용법

```java
@Component
@Profile("dev")
public class DevDataSource implements DataSource { ... }

@Component
@Profile("prod")
public class ProdDataSource implements DataSource { ... }
```

활성 프로파일에 맞는 Bean만 Spring 컨테이너에 등록됨

---

## @Profile 표현식

|표현식|설명|
|---|---|
|`@Profile("prod")`|prod 프로파일일 때만 활성화|
|`@Profile("!prod")`|prod가 아닐 때 활성화|
|`@Profile({"dev", "test"})`|dev 또는 test일 때 활성화|
|`@Profile("prod & local")`|prod AND local 둘 다일 때 활성화 (Spring 5.1+)|

---

## @Configuration 클래스에 적용

```java
@Configuration
@Profile("prod")
public class ProdConfig {

    @Bean
    public DataSource dataSource() {
        // 운영 DB 설정
        return new HikariDataSource();
    }
}
```

---

## application-{profile}.yml 분리 패턴

```
resources/
├── application.yml          # 공통 설정
├── application-dev.yml      # 개발 환경
├── application-prod.yml     # 운영 환경
└── application-test.yml     # 테스트 환경
```

> `application.yml`은 **공통 설정**, `application-{profile}.yml`은 **환경별 덮어쓰기** 개념 prod로 실행하면 `application-prod.yml`이 공통 설정 위에 덮어씌워짐

### application.yml (공통)

```yaml
spring:
  profiles:
    active: dev  # 기본값 (여기서 바꾸면 됨)
```

### application-dev.yml

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb   # 인메모리 DB
  jpa:
    show-sql: true             # SQL 로그 출력
    ddl-auto: create-drop      # 앱 시작/종료 시 테이블 자동 생성/삭제
```

### application-prod.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://prod-server/mydb  # 실제 DB
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  jpa:
    show-sql: false                      # SQL 로그 숨김
    ddl-auto: validate                   # 테이블 건드리지 않음
```

---

## 프로파일 활성화 방법 (dev / prod 선택)

```
active=dev   →   application.yml + application-dev.yml 적용
active=prod  →   application.yml + application-prod.yml 적용
```

### 1. application.yml에서 기본값 지정

```yaml
spring:
  profiles:
    active: dev  # dev 또는 prod로 변경
```

### 2. IDE에서 실행 (IntelliJ)

Run Configuration → Active profiles 입력란에 `dev` 또는 `prod` 입력

### 3. 빌드 후 실행 시 직접 지정

```bash
# dev로 실행
java -jar app.jar --spring.profiles.active=dev

# prod로 실행
java -jar app.jar --spring.profiles.active=prod
```

### 4. 환경 변수로 지정

```bash
export SPRING_PROFILES_ACTIVE=prod
```

---

## prod 환경 주요 설정 포인트

- `show-sql: false` → SQL 로그 비활성화
- `ddl-auto: validate` → 스키마 자동 변경 금지 (`none` 또는 `validate` 권장)
- DB 계정 정보는 환경 변수로 주입 (`${DB_PASSWORD}`)
- 로그 레벨을 `WARN` 또는 `ERROR`로 설정
- Actuator endpoint 노출 최소화

---

## 주의사항

- `@Profile`이 없는 Bean은 **모든 환경에서 항상 활성화**
- 프로파일 미설정 시 `default` 프로파일이 적용됨
- `spring.profiles.active`와 `spring.profiles.include` 혼용 가능