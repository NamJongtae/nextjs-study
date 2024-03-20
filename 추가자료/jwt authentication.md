## 1. JWT Authentication Flow

사용자가 로그인을 시도하면 서버는 사용자를 인증하고, 인증이 성공하면 `accessToken`과 `refreshToken`을 발급합니다. 발급된 토큰은 클라이언트에게 쿠키나 데이터로 전송됩니다.

사용자는 서버에 요청을 보낼 때 `accessToken`을 헤더에 실어 보냅니다.

서버는 `accessToken`를 검증하여 유효한 토큰인지를 파악합니다.

유효하지 않은 `accessToken` 이라면 클라이언트에게 `accessToken` 이 유효하지 않다는 에러를 보냅니다.

클라이언트는 에러 메세지를 받으면 `refreshToken` 를 서버에 전송합니다.

서버는 `refreshToken` 을 받아 유효한 토큰인지 파악하고, 유효한 토큰이라면 새로운 `accessToken` 를 발급하여 클라이언트에게 전송합니다. 만약 `refreshToken` 이 유효하지 않다면 현재 계정을 로그아웃 시킵니다.

![JWT_Flow](https://github.com/NamJongtae/nextjs-study/assets/113427991/afabcdd0-654a-4fca-9e99-f9c5a3534824)

<br/>

## 2. JWT Authentication 구현 방식

먼저 로그인이 성공하면 accessToken RefreshToken를 발급하고, 만료시간 체크를 위해 accessTokend의 EXP와 함께 쿠키를 통해 클라이언트에 보냅니다. refreshToken은 보안을 위해 redis에 저장합니다.

사용자가 api 요청시 마다 토큰을 검사해야하는데 여기서 2가지 방식으로 로직을 구현해보았습니다.

<br/>

### 1 ) 클라이언트에서 반복적으로 토큰 검사요청

클라이언트에서 interval API를 이용하여 10초마다 반복적으로 토큰을 검사하는 로직을 요청합니다.

쿠키에 저장 exp 값을 통해 accessToken의 만료시간이 10분 이하 일 때 accessToken 재발급 요청을 보냅니다. 서버에서는 refreshToken를 검증하고 유효한 토큰이라면 accessToken를 재발급하고, accessToken과 accessToken 만료시간인 exp를 쿠키를 통해 다시 보내게 됩니다. 만약 refreshToken이 만료되었다면 쿠키를 모두 삭제하고 401 에러코드를 보내게 됩니다. 401 코드를 받은 클라이언트는 로그아웃 처리를 하게됩니다.

**구현 코드**

토큰을 반복적으로 검사하는 컴포넌트를 구현하였습니다.

해당 컴포넌트 \_app.js에서 추가하였습니다.

tokenInfo은 Authcontext에 저장된 토큰 정보입니다.

토큰 정보는 클라이언트에서 쿠키로 직접 접근할 수 없기 때문에 서버를 통해 쿠키를 통해 받아오게 됩니다.

```jsx
import React, { useContext, useEffect, useRef } from "react";
import { useRouter } from "next/router";
import { AuthContext } from "@/store/AuthContext";
import customAxios from "../../lib/customAxios";

const TIME = 60 * 10; // 10m

export default function IntervalValid() {
  const { isLogin, setUser, tokenInfo } = useContext(AuthContext);
  // 토큰 정보
  const { refreshToken, accessTokenExp, resetToken, updateAccessTokenExp } =
    tokenInfo;
  const router = useRouter();
  const interval = useRef(null);

  useEffect(() => {
    if (interval.current) {
      clearInterval(interval.current);
    }

    const watchAccessTokenExpire = async () => {
      if (isLogin) {
        const now = Math.floor(new Date().getTime() / 1000);
        // 남은 시간 = accessTokenExp - 10분 - 현재시간
        const timeRemaining = parseInt(accessTokenExp, 10) - TIME - now;
        //  남은 시간 0 이하인 경우
        if (timeRemaining <= 0) {
          try {
            // refreshToken를 통해 새로운 accessToken 발급 요청
            const { data } = await customAxios("/api/auth/refreshToken");

            // 최신 유저 정보 갱신
            localStorage.setItem("uid", JSON.stringify(data.user));
            // accesTokenExp 갱신
            updateAccessTokenExp(data.exp);
          } catch (error) {
            // refreshToken 만료된 경우
            if (error.response.status === 401) {
              resetToken();
              setUser(null);
              localStorage.clear("uid");
              router.replace("/signin");
              alert("로그인이 만료되었습니다.");
            }
            console.log(error);
          }
        }
      }
    };

    interval.current = setInterval(watchAccessTokenExpire, 1000 * 10);

    return () => clearInterval(interval.current);
  }, [isLogin, accessTokenExp]);
  return <></>;
}
```

**장점**

- 지속적으로 토큰 검사를 하기 때문에 사용자 경험이 좋아집니다.
- 최신 상태의 유저 정보를 갱신할 수 있습니다.

**단점**

- 서버에 지속적으로 토큰 검사를 요청하기 때문에 서버 부하를 줄 수 있습니다.
- 매 요청마다 토큰 상태를 갱신하기 때문에 리렌더링이 발생합니다.

<br/>

### 2 ) Middleware를 통해 토큰 검사하기

사용자가 api를 요청하면 api 처리로 가기전 middleware에서 해당 accessToken이 유효한지 판단합니다. cookie에 저장된 exp를 통해 accessToken 만료시간을 체크합니다.

만약 만료시간이 10분 이하이거나 유효한 토큰이 아니라면 바로 refreshToken를 통해 accessToken 재발급 요청을 하게 됩니다. 재발급된 accessToken으로 기존 api 요청을 다시 진행하게되고 그 결과를 클라이언트에게 반환하게됩니다. 만약, refreshToken 조차 만료되었다면 로그아웃 처리를 하게됩니다.

**구현 코드**

```jsx
import { NextResponse } from "next/server";
import { cookies } from "next/headers";
import { checkWithOutAuthPathname, withOutAuth } from "./lib/withOutAuth";
import { checkWithAuthPathname, withAuth } from "./lib/withAuth";
import { cookieOptions, isValidToken } from "./lib/api/token";
import cookie from "cookie";

const ACCESS_TOKEN_KEY = process.env.NEXT_SECRET_ACCESS_TOKEN_KEY;

const REFRESH_TOKEN_KEY = process.env.NEXT_SECRET_REFRESH_TOKEN_KEY;

export async function middleware(req, res) {
  const { pathname, search } = req.nextUrl;
  // 로그인을 하였을 때만 접속 가능한 url인지 확인
  const isWithAuth = checkWithAuthPathname(pathname);

  // 로그아웃 하였을 때만 접속 가능한 url인지 확인
  const isWithOutAuth = checkWithOutAuthPathname(pathname);

  const accessToken = cookies().get("accessToken")?.value;
  const refreshToken = cookies().get("refreshToken")?.value;

  // 쿠키 토큰 유효성 검사 => 로그인 유뮤 확인
  const isValidAccessToken = await isValidToken(accessToken, ACCESS_TOKEN_KEY);
  const isValidRefreshToken = await isValidToken(
    refreshToken,
    REFRESH_TOKEN_KEY
  );

  // 로그인 확인 및 로그인 페이지는 검사 제외
  if (pathname !== "/signin" && refreshToken) {
    // accessToken 만료 시간
    const accessTokenExp = cookies().get("exp")?.value || 0;
    // accessToken 만료 체크
    const timeReming =
      accessTokenExp - (Math.floor(new Date().getTime() / 1000) + 10 * 60) || 0;

    // 토큰이 유효 않은 경우, 만료시간이 10분 이전인 경우
    // 엑세스 토큰 재발급 요청
    if (!isValidAccessToken || timeReming <= 0) {
      const refreshTokenCookie = cookie.serialize(
        "refreshToken",
        refreshToken,
        cookieOptions
      );
      const response = await fetch(
        `${process.env.NEXT_PUBLIC_API_URL}/api/auth/refreshToken`,
        {
          headers: {
            Cookie: refreshTokenCookie,
          },
        }
      );

      // refresh 토큰이 만료된 경우
      if (response.status === 401) {
        // 리다이렉트
        const redirect = NextResponse.redirect(
          `${process.env.NEXT_PUBLIC_API_URL}/signin?tokenExpired=true`
        );

        redirect.cookies.delete("accessToken");
        redirect.cookies.delete("refreshToken");
        redirect.cookies.delete("exp");
        return redirect;
      }

      // refresh 토큰이 만료 되지 않은 경우
      const data = await response.json();
      const newAccessToken = data.accessToken;

      const exp = data.exp;
      const expCookie = cookie.serialize("exp", exp, cookieOptions);

      // 새로운 엑세스 토큰으로 원래 요청 재시도
      const originalRequestHeaders = new Headers(req.headers);
      const newAccessTokenCookie = cookie.serialize(
        "accessToken",
        newAccessToken,
        cookieOptions
      );
      originalRequestHeaders.set("Cookie", newAccessTokenCookie);

      const originalRequest = new Request(req, {
        headers: originalRequestHeaders,
      });

      const originalResponse = await fetch(originalRequest);

      const responseHeaders = new Headers(originalResponse.headers);
      responseHeaders.append("Set-Cookie", newAccessTokenCookie);
      responseHeaders.append("Set-Cookie", expCookie);

      const newResponse = new NextResponse(originalResponse.body, {
        status: originalResponse.status,
        statusText: originalResponse.statusText,
        headers: responseHeaders,
      });

      return newResponse;
    }
  }

  // 로그인시 접속할 수 있는 페이지를 로그인 없이 접속 방지
  if (isWithAuth && !isValidAccessToken && !isValidRefreshToken) {
    return withAuth(req);
  }

  // 미 로그인시 접속할 수 있는 페이지를 로그인 후 접속 방지
  if (isWithOutAuth && isValidAccessToken && isValidRefreshToken) {
    const callbackUrl = search.replace("?callbackUrl=", "");
    return withOutAuth(req, callbackUrl);
  }
}

export const config = {
  matcher: ["/", "/signin", "/signup", "/profile"],
};
```

**장점**

- 기존 반복적으로 토큰을 검증하는 방식 보다 서버 부하가 덜합니다.
- 모든 토큰 관련 검사 및 갱신 처리가 미들웨어에서 이루어지므로, 토큰 관리 로직을 한 곳에서 관리할 수 있어 코드의 유지 보수성이 향상됩니다.

**단점**

- 서버 부하가 덜 하더라도 여전히 성능저하의 문제가 나타날 수 있습니다.
- 사용자가 아무 행동도 하지 않으면 만료가되더라도 로그아웃 처리가 되지 않습니다.

<br/>

## 3. RefreshToken를 활용한 중복로그인 방지

redis에 저장된 refreshToken를 통하여 중복로그인을 방지 할 수 있습니다.

중복로그인 확인 body에 보내는 isDuplicateLogin flag를 통해 확인합니다.

로그인시 해당 계정에 해당되는 refreshToken이 redis에 남아있는지 조회합니다.

만약 refreshToken이 redis에 남아있다면 서버는 중복 로그인 에러를 보냅니다.

클라이언트는 중복로그인 에러를 받아 로그인을 계속할 건지 사용자에게 물어보게됩니다.

사용자가 로그인을 계속한다면 isDuplicateLogin 값을 true로 해서 다시 로그인 API를 요청합니다.

refreshToken가 덮어쓰여져 기존 로그인된 계정은 accessToken를 갱신할 때 refreshToken이 없어졌으므로 만료처리되어 로그아웃이 됩니다.

**구현 코드**

**서버 로그인 API Router**

```jsx
export default async function handler(req, res) {
  if (req.method === "POST") {
    //...

     // redis에 유저의 refreshToken이 존재하는지 확인
     const refreshTokenData = await getRefreshToken(user.name);
     // 중복로그인이 아니고 refreshTokenData가 존재한다면 에러 409 status 전송
     if (refreshTokenData && !isDuplicateLogin) {
	     res.status(409).json({message: "User already signed in."});
	     return;
     }

     // refreshToken를 기존 refreshToken에 덮어씀
      await saveRefreshToken(user.name, refreshToken, REFRESH_TOKEN_EXP);

     //...
    }
  }
}

```

<br/>

**클라이언트**

로그인 함수

```jsx
export async function signIn(email, password) {
  try {
    const data = await customAxios.post("/api/auth/signin", {
      email,
      password,
      isDuplicateLogin: false,
    });
    return data;
  } catch (error) {
    if (error.response.status === 409) {
      // 로그인 여부 확인
      const isLogin = confirm(
        "Your account is currently logged in on another device. Would you like to log out from that device and log in here instead?"
      );
      if (isLogin) {
        // 기존 로그인을 해제하고 로그인하는 API 요청
        const data = await customAxios.post("/api/auth/signin", {
          email,
          password,
          isDuplicateLogin: true,
        });
        return data;
      }
    }
  }
}
```

하지만 accessToken이 남아있는 시간동안 계정이 계속 활성화 될 수 있다는 단점이 존재합니다.

이를 위해 BlackList를 만들어 이를 방지할 수 있습니다.

<br/>

## 4. BlackList로 기존 accessToken 무효화 시키기

유저가 로그아웃 요청을 보내면 서버에서는 추가로 redis의 BlackList에 해당 유저의 accessToken를 남은 만료시간 만큼 포함시킵니다. 이후 api 요청마다 서버에서는 BackList에 해당 엑세스 토큰이 있는지를 검사하는 로직을 추가하여 BlackList에 있다면 API 요청을 거절하도록 합니다.

**구현코드**

**로그아웃 API Router**

```jsx
export default async function handler(req, res) {
  //...

  const decodeRefreshToken = await isValidToken(
    refreshToken,
    process.env.NEXT_SECRET_REFRESH_TOKEN_KEY
  );
  const decodeAccessToken = await isValidToken(
    accessToken,
    process.env.NEXT_SECRET_REFRESH_TOKEN_KEY
  );
  // 로그아웃시 accessToken를 blackList로 등록
  // accessToken Exp를 balckList 만료시간으로 설정
  await setBlackList(accessToken, decodeAccessToken.exp);
  await deleteRefreshToken(decode.user.name);

  //...
}
```

<br/>

**Middleware**

blackList 확인 로직 추가

```jsx
export async function middleware(req, res) {
  //...

  if (accessToken) {
    const isBlackList = await getBlackList(accessToken);
    if (isBlackList) {
      // 블랙리스트인 경우 모든 쿠키를 삭제시키기
      destroyCookie({ res }, "accessToken", { path: "/" });
      destroyCookie({ res }, "refreshToken", { path: "/" });
      destroyCookie({ res }, "exp", { path: "/" });
      res.status(403).json({ message: "BlackList Token." });
      return;
    }
  }

  //...
}
```

위와 같이 구현하려 했으나 Middleware상 에러가 발생하여 별도의 api를 만들어서 처리하는 것으로 변경하였습니다.

아래는 해당 에러 관련 내용입니다.

⛔ [API resolved without sending a response in Nextjs](https://stackoverflow.com/questions/60684227/api-resolved-without-sending-a-response-in-nextjs)

<br/>

**코드 변경**

```jsx
export async function middleware(req, res) {
  //...

  if (accessToken) {
    const accessTokenCookie = cookie.serialize(
      "accessToken",
      accessToken,
      cookieOptions
    );
    const response = await fetch(
      `${process.env.NEXT_PUBLIC_API_URL}/api/auth/accessToken/blackList`,
      {
        headers: {
          Cookie: accessTokenCookie,
        },
      }
    );
    const { isBlackList } = await response.json();
    if (isBlackList) {
      // 블랙리스트인 경우 모든 쿠키를 삭제시키기
      destroyCookie({ res }, "accessToken", { path: "/" });
      destroyCookie({ res }, "refreshToken", { path: "/" });
      destroyCookie({ res }, "exp", { path: "/" });
      res.status(403).json({ message: "BlackList Token." });
    }
  }

  //...
}
```
