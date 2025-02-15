# OAuth (RefreshToken - Cookie & AccessToken - Cookie)

## 학습 기대 효과

이 기능은 사용자가 OAuth 프로세스와 토큰 기반의 인증 방식을 학습할 수 있도록 설계되었습니다.

OAuth 인증 흐름을 통해 사용자가 인증 과정을 이해하고, Access Token과 Refresh Token을 사용한 인증 및 재발급 과정을 학습하며, 토큰 인증 방식을 학습할 수 있습니다.

- **Access Token**: API 요청 시 권한을 부여하는 토큰
- **Refresh Token**: Access Token이 만료되었을 때 새로 발급받기 위한 토큰

## 사용방법

OAuth 구현에 앞서 [**[카카오 로그인 공식문서]**](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api) 에서 OAuth의 흐름을 익혀주세요.

1. 카카오로부터 Authorization Code를 발급받아 토큰을 교환합니다.
2. 서버에서 유저 정보를 데이터베이스에 저장하고, 자체적인 토큰을 발급받아 클라이언트로 전송합니다.
3. Access Token을 사용해 유저 정보와 관련된 API 요청을 수행합니다.
4. Access Token을 Refresh Token으로 재발급하는 요청을 할 수 있습니다.
5. 로그아웃을 통해 서버에서 Refresh Token을 무효화할 수 있습니다.

## 주의 사항

이 사이트에서 `쿠키`를 사용하는 경우, https 통신에서만 가능하기 때문에 배포 Url을 통해서만 통신할 수 있습니다.

또한 다음 설정을 해야합니다.

**[쿠키 설정]**

- httponly
- samesite: none
- secure: true

**[CORS 설정]**

- Access-Control-Allow-Origin: `https://ssafysandbox.vercel.app`
- Access-Control-Allow-Credentials: `true`

## POST /oauth/cookie/auth

KAKAO Auth Code를 통해 카카오로부터 Access Token을 발급받습니다.
이 Access Token을 사용하여 사용자 정보를 조회한 후, 해당 정보를 데이터베이스에 저장합니다.

이후, 서비스 자체적으로 관리할 Access Token과 Refresh Token을 발급하여 클라이언트에 전달합니다.

#### **환경변수값**

- client_id : 0da56a0700a56821782b91c49ce03b42
- redirect_uri: https://ssafysandbox.vercel.app/oauth/redirect
- response_type: code

### Request

```json
// Request Body

{
  "code": "kakao_auth_code_value"
}
```

### Response

```http
Set-Cookie: refreshToken=...
Set-Cookie: accessToken=...
```

주의사항을 참고해 리프레시 토큰과 엑세스 토큰을 쿠키에 저장합니다.

```json
// Response Body

// 1. Success
(200 OK 상태 코드만 반환하며, 바디는 생략할 수 있습니다.)


// 2. Failure (400 Bad Request) - 인가 코드가 누락된 경우

{
  "status": 400,
  "code": "ERR_MISSING_AUTHORIZATION_CODE"
}
```

[**[카카오 로그인 공식문서]**](https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api) 참고

>

## GET /oauth/cookie/member

발급된 Access Token을 사용해 사용자 닉네임 정보를 조회합니다.

### Request

```http
// Request header

Cookie: accessToken=your_access_token_value;
```

### Response

```json
// Response body

// 1. Success
{
  "nickname": "메롱"
}

// 2. Failure (400 Unauthorized) - Access Token이 누락된 경우
{
  "status": 400,
  "code": "ERR_MISSING_ACCESS_TOKEN"
}

// 3. Failure (401 Unauthorized) - Access Token이 만료된 경우
{
  "status": 401,
  "code": "ERR_ACCESS_TOKEN_EXPIRED"
}

// 4. Failure (404 Not Found) - 토큰에 해당하는 사용자를 찾을 수 없는 경우
{
  "status": 404,
  "code": "ERR_NOT_FOUND_MEMBER"
}
```

>

## GET /oauth/cookie/reissue

유저 정보 요청(/member)을 보냈을 때 서버로 부터 401 응답이 내려오면 클라이언트는 reissue 요청을 통해 토큰을 재발급받습니다.

이는 만료된 Access Token을 재발급 받는 것으로, Refresh Token을 통해 진행됩니다.

### Request

```http
// Request header

Cookie: refreshToken={refresh_token_value};
```

### Response

```http
Set-Cookie: accessToken=...
```

```json
// Response body

// 1. Success
(200 OK 상태 코드만 반환하며, 바디는 생략할 수 있습니다.)

// 2. Failure (400 Bad Request) -  Refresh Token이 누락된 경우

{
  "status": 400,
  "code": "ERR_MISSING_REFRESH_TOKEN"
}

// 3. Failure (401 Unauthorized) - Refresh Token이 만료된 경우

{
  "status": 401,
  "code": "ERR_REFRESH_TOKEN_EXPIRED"
}
```

>

## POST /oauth/cookie/logout

로그아웃 처리 시 Refresh Token을 서버에서 무효화합니다.

쿠키를 만료 처리하고 브라우저에 심어, 브라우저에 있는 쿠키를 사라지게 하는 방식으로 진행됩니다.

### Request

```http
// Request header

Cookie: refreshToken={refresh_token_value};
```

### Response

```http
Set-Cookie: refreshToken=...
```

어떻게 쿠키를 만료 처리할지 찾아보세요!

```json
// Response Body

// 1. Success
(200 OK 상태 코드만 반환하며, 바디는 생략할 수 있습니다.)

// 2. Failure (400 Bad Request) -  Refresh Token이 누락된 경우

{
  "status": 400,
  "code": "ERR_MISSING_REFRESH_TOKEN"
}

// 3. Failure (401 Unauthorized) - Refresh Token이 만료된 경우

{
  "status": 401,
  "code": "ERR_REFRESH_TOKEN_EXPIRED"
}

```
