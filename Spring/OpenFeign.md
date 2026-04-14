## 개요

**OpenFeign**은 Spring Cloud에서 제공하는 HTTP 클라이언트로, 인터페이스와 어노테이션만으로 서버 간 HTTP 통신을 구현할 수 있게 해줍니다.

RestTemplate이나 WebClient처럼 직접 HTTP 요청을 작성하는 대신, **마치 로컬 메서드를 호출하듯** 다른 서버의 API를 호출할 수 있습니다.

---

## WebClient vs OpenFeign 비교

![[Pasted image 20260414230053.png]]

---

## 의존성 추가

```gradle
// build.gradle
implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
```

```gradle
// Spring Cloud BOM도 필요
dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:2023.0.0"
    }
}
```

---

## 기본 사용법

### 1. @EnableFeignClients 활성화

```java
@SpringBootApplication
@EnableFeignClients
public class TravelCoreApplication {
    public static void main(String[] args) {
        SpringApplication.run(TravelCoreApplication.class, args);
    }
}
```

### 2. Feign 인터페이스 선언

```java
// core 서버에 작성
@FeignClient(name = "supply-service", url = "${sofly.supply.url}")
public interface PlaceClient {

    @GetMapping("/api/places/search")
    PlacesResponse searchPlace(@RequestParam String query);
}
```

### 3. 그냥 주입해서 사용

```java
@Service
public class TravelPlaceTool {

    private final PlaceClient placeClient;

    public TravelPlaceTool(PlaceClient placeClient) {
        this.placeClient = placeClient;
    }

    @Tool(description = "장소 이름으로 Google Places 정보를 검색한다")
    public PlacesResponse searchPlace(String placeName) {
        return placeClient.searchPlace(placeName);  // 그냥 메서드 호출
    }
}
```

---

## WebClient로 짰을 때 vs Feign 비교

### WebClient

```java
public PlacesResponse searchPlace(String placeName) {
    return webClient.get()
            .uri("/api/places/search?query={query}", placeName)
            .retrieve()
            .bodyToMono(PlacesResponse.class)
            .block();
}
```

### Feign

```java
@GetMapping("/api/places/search")
PlacesResponse searchPlace(@RequestParam String query);
```

훨씬 간결합니다.

---

## 주요 어노테이션

```java
@FeignClient(name = "supply-service", url = "${sofly.supply.url}")
public interface PlaceClient {

    // GET 요청
    @GetMapping("/api/places/{id}")
    PlacesResponse getPlace(@PathVariable Long id);

    // GET + 쿼리 파라미터
    @GetMapping("/api/places/search")
    PlacesResponse searchPlace(@RequestParam String query);

    // POST 요청
    @PostMapping("/api/places")
    PlacesResponse createPlace(@RequestBody PlaceRequest request);

    // DELETE 요청
    @DeleteMapping("/api/places/{id}")
    void deletePlace(@PathVariable Long id);

    // 헤더 추가
    @GetMapping("/api/places/search")
    PlacesResponse searchWithHeader(
        @RequestHeader("Authorization") String token,
        @RequestParam String query
    );
}
```

---

## application.yml 설정

```yaml
sofly:
  supply:
    url: ${SUPPLY_SERVICE_URL:http://supply:8081}
```

---

## 에러 처리 (ErrorDecoder)

Feign은 기본적으로 4xx, 5xx 응답이 오면 `FeignException`을 던집니다. 커스텀 처리가 필요하면 `ErrorDecoder`를 구현합니다.

```java
@Component
public class PlaceClientErrorDecoder implements ErrorDecoder {

    @Override
    public Exception decode(String methodKey, Response response) {
        return switch (response.status()) {
            case 404 -> new PlaceNotFoundException("장소를 찾을 수 없습니다");
            case 500 -> new PlaceServiceException("Places 서비스 오류");
            default -> new Default().decode(methodKey, response);
        };
    }
}
```

---

## 타임아웃 설정

```yaml
spring:
  cloud:
    openfeign:
      client:
        config:
          supply-service:        # @FeignClient name과 일치
            connect-timeout: 3000   # 3초
            read-timeout: 5000      # 5초
```

---

## 현재 프로젝트에 적용하면

### 기존 WebClient 방식

```java
// PlaceServiceClient.java - 코드가 길다
public PlacesResponse searchPlace(String placeName) {
    return webClient.get()
            .uri("/api/places/search?query={query}", placeName)
            .retrieve()
            .bodyToMono(PlacesResponse.class)
            .block();
}
```

### Feign으로 교체

```java
// PlaceClient.java - 인터페이스만 선언
@FeignClient(name = "supply-service", url = "${sofly.supply.url}")
public interface PlaceClient {

    @GetMapping("/api/places/search")
    Optional<PlacesResponse> searchPlace(@RequestParam String query);
}
```

---

## 주의사항

- `@EnableFeignClients`를 Main 클래스에 꼭 추가해야 합니다.
- `@FeignClient`의 `name`은 고유해야 합니다.
- 기본적으로 **동기 방식**이라 대용량 트래픽에는 WebClient가 더 적합합니다.
- Spring Cloud 버전과 Spring Boot 버전 호환성을 꼭 확인해야 합니다.