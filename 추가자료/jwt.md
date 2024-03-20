## 1. JWT 란?

> JSON Web Token 으로 JSON 객체를 암호화하여 안전하게 정보를 주고받을 수 있도록 설계된 토큰입니다.


세션 로그인 방식과 다르게 토큰 자체가 필요한 모든 정보를 포함하고 있어 별도의 세션 저장소가 필요 없습니다.

<br/>

## 2. JWT의 구성

JWT는 Header(헤더) Payload(내용) Signature(서명) 세 부분으로 구성됩니다.

![HEADER   PAYLOAD   SIGNATURE](https://github.com/NamJongtae/nextjs-study/assets/113427991/46cca232-93e5-422b-a529-cd043e9a4b32)


위와 같이 Header, Payload, Signature  ( . ) 으로 구분되어 JWT를 구성합니다.

예시 JWT

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

<br/>

### 1 ) Header : Algorithm, Token Type

헤더는 토큰의 타입(JWT)과 사용된 알고리즘(HS256, RS256 등)을 정의합니다.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

<br/>

### 2 ) Payload : JSON Data

페이로드는 실제로 전달하고자 하는 정보(claim)를 포함합니다. claim은 토큰에 대한 속성 값들이며, claim의 종류에는 registred, public, private가 있습니다.

registred : 토큰에 대한 정보를 제공하는 claim으로 예를 들어 발행자(iss), 만료 시간(exp), 주체(sub), 대상자( aud) 등이 있습니다.

public : 사용자가 자유롭게 정의할 수 있으며, 충돌이 없도록 URI 형태로 이름을 정의 할 수 있습니다.

private : 두 당사자 간에 협의하에 사용되는 claim으로 정보를 공유하기 위해 만들어진 커스텀 claim으로 사용자나 ID, 이메일 등이 될 수 있습니다.

```json
{
  "sub": "1234567890", 
  "name": "John Doe",
  "admin": true
}
```

<br/>

### 3 )  Signature : Verify

서명은 헤더의 Base64Url 인코딩 값, 페이로드의 Base64Url 인코딩 값, 비밀키를 사용한 해시를 조합하여 생성됩니다. 이를 통해 메시지의 무결성과 인증이 보장됩니다.

```javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

<br/>

## 3. JWT의 사용방식

### **1 ) 인증(Authentication)**

사용자가 로그인을 하면 서버는 사용자의 인증 정보를 검증한 후 JWT를 생성하여 반환합니다. 사용자는 이후의 요청에서 이 토큰을 Authorization 헤더에 포함시켜 서버에 전송합니다. 서버는 토큰을 검증하고 요청을 처리합니다.

### **2 ) 정보 교환(Information Exchange)**

JWT는 안전하게 정보를 교환하기에 적합합니다. 토큰이 서명되어 있기 때문에 정보가 중간에 변경되었는지 확인할 수 있습니다.

<br/>

## 4. JWT 장점 단점

### 1 ) 장점

- JWT는 필요한 모든 정보를 자체적으로 가지고 있어, 별도의 저장소에 접근할 필요 없이 토큰 하나로 인증 및 정보 교환을 처리할 수 있습니다.
- 세션 상태를 서버에 저장할 필요가 없기 때문에, 서버의 부하를 줄이고 시스템의 확장성을 높일 수 있습니다.
- 토큰 기반으로 다른 로그인 시스템에 접근 및 권한 공유가 가능합니다.(OAuth)
- 디지털 서명이 되어 있기 때문에 정보의 변조를 방지할 수 있으며, https를 통해 추가적인 보안을 제공할 수 있습니다.

### 2 ) 단점

- JWT가 탈취되면, 탈취자는 토큰의 유효기간 동안 사용자의 권한을 가질 수 있습니다. 이를 방지하기 위한 추가적인 보안 조치가 필요합니다.
- JWT는 데이터를 포함한다는 장점이 있지만, 이로 인해 HTTP 헤더에 추가되는 토큰의 크기가 커질 수 있으며, 이는 네트워크 대역폭에 영향을 줄 수 있습니다.
- JWT는 Payload 자체는 암호화가 되지 않아 중요한 정보는 담을 수 없습니다.
- JWT는 발급된 후에 서버 측에서 무효화시킬 수 있는 방법이 제한적입니다.
- 클라이언트 측에서 JWT를 저장할 때, 쿠키나 로컬 스토리지 등이 사용되는데, 이러한 저장소가 XSS 공격에 노출될 경우 토큰이 탈취될 수 있습니다.

<br/>

## 5. AccessToken & RefreshToken

JWT를 사용할 때 `accessToken`과 `RefreshToken`은 인증 시스템에서 중요한 역할을 합니다. 이 두 토큰은 사용자 인증과 관련된 데이터를 안전하게 전송하기 위해 사용됩니다.

### 1 ) AccessToken

API를 사용하기 위한 인증 토큰으로 사용자가 누구인지 확인하고, 해당 사용자가 요청한 작업을 수행할 권한이 있는지 검증하는 데 사용됩니다. 

일반적으로 짧은 유효시간을 갖습니다. 이는 보안상의 이유로, `accessToken`이 탈취되더라도 유효 기간이 짧아 잠재적인 피해를 최소화하기 위함입니다.

`accessToken` 은 클라이언트 측에서 보관되며, 사용자가 서버에 요청을 보낼 때마다 HTTP 헤더에 포함되어 서버로 전송됩니다.

### 2 ) RefreshToken

`RefreshToken`은 `accessToken`의 유효 기간이 만료되었을 때, 사용자가 서버에 재인증을 요청하지 않고도 새로운 `accessToken`을 받을 수 있게 해주는 역할을 합니다. 

`RefreshToken`은 `accessToken`보다 훨씬 긴 유효 기간을 가집니다. 하지만 `RefreshToken`이 탈취되는 것은 큰 보안 위험이 될 수 있으므로, 안전하게 보관해야 합니다.

 일반적으로 `accessToken`과 함께 클라이언트 측에 저장되지만, 보안에 더 민감한 애플리케이션에서는 서버 측에서 `RefreshToken`을 관리하기도 합니다.

<br/>

## 6. JWT **Authentication Flow**

사용자가 로그인을 시도하면 서버는 사용자를 인증하고, 인증이 성공하면 `accessToken`과 `refreshToken`을 발급합니다. 발급된 토큰은 클라이언트에게 쿠키나 데이터로 전송됩니다.

사용자는 서버에 요청을 보낼 때 `accessToken`을 헤더에 실어 보냅니다.

서버는 `accessToken`를 검증하여 유효한 토큰인지를 파악합니다. 

유효하지 않은 `accessToken` 이라면 클라이언트에게 `accessToken` 이 유효하지 않다는 에러를 보냅니다.

클라이언트는 에러 메세지를 받으면 `refreshToken` 를 서버에 전송합니다.

서버는 `refreshToken` 을 받아 유효한 토큰인지 파악하고, 유효한 토큰이라면 새로운 `accessToken` 를 발급하여 클라이언트에게 전송합니다. 만약 `refreshToken` 이 유효하지 않다면 현재 계정을 로그아웃 시킵니다.

![JWT_Flow](https://github.com/NamJongtae/nextjs-study/assets/113427991/afabcdd0-654a-4fca-9e99-f9c5a3534824)