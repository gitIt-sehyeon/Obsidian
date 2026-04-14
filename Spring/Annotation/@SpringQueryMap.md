# @SpringQueryMap

## 개요

Feign에서 **객체를 GET 쿼리 파라미터로 변환**해주는 어노테이션

---

## 문제 상황

파라미터가 많아지면 `@RequestParam`이 너무 길어짐

```java
// ❌ 파라미터가 많으면 지저분함
@GetMapping("/api/places/search")
PlacesResponse searchPlace(
    @RequestParam String query,
    @RequestParam String languageCode,
    @RequestParam Integer maxResults,
    @RequestParam String type
);
```

---

## @SpringQueryMap 사용

```java
// 요청 객체 만들기
public record PlaceSearchRequest(
    String query,
    String languageCode,
    Integer maxResults,
    String type
) {}

// Feign 인터페이스
@GetMapping("/api/places/search")
PlacesResponse searchPlace(@SpringQueryMap PlaceSearchRequest request);
```

실제 요청은 이렇게 변환됨

```
GET /api/places/search?query=롯데호텔&languageCode=ko&maxResults=5&type=hotel
```

---

## @SpringQueryMap 없이 객체만 쓰면 안되는 이유

Feign은 어노테이션 없이 객체를 넘기면 **Request Body**로 인식함

```java
// ❌ 어노테이션 없이 객체만 넘기면
@GetMapping("/api/places/search")
PlacesResponse searchPlace(PlaceSearchRequest request);
// GET인데 body로 넘겨버림 → 서버에서 못 받음

// ✅ @SpringQueryMap 붙여야 쿼리 파라미터로 변환됨
@GetMapping("/api/places/search")
PlacesResponse searchPlace(@SpringQueryMap PlaceSearchRequest request);
// GET /api/places/search?query=롯데호텔&languageCode=ko
```

---

## 요청 방식별 정리

| 요청 방식         | 파라미터 전달 방법                 |
| ------------- | -------------------------- |
| GET + 단일 파라미터 | `@RequestParam`            |
| GET + 여러 파라미터 | `@SpringQueryMap` + record |
| POST          | `@RequestBody` + 객체        |

---

## 정리

> GET 쿼리 파라미터가 3개 이상이면 `@SpringQueryMap` + record로 묶는 게 훨씬 깔끔함 POST + `@RequestBody`면 그냥 객체 넘겨도 되지만, GET은 꼭 `@SpringQueryMap` 붙여야 함