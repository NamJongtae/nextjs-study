# STMP 이메일 인증

## 1. STMP란?

> **SMTP(Simple Mail Transfer Protocol)는 인터넷을 통해 이메일을 전송하기 위해 사용되는 프로토콜입니다.**

**이 프로토콜은 이메일을 송신자에서 수신자의 메일 서버로, 그리고 최종적으로 수신자의 이메일 계정으로 전달하는 과정을 규정합니다. SMTP는 주로 이메일을 보내는 데 사용되며, 이메일을 받아오는 데는 POP(Post Office Protocol)나 IMAP(Internet Message Access Protocol) 같은 다른 프로토콜이 사용됩니다.**

💡 **POP(Post Office Protocol)**

**POP는 이메일 클라이언트가 메일 서버로부터 이메일을 다운로드하고, 대부분의 경우 서버에서 해당 메일을 삭제하는 방식으로 작동합니다. 이는 이메일을 로컬 컴퓨터에서 관리하고자 할 때 유용합니다.**

💡 **IMAP(Internet Message Access Protocol)**

**IMAP은 이메일 클라이언트가 서버에 저장된 이메일을 원격으로 조회하고 관리할 수 있도록 합니다. 이메일은 서버에 남아있기 때문에, 여러 기기에서 동일한 이메일 계정에 접근할 때 일관된 상태를 유지할 수 있습니다.**

**SMTP 서버는 이메일 전송을 위한 중앙 허브 역할을 하며, 사용자의 이메일 클라이언트(예: Outlook, Gmail)에서 이메일을 보낼 때 해당 서버를 통해 이메일이 전송됩니다. 이메일이 성공적으로 전송되기 위해서는 보내는 이의 이메일 서버 설정에서 올바른 SMTP 서버 주소와 포트 번호가 필요합니다.**

<br/>

## 2. STMP 이메일 인증 Flow

먼저, 클라이언트에서 사용자가 이메일과 함께 인증 요청을 보냅니다.

서버에서는 인증 요청을 받아 STMP를 통해 해당 유저의 이메일로 인증번호를 발송합니다.

여기서 서버는 해당 인증 번호를 DB에 저장합니다.

이후, 유저는 인증번호를 확인 후 인증 번호 확인 요청을 보냅니다.

서버에서는 해당 인증번호가 DB에 저장된 인증번호와 일치하는지 확인합니다.

인증번호가 일치하다면 인증완료 응답을 클라이언트에게 전송합니다.

![STMP_Flow ](https://github.com/NamJongtae/nextjs-study/assets/113427991/5cb7046e-2769-4882-9c88-30a2897bba0c)

<br/>

## 3. STMP 이메일 인증 구현

### 1 ) STMP 설정하기

여기서는 Naver STMP를 이용하였습니다.

네이버 사이트에 들어가 로그인 후 메일 페이지로 들어갑니다.

이후 메일 페이지 왼쪽 하단 환경설정 페이지로 들어갑니다.

![STMP_1](https://github.com/NamJongtae/nextjs-study/assets/113427991/fbeac90a-fec7-4047-bd19-a84b06b52ae7)

환경 설정 페이지에서 POP3/IMAP 설정을 클릭합니다.

![STMP_2](https://github.com/NamJongtae/nextjs-study/assets/113427991/9b3e527c-aa65-4483-9c7c-db9ca2f6f691)

메뉴에서 IMAP/STMP 설정을 클릭합니다.

IMAP/SMTP 사용함에 체크합니다.

이후 저장 버튼을 눌러 저장합니다.

![STMP_3](https://github.com/NamJongtae/nextjs-study/assets/113427991/b959fa2f-09bf-4901-a178-b2a93a4b011d)

<br/>

### 2 ) 라이브러리 설치

nodemailer는 Node.js에서 이메일을 쉽게 보낼 수 있게 도와주는 모듈입니다. 이 모듈을 사용하면 SMTP 서버를 직접 설정하거나 관리하지 않고도, 몇 가지 간단한 설정을 통해 이메일을 보낼 수 있게 됩니다.

**nodemailer** 라이브러리를 설치합니다.

```bash
npm install nodemailer
```

nodemailer transporter를 아래와 같이 생성합니다.

이메일 계정과 비밀번호를 사용하기때문에 환경변수를 사용하는 것을 권장드립니다.

```jsx
export const smtpTransport = nodemailer.createTransport({
  pool: true,
  maxConnections: 1,
  service: "naver",
  host: "smtp.naver.com",
  port: 587,
  secure: false,
  requireTLS: true,
  auth: {
    user: "네이버 이메일 계정",
    pass: "계정 비밀번호",
  },
  tls: {
    rejectUnauthorized: false,
  },
});
```

자세한 옵션은 Nodemailer 공식 사이트를 참고해주세요.

📃 [Nodemailer :: Nodemailer](https://www.nodemailer.com/)

<br/>

### 3 ) 인증 로직

**이메일 인증 요청 API Router**

이메일 인증번호를 저장할 때 redis를 이용하였습니다.

redis를 사용한 이유는 간단하게 만료시간을 지정할 수 있기때문입니다.

redis에 만료시간을 주어 해당 시간안에 인증번호를 입력하지 못하면 인증이 되지 않도록 구현하였습니다.

```jsx
import { saveVerificationCode } from "@/lib/api/redis";
import { setEXP } from "@/lib/api/token";
import { generateRandomNumber, smtpTransport } from "@/lib/server";

export default async function handler(req, res) {
  const verificationCode = generateRandomNumber(111111, 999999);
  const { email } = req.body;
  const mailOptions = {
    from: "발신 이메일 주소",
    to: email,
    subject: "서비스 인증 메일입니다.",
    html: `<h1>인증번호를 입력해주세요.</h1><h1>${number}</h1>`,
  };

  try {
    await new Promise((resolve, reject) => {
      smtpTransport.sendMail(mailOptions, (err, response) => {
        if (err) {
          console.log(err);
          reject(err);
        } else {
          console.log(response);
          resolve(response);
        }
      });
    });
    // 만료시간 설정 180초 => 3분
    const exp = setEXP(180);
    // redis에 인증코드 저장
    await saveVerificationCode(email, verificationCode, exp);
    res.json({ message: "메일 전송에 성공하였습니다.", ok: true });
  } catch (error) {
    console.log(error);
    res.status(500).json({ message: "메일 전송에 실패하였습니다.", ok: false });
  }
}
```

메일을 보내면 아래와 같이 사용자의 메일에 인증메일이 도착하게 됩니다.

![STMP4](https://github.com/NamJongtae/nextjs-study/assets/113427991/d14b6c2a-7336-4ef9-9e1a-92edf80f9f8e)

**이메일 인증 확인 API Router**

redis에서 인증번호를 가져와 유저가 보낸 인증번호와 일치하는지 확인합니다.

```jsx
import { getVerificationCode } from "@/lib/api/redis";

export default async function handler(req, res) {
  const { email, number } = req.body;
  // 인증번호를 redis에서 불러옵니다.
  const verificationCode = await getVerificationCode(email);
  if (parseInt(number, 10) === certificationNum) {
    res.status(200).json({ message: "인증이 완료되었습니다.", verify: true });
  } else {
    res.status(401).json({ message: "인증에 실패하였습니다.", verify: false });
  }
}
```

<br/>

### 4 ) 동작 화면

<div align="center">
  <img src="https://github.com/NamJongtae/nextjs-study/assets/113427991/d779a8c2-a4d6-4fe9-86e7-4d1e976bbb5d" alt="STMP_Email" width="100%"/>
</div>
