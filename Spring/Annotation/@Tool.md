# Spring AI @Tool 어노테이션

## 개요

`@Tool`은 **Spring AI**에서 제공하는 어노테이션으로, AI 모델(Claude, GPT 등)이 Java 메서드를 직접 호출할 수 있게 해주는 기능입니다.

AI가 대화 중 "이 정보가 필요하다"고 판단하면 자동으로 해당 메서드를 실행하고, 결과를 답변에 활용합니다.

---

## 동작 원리

```
사용자 → "롯데호텔 정보 알려줘"
           ↓
         AI 모델
           ↓  "장소 검색이 필요하다" 판단
        @Tool 메서드 자동 호출
           ↓
        searchPlace("롯데호텔")
           ↓
        Google Places API 호출
           ↓
        결과를 AI가 받아서 답변 생성
           ↓
사용자 ← "롯데호텔은 서울 중구에 위치한 5성급 호텔로..."
```

---

## 기본 사용법

### 1. 의존성 추가

```gradle
// build.gradle
implementation 'org.springframework.ai:spring-ai-openai-spring-boot-starter'
// 또는
implementation 'org.springframework.ai:spring-ai-anthropic-spring-boot-starter'
```

### 2. @Tool 메서드 작성

```java
@Service
public class PlaceTool {

    private final GooglePlacesClient googlePlacesClient;

    public PlaceTool(GooglePlacesClient googlePlacesClient) {
        this.googlePlacesClient = googlePlacesClient;
    }

    @Tool(description = "장소 이름으로 Google Places 정보를 검색한다")
    public PlacesResponse searchPlace(String placeName) {
        return googlePlacesClient.searchText(placeName)
                .orElse(null);
    }
}
```

> `description`이 핵심입니다. AI는 이 설명을 읽고 언제 이 메서드를 호출할지 판단합니다.

### 3. AI 호출 시 Tool 등록

```java
@Service
public class TravelChatService {

    private final ChatClient chatClient;
    private final PlaceTool placeTool;

    public TravelChatService(ChatClient.Builder builder, PlaceTool placeTool) {
        this.chatClient = builder.build();
        this.placeTool = placeTool;
    }

    public String chat(String userMessage) {
        return chatClient.prompt()
                .user(userMessage)
                .tools(placeTool)       // Tool 등록
                .call()
                .content();
    }
}
```

---

## 여러 Tool 등록

```java
@Service
public class WeatherTool {

    @Tool(description = "도시 이름으로 현재 날씨를 조회한다")
    public String getWeather(String city) {
        // 날씨 API 호출
        return weatherClient.getWeather(city);
    }
}

@Service
public class FlightTool {

    @Tool(description = "출발지와 목적지로 항공편을 검색한다")
    public List<Flight> searchFlights(String origin, String destination) {
        // 항공편 API 호출
        return flightClient.search(origin, destination);
    }
}

// 여러 Tool 동시 등록
chatClient.prompt()
        .user("서울에서 도쿄 여행 계획 짜줘")
        .tools(placeTool, weatherTool, flightTool)  // 동시 등록
        .call()
        .content();
```

---

## @Tool description 잘 쓰는 법

AI가 메서드를 호출할지 말지는 `description`으로 결정합니다.

```java
// ❌ 나쁜 예 - 너무 모호함
@Tool(description = "장소 검색")
public PlacesResponse search(String query) { ... }

// ✅ 좋은 예 - 언제 써야 하는지 명확함
@Tool(description = "사용자가 특정 장소, 호텔, 음식점, 관광지에 대한 정보를 요청할 때 "
                  + "장소 이름으로 위치, 평점, 주소, 영업 여부를 검색한다")
public PlacesResponse searchPlace(String placeName) { ... }
```

---

## 파라미터에 설명 추가

```java
@Tool(description = "장소 이름으로 Google Places 정보를 검색한다")
public PlacesResponse searchPlace(
        @ToolParam(description = "검색할 장소의 이름 (예: 롯데호텔 서울, 경복궁)") String placeName
) {
    return googlePlacesClient.searchText(placeName).orElse(null);
}
```

---

## 현재 프로젝트에 적용하면

```java
@Service
public class TravelPlaceTool {

    private final GooglePlacesClient googlePlacesClient;

    public TravelPlaceTool(GooglePlacesClient googlePlacesClient) {
        this.googlePlacesClient = googlePlacesClient;
    }

    @Tool(description = "여행지, 호텔, 음식점, 관광지 이름으로 상세 정보를 검색한다. "
                      + "위치, 주소, 평점, 영업시간, 사진 정보를 반환한다.")
    public PlacesResponse searchPlace(
            @ToolParam(description = "검색할 장소 이름") String placeName
    ) {
        return googlePlacesClient.searchText(placeName).orElse(null);
    }
}
```

AI한테 "서울 5성급 호텔 추천해줘" 라고 하면 → AI가 알아서 `searchPlace("롯데호텔 서울")` 등을 호출해서 실시간 데이터 기반으로 답변합니다.

---

## @Tool vs 직접 API 호출 비교

||직접 호출|@Tool|
|---|---|---|
|호출 시점|개발자가 코드에서 직접|AI가 필요하다고 판단할 때 자동|
|유연성|고정된 로직|대화 흐름에 따라 동적|
|코드 복잡도|단순|약간 복잡|
|AI 활용도|낮음|높음|
|적합한 상황|단순 CRUD|AI 챗봇, 추천 시스템|

---

## 주의사항

- `description`을 구체적으로 작성할수록 AI가 더 정확하게 호출합니다.
- 반환 타입이 복잡할수록 AI가 결과를 파싱하기 어려울 수 있으니 필요한 정보만 담은 DTO로 반환하는 게 좋습니다.
- Tool이 너무 많으면 AI가 어떤 Tool을 써야 할지 혼란스러워할 수 있으니 5~10개 이내로 유지하는 게 좋습니다.
- Tool 내부에서 예외가 발생하면 AI가 오류 메시지를 받아 처리하므로, 예외 처리를 꼼꼼히 해야 합니다.