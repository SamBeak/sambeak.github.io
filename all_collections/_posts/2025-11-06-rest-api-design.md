---
layout: post
title: 🌐 REST API 설계 원칙과 Best Practice 완벽 가이드
date: 2025-11-06
categories:
  [
    "SamBeak",
    "REST",
    "API",
    "HTTP",
    "Web",
    "Backend",
  ]
---

# REST API란 무엇인가

<br>
웹 서비스를 만들 때 가장 먼저 고민하는 것은 <br>
"클라이언트와 서버가 어떻게 대화할까?"이다. <br><br>

REST API는 이 대화의 규칙이다. <br>
마치 우체국에서 편지를 보낼 때 <br>
정해진 형식(주소, 우표, 내용)이 있듯이, <br>
REST API도 정해진 규칙으로 데이터를 주고받는다. <br><br>

REST(Representational State Transfer)는 <br>
2000년 로이 필딩(Roy Fielding)이 제안한 <br>
웹의 장점을 최대한 활용하는 아키텍처 스타일이다. <br><br>

> ## 왜 REST API를 배워야 할까?

<br>

**이유 1: 산업 표준** <br>
대부분의 웹 서비스가 REST API를 사용한다. <br><br>

**이유 2: 단순함** <br>
HTTP를 그대로 활용하므로 별도 학습 비용이 적다. <br><br>

**이유 3: 명확성** <br>
URI만 봐도 어떤 리소스인지 알 수 있다. <br><br>

**이유 4: 면접 필수** <br>
REST API 설계는 백엔드 면접의 단골 질문이다. <br><br>

# 기본 개념 요약

<br>

## 🏷️ REST 6가지 제약 조건

<br>

### 1. Client-Server (클라이언트-서버)

<br>
**개념**: 클라이언트와 서버의 역할을 명확히 분리 <br><br>

**식당 비유**: <br>
손님(클라이언트)은 주문만 하고, <br>
주방(서버)은 요리만 한다. <br>
각자의 역할에만 집중한다. <br><br>

### 2. Stateless (무상태)

<br>
**개념**: 서버는 클라이언트의 상태를 저장하지 않음 <br><br>

**우체국 비유**: <br>
우체부는 매번 주소를 확인하고 배달한다. <br>
이전에 어디로 배달했는지 기억하지 않는다. <br><br>

### 3. Cacheable (캐시 가능)

<br>
**개념**: HTTP의 캐시 기능을 그대로 활용 <br><br>

### 4. Uniform Interface (일관된 인터페이스)

<br>
**개념**: 리소스 식별, HTTP 메서드 사용 <br><br>

### 5. Layered System (계층화)

<br>
**개념**: 중간에 프록시, 게이트웨이 추가 가능 <br><br>

### 6. Code-On-Demand (선택사항)

<br>
**개념**: 서버가 클라이언트에 실행 코드 전송 가능 <br><br>

## 🏷️ HTTP 메서드

<br>

| 메서드 | 의미 | 멱등성 | 안전성 | 사용 예시 |
|--------|------|--------|--------|-----------|
| **GET** | 조회 | O | O | 게시글 목록 조회 |
| **POST** | 생성 | X | X | 게시글 작성 |
| **PUT** | 전체 수정 | O | X | 게시글 전체 수정 |
| **PATCH** | 부분 수정 | X | X | 게시글 제목만 수정 |
| **DELETE** | 삭제 | O | X | 게시글 삭제 |

<br>

**멱등성**: 동일한 요청을 여러 번 해도 결과가 같음 <br>
**안전성**: 서버 상태를 변경하지 않음 <br><br>

## 🏷️ HTTP 상태 코드

<br>

### 2xx - 성공

<br>

- **200 OK**: 요청 성공
- **201 Created**: 생성 성공
- **204 No Content**: 성공 (응답 Body 없음)

<br>

### 4xx - 클라이언트 오류

<br>

- **400 Bad Request**: 잘못된 요청
- **401 Unauthorized**: 인증 필요
- **403 Forbidden**: 권한 없음
- **404 Not Found**: 리소스 없음
- **409 Conflict**: 충돌
- **422 Unprocessable Entity**: 처리 불가
- **429 Too Many Requests**: 요청 과다

<br>

### 5xx - 서버 오류

<br>

- **500 Internal Server Error**: 서버 오류
- **502 Bad Gateway**: 게이트웨이 오류
- **503 Service Unavailable**: 서비스 불가

<br>

# URI 설계 원칙

<br>

## 🏷️ Best Practices

<br>

### 1. 명사 사용, 동사 금지

<br>

```
✅ Good:
GET    /api/users
POST   /api/users
GET    /api/users/123
DELETE /api/users/123

❌ Bad:
/api/getUsers
/api/createUser
/api/deleteUser
```

<br>

### 2. 복수형 사용

<br>

```
✅ Good: /api/users
❌ Bad: /api/user
```

<br>

### 3. 계층 구조 표현

<br>

```
✅ Good:
GET /api/users/123/posts
GET /api/posts/456/comments

❌ Bad:
GET /api/posts?userId=123
```

<br>

### 4. 소문자 사용, 하이픈으로 구분

<br>

```
✅ Good: /api/user-profiles
❌ Bad: /api/UserProfiles
```

<br>

# 실전 예시

<br>

## 🏷️ Spring Boot API 구현

<br>

```java
@RestController
@RequestMapping("/api/posts")
public class PostController {
    
    @Autowired
    private PostService postService;
    
    // 게시글 목록 조회 (페이징, 필터링, 정렬)
    @GetMapping
    public ResponseEntity<Page<PostResponse>> getPosts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(required = false) String status,
        @RequestParam(defaultValue = "createdAt:desc") String sort
    ) {
        Pageable pageable = PageRequest.of(page, size);
        Page<PostResponse> posts = postService.getPosts(status, pageable);
        return ResponseEntity.ok(posts);
    }
    
    // 게시글 단건 조회
    @GetMapping("/{id}")
    public ResponseEntity<PostResponse> getPost(@PathVariable Long id) {
        PostResponse post = postService.getPost(id);
        return ResponseEntity.ok(post);
    }
    
    // 게시글 작성
    @PostMapping
    public ResponseEntity<PostResponse> createPost(
        @Valid @RequestBody PostCreateRequest request
    ) {
        PostResponse post = postService.createPost(request);
        
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(post.getId())
            .toUri();
        
        return ResponseEntity.created(location).body(post);
    }
    
    // 게시글 수정
    @PutMapping("/{id}")
    public ResponseEntity<PostResponse> updatePost(
        @PathVariable Long id,
        @Valid @RequestBody PostUpdateRequest request
    ) {
        PostResponse post = postService.updatePost(id, request);
        return ResponseEntity.ok(post);
    }
    
    // 게시글 삭제
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletePost(@PathVariable Long id) {
        postService.deletePost(id);
        return ResponseEntity.noContent().build();
    }
}
```

<br>

## 🏷️ 에러 응답 표준화

<br>

```java
@Getter
@Builder
public class ErrorResponse {
    private String timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
}

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
        ResourceNotFoundException ex,
        HttpServletRequest request
    ) {
        ErrorResponse error = ErrorResponse.builder()
            .timestamp(LocalDateTime.now().toString())
            .status(404)
            .error("Not Found")
            .message(ex.getMessage())
            .path(request.getRequestURI())
            .build();
        
        return ResponseEntity.status(404).body(error);
    }
}
```

<br>

## 🏷️ API 버저닝

<br>

### URI 버저닝 (추천)

<br>

```
GET /api/v1/users
GET /api/v2/users
```

<br>

**장점**: 명확하고 캐시 가능 <br>
**단점**: URI가 변경됨 <br><br>

### Header 버저닝

<br>

```http
GET /api/users
Accept: application/vnd.company.v1+json
```

<br>

## 🏷️ HATEOAS

<br>

```json
{
  "id": 123,
  "title": "REST API 설계",
  "_links": {
    "self": {
      "href": "/api/posts/123"
    },
    "comments": {
      "href": "/api/posts/123/comments"
    },
    "author": {
      "href": "/api/users/456"
    }
  }
}
```

<br>

# 실전 체크리스트

<br>

## ✅ URI 설계

<br>

- [ ] 명사 사용 (동사 금지)
- [ ] 복수형 사용
- [ ] 소문자, 하이픈 사용
- [ ] 계층 구조 표현
- [ ] 파일 확장자 제거

<br>

## ✅ HTTP 메서드

<br>

- [ ] GET: 조회 (멱등성 O, 안전성 O)
- [ ] POST: 생성 (멱등성 X)
- [ ] PUT: 전체 수정 (멱등성 O)
- [ ] PATCH: 부분 수정
- [ ] DELETE: 삭제 (멱등성 O)

<br>

## ✅ 상태 코드

<br>

- [ ] 2xx: 성공 (200, 201, 204)
- [ ] 4xx: 클라이언트 오류 (400, 401, 403, 404)
- [ ] 5xx: 서버 오류 (500, 502, 503)

<br>

## ✅ 응답 형식

<br>

- [ ] 일관된 JSON 구조
- [ ] 에러 응답 표준화
- [ ] 페이징 정보 포함
- [ ] HATEOAS 링크 (선택)

<br>

## ✅ 보안

<br>

- [ ] HTTPS 사용
- [ ] 인증/인가 (JWT, OAuth)
- [ ] Rate Limiting
- [ ] CORS 설정
- [ ] 입력 검증

<br>

## ✅ 문서화

<br>

- [ ] Swagger/OpenAPI
- [ ] 예시 코드
- [ ] 에러 코드 목록
- [ ] 변경 이력

<br>

# 요약

<br>
REST API 설계는 일관성과 직관성이 핵심이다. <br><br>

**💎 핵심 포인트**:

1. **리소스 중심**: URI는 명사로, 동사는 HTTP 메서드로
2. **무상태**: 매 요청에 필요한 정보 포함
3. **HTTP 활용**: 메서드와 상태 코드를 올바르게 사용
4. **일관성**: 네이밍 규칙을 프로젝트 전체에 적용
5. **계층 구조**: 리소스 간 관계를 URI로 표현
6. **에러 처리**: 명확한 에러 메시지와 상태 코드

<br>

**🎯 설계 원칙**:

```
✅ Do:
- GET /api/users (목록 조회)
- POST /api/users (생성)
- GET /api/users/123 (단건 조회)
- PUT /api/users/123 (수정)
- DELETE /api/users/123 (삭제)

❌ Don't:
- POST /api/getUserList
- GET /api/createUser
- GET /api/user/delete?id=123
```

<br>

**📌 상태 코드 가이드**:

- **200**: GET, PUT 성공
- **201**: POST 성공 (Location 헤더 포함)
- **204**: DELETE 성공
- **400**: 잘못된 요청
- **401**: 인증 필요
- **403**: 권한 없음
- **404**: 리소스 없음
- **500**: 서버 오류

<br>

**🚀 Best Practices**:

1. 버저닝은 URI에 명시 (/api/v1/users)
2. 페이징은 쿼리 파라미터 (?page=1&size=20)
3. 필터링도 쿼리 파라미터 (?status=active)
4. 정렬은 sort 파라미터 (?sort=createdAt:desc)
5. 에러 응답은 표준 형식 유지
6. HATEOAS로 다음 행동 안내 (선택)

<br>

REST API는 단순히 HTTP를 사용하는 것이 아니라, <br>
웹의 아키텍처를 최대한 활용하는 설계 철학이다. <br>
명확하고 일관된 API는 개발자 경험을 향상시키고, <br>
유지보수 비용을 크게 줄여준다. <br><br>
