# Socket.io

## 1. Socket.io 란?

> **Socket.IO는 실시간, 양방향, 이벤트 기반의 통신을 가능하게 하는 JavaScript 라이브러리입니다.**

주로 웹 애플리케이션에서 사용되며, 클라이언트와 서버 간에 실시간 데이터를 교환하기 위해 설계되었습니다. WebSocket을 기반으로 하지만, WebSocket을 지원하지 않는 환경에서도 작동할 수 있도록 폴백 옵션을 제공합니다.

<br/>

### **특징**

**실시간 양방향 통신** : 클라이언트와 서버가 서로에게 실시간으로 데이터를 보낼 수 있습니다.

**자동 재연결 지원** : 네트워크 연결이 끊어진 후 자동으로 재연결을 시도합니다.

**방(Rooms) 기반 방송** : 특정 클라이언트 그룹에게만 메시지를 전송할 수 있습니다.

**다양한 플랫폼 및 브라우저 지원** : 대부분의 주요 브라우저와 Node.js 환경에서 사용할 수 있습니다.**폴백 메커니즘** : `WebSocket`을 지원하지 않는 환경에서도 `Long Polling` 등의 방법을 사용하여 통신할 수 있습니다.

<br/>

💡 **WebSocket**

WebSocket은 HTML5에 도입된 기술로, 클라이언트와 서버 간에 지속적인 연결을 유지하며 양방향 통신을 가능하게 합니다. 이 연결은 HTTP를 통해 초기에 열리지만, 연결이 확립되면 HTTP 프로토콜과는 독립적으로 작동합니다. WebSocket 연결은 연결이 명시적으로 닫힐 때까지 유지되므로, 데이터를 주고받을 때 추가적인 요청이 필요하지 않습니다. 이는 매우 낮은 지연시간으로 실시간 통신을 가능하게 합니다.

💡 **Long Polling**

Long Polling은 실시간 통신을 구현하기 위한 더 오래된 방법 중 하나입니다. 클라이언트가 서버에 요청을 보내고, 서버가 응답할 데이터가 준비될 때까지 요청을 "대기"시킵니다. 데이터가 준비되면 서버는 그 데이터를 응답으로 보내고 연결을 종료합니다. 그 후 클라이언트는 즉시 다른 요청을 보내어 과정을 반복합니다. 이 방법은 실시간으로 보이지만, 실제로는 연속적인 요청과 응답의 시퀀스로 구성됩니다. 이 과정에서 발생하는 지연과 추가적인 HTTP 요청으로 인해 WebSocket보다 효율성이 떨어집니다.

<br/>

### 이벤트 통신

socket.io의 메서드는 클라이언트에서 발생하는 이벤트는 개발자가 원하도록 설정할 수 있습니다.

이벤트는 문자열로 작성하며, 직접 이벤트를 발생시킬 수 있습니다.

```jsx
// 해당 이벤트를 받고 콜백함수를 실행
socket.on("받을 이벤트 명", (msg) => {});

// 이벤트 명을 지정하고 메세지를 보낸다.
socket.emit("전송할 이벤트 명", msg);
```

<br/>

### 송신/수신 메서드

**메세지 수신**

```jsx
// 접속된 모든 클라이언트에게 메시지를 전송한다
io.emit("event_name", msg);

// 메시지를 전송한 클라이언트에게만 메시지를 전송한다
socket.emit("event_name", msg);

// 메시지를 전송한 클라이언트를 제외한 모든 클라이언트에게 메시지를 전송한다
socket.broadcast.emit("event_name", msg);

// 특정 클라이언트에게만 메시지를 전송한다
io.to(id).emit("event_name", data);
```

**메세지 송신**

```jsx
// 클라이언트와 소켓IO 연결됬는지 안됬는지 이벤트 실행. (채팅방에 누가 입장하였습니다/퇴장하였습니다 )
io.on("connection/disconnection", (socket) => {});

// 클라이언트에서 지정한 이벤트가 emit되면 수신 발생
socket.on("event_name", (data) => {});
```

<br/>

## 2. Nextjs에서 Socket.io 사용하기

Nextjs의 API router 기능을 통해 serverless 환경에서 Socket.io를 구현할 수 있습니다.

### 1 ) Socket.io 라이브러리 설치하기

```bash
npm install socket.io socket.io-client
```

<br/>

### 2 ) Socket.io API router 생성하기

```jsx
// pages/api/socket/io
import { Server as ServerIO } from "socket.io";

const ioHandler = (req, res) => {
  if (!res.socket.server.io) {
    const path = "/api/socket/io";
    const httpServer = res.socket.server;
    const io = new ServerIO(httpServer, {
      path: path,
      addTrailingSlash: false,
    });
    res.socket.server.io = io;
    res.end();
  }
};
export default ioHandler;
```

<br/>

### 3 ) client에 연결하기

```jsx

import { useEffect, useRef } from "react";
import SocketIOClient from "socket.io-client";

export default function MyComponent() {
  const [socket, setSocket] = useState(null);

  // socket.io 연결
  useEffect(() => {
    const socketInstance = SocketIOClient.connect(process.env.NEXT_PUBLIC_API_URL, {
      path: "/api/socket/io",
    });

    socket.current.on("connect", () => {
      console.log("SOCKET CONNECTED!", socket.current.id);
    });

    setSocket(socketInstance);

    return() => {
      if (socket) {
        socket.disconnect();
      }
    }
  }, []);

  return (
		//...
  );
};

```

<br/>

## 3. Socket.io 채팅구현하기

### 1 ) 채팅 API Router 생성

```jsx
// pages/api/chat

export default async function handler(req, res) {
  if (req.method === "POST") {
    const message = req.body;
    res.socket.server.io.emit("message", message);
    res.status(201).json(message);
  }
}
```

<br/>

### 2 ) Client 채팅 연결하기

```jsx
import React, { useState, useEffect } from "react";
import axios from "axios";

export default function Chat() {
  const [socket, setSocket] = useState(null);
  const [currentMessage, setCurrentMessage] = useState("");
  const [messages, setMessages] = useState([]);
  const userId = +new Date();

  // socket.io 연결
  useEffect(() => {
    const socketInstance = SocketIOClient.connect(process.env.NEXT_PUBLIC_API_URL, {
      path: "/api/socket/io",
    });

    socket.current.on("connect", () => {
      console.log("SOCKET CONNECTED!", socket.current.id);
    });

    setSocket(socketInstance);

    return() => {
      if (socket) {
        socket.disconnect();
      }
    }
  }, []);

  // socket.io 채팅 이벤트
  useEffect(() => {
    if (!socket) return;

    socket.on("message", (message) => {
      setMessages((prevMessages) => [...prevMessages, message]);
    });

    return () => {
      if (socket) {
        socket.off("message");
      }
    };
  }, [userId, socket]);

  // 채팅 메세지 전송
  const sendMessage = async (e) => {
    e.preventDefault();
    await axios.post("/api/chat", {
      userName: userId,
      content: currentMessage,
    });

    setCurrentMessage("");
  };

  return (
    //...
  )
}
```

<br/>

### 구현화면

![socketio_chat](https://github.com/NamJongtae/nextjs-study/assets/113427991/6c082c96-6594-4ef3-80d6-8e95d530e974)

<br/>

## 4. Socket.io 마우스 커서 공유 구현하기

### 1 ) API Router 생성하기

마우스 관리 API

```jsx
// pages/api/mouse/index.js

export default function handler(req, res) {
  if (req.method === "POST") {
    const mouseInfo = req.body;
    res.socket.server.io.emit("mousemove", userInfo);
    res.status(201).json(mouseInfo);
  }
}
```

유저 연결 해제 API ⇒ 마우스 커서 삭제

```jsx
// pages/api/disconnected/index.js

export default async function handler(req, res) {
  if (req.method === "POST") {
    const user = req.body;
    res.socket.server.io.emit("disconnected", user);
    res.status(200).json(user);
  }
}
```

<br/>

### 2 ) Client 마우스 커서 공유 연결하기

```jsx
// pages/usermoues/index.js

import React, { useState, useEffect } from "react";
import UserMousePointer from "./UserMousePointer";
import axios from "axios";

function throttle(cb, time) {
  let timer;
  return function (...args) {
    if (timer) {
      return;
    }

    timer = setTimeout(() => {
      cb.apply(this, args);
      timer = null;
    }, time);
  };
}

export default function UserMouse({ socket }) {
  const [socket, setSocket] = useState(null);
  const userId = +new Date();
  const [mouseInfo, setMouseInfo] = useState(null);
  const [screenSize, setScreenSize] = useState({ width: 0, height: 0 });

  const handleMouseMove = throttle(async (event) => {
    // 현재 화면의 크기를 받아옵니다.
    const screenSize = {
      width: window.innerWidth,
      height: window.innerHeight,
    };

    // 마우스 위치를 화면 크기에 대해 정규화합니다.
    const normalizedX = event.clientX / screenSize.width;
    const normalizedY = event.clientY / screenSize.height;

    // 정규화된 위치를 서버에 전송합니다.
    await axios.post("/api/mouse", {
      [userId.toString()]: { x: normalizedX, y: normalizedY },
    });
  }, 50);

  // 유저 연결 종료
  const fetchDisconncted = async () => {
    await axios.post("/api/disconnected", {
      userId,
    });
  };

  // socket.io 연결
  useEffect(() => {
    const socketInstance = SocketIOClient.connect(
      process.env.NEXT_PUBLIC_API_URL,
      {
        path: "/api/socket/io",
      }
    );

    socket.on("connect", () => {
      console.log("SOCKET CONNECTED!", socket.current.id);
    });

    setSocket(socketInstance);

    return () => {
      if (socket) {
        socket.disconnect();
      }
    };
  }, []);

  // 창 크기 설정
  useEffect(() => {
    const updateScreenSize = throttle(() => {
      setScreenSize({ width: window.innerWidth, height: window.innerHeight });
    }, 100);

    // 컴포넌트 마운트 시 screenSize를 현재 창 크기로 설정
    updateScreenSize();

    // 창 크기가 변경될 때마다 screenSize 업데이트
    window.addEventListener("resize", updateScreenSize);

    // 컴포넌트 언마운트 시 이벤트 리스너 제거
    return () => {
      window.removeEventListener("resize", updateScreenSize);
    };
  }, []);

  // 마우스 커서 공유
  useEffect(() => {
    if (!socket) return;
    socket.on("mousemove", (userInfo) => {
      setMouseInfo((prevMouseInfo) => ({
        ...prevMouseInfo,
        ...mouseInfo,
      }));
    });

    socket.on("disconnected", (data) => {
      setMouseInfo((prevMouseInfo) => {
        const newMouseInfo = { ...prevMouseInfo };
        delete newUserInfo[data.userId];
        return newUserInfo;
      });
    });

    return () => {
      if (!socket) return;
      fetchDisconncted();
      socket.off("mousemove");
      socket.off("userDisconnected");
    };
  }, [userId, socket]);

  // 마우스 이벤트 등록 및 제거
  useEffect(() => {
    window.addEventListener("mousemove", handleMouseMove);

    return () => {
      window.removeEventListener("mousemove", handleMouseMove);
    };
  }, [handleMouseMove]);

  return <UserMousePointer mousePositions={userInfo} screenSize={screenSize} />;
}
```

<br/>

**UserMousePointer**

```jsx
import { AuthContext } from "@/store/AuthContext";
import React, { useContext } from "react";
import MousePointer from "./MousePointer";
import { generateRandomColor } from "@/lib/color";

function UserMousePointer({ mousePositions, screenSize }) {
  const { user } = useContext(AuthContext);

  if (!mousePositions) return null;

  return (
    <>
      {Object.keys(mousePositions).map((key) => {
        const position = mousePositions[key];
        if (!position) return null;

        // 화면의 실제 픽셀 위치로 변환
        const { width, height } = screenSize;
        const { x, y } = position;
        const adjustedX = x * width;
        const adjustedY = y * height;

        if (key === user?.name) return null;

        // 유저별 마우스 랜덤 색상 부여
        const color = generateRandomColor(key);
        return (
          <div
            id={user?.name || key}
            key={key}
            style={{
              position: "absolute",
              left: `${adjustedX}px`,
              top: `${adjustedY}px`,
              width: "24px",
              height: "24px",
              transform: "translate(-50%, -50%)",
            }}
          >
            <p className="absolute top-6 left-[50%] translate-x-[-30%] text-[11px] bg-gray-200 py-1 px-1 rounded-md text-sky-700">
              {key}
            </p>
            <MousePointer color={color} />
          </div>
        );
      })}
    </>
  );
}

export default UserMousePointer;
```

<br/>

**MousePointer**

```jsx
import React from "react";

const MousePointer = ({ color }) => {
  return (
    <svg
      width="24px"
      height="24px"
      viewBox="0 0 24 24"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
      stroke="#eee"
    >
      <g id="SVGRepo_bgCarrier" strokeWidth="0" />

      <g
        id="SVGRepo_tracerCarrier"
        strokeLinecap="round"
        strokeLinejoin="round"
      />

      <g id="SVGRepo_iconCarrier">
        <path
          d="M7.92098 2.29951C6.93571 1.5331 5.5 2.23523 5.5 3.48349V20.4923C5.5 21.9145 7.2945 22.5382 8.17661 21.4226L12.3676 16.1224C12.6806 15.7267 13.1574 15.4958 13.6619 15.4958H20.5143C21.9425 15.4958 22.5626 13.6887 21.4353 12.8119L7.92098 2.29951Z"
          fill={color}
        />
      </g>
    </svg>
  );
};

export default MousePointer;
```

<br/>

### 구현화면

![socketio_mouse](https://github.com/NamJongtae/nextjs-study/assets/113427991/382e0e7e-af28-4ec7-8022-a5c9cfdd693c)

<br/>

## 5. Nextjs API Router 구현 Soket.io의 한계점

Next.js의 API 라우터는 REST API를 쉽게 구현할 수 있도록 설계되었습니다. Socket.IO와 같은 실시간 통신 라이브러리는 Next.js의 API 라우터를 사용하여 직접 구현하기 어려울 수 있으며, 이는 Next.js의 API 라우터가 제공하는 자동화된 기능과 최적화를 활용하기 어렵게 만듭니다.

<br/>

## 참고 사이트

[[SOCKET] 📚 Socket.IO 사용 해보기](https://inpa.tistory.com/entry/SOCKET-📚-SocketIO-사용-해보기)
