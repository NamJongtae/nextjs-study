# Next.js Image

## 1. Next.js Image 컴포넌트 란?

> **Next.js의 `Image` 컴포넌트는 웹 애플리케이션에서 이미지를 더 효율적으로 로딩하고 표시하기 위해 설계된 컴포넌트입니다.**

<br/>

## 2. Next.js Image 컴포넌트의 기능

### 1 ) 이미지 자동 최적화

이미지는 자동으로 최적화되어 로드 시간이 단축되며, 사용자의 디바이스 및 브라우저에 따라 적절한 크기와 형식으로 제공됩니다.

### 2 ) lazyLoading

**화면에 보이지 않는 이미지는 로딩되지 않아 초기 페이지 로드 시간이 단축됩니다. 사용자가 스크롤하여 이미지가 보일 때만 이미지가 로드됩니다.**

### 3 ) placeholder

**이미지가 로드되는 동안 표시될 플레이스홀더(예: 블러 이미지)를 설정할 수 있어 사용자 경험을 개선할 수 있습니다.**

### 4 ) srcset 및 sizes

**다양한 해상도를 지원하기 위해 `srcset`과 `sizes` 속성이 자동으로 생성됩니다. 이는 브라우저가 화면 크기와 밀도에 가장 적합한 이미지를 선택하도록 도와줍니다.**

### 5 ) 보안 측면

**외부 이미지 URL을 사용할 경우, `next.config.js` 파일에서 도메인을 명시해야 하므로, 안전하지 않은 이미지 소스로부터 보호할 수 있습니다.**

### Next.js Image를 사용하면 얻게되는 장점

- **성능 향상**: 디바이스마다 적절한 사이즈의 이미지를 서빙하고, webp와 같은 작은 용량의 포맷을 사용합니다.
- **시각적인 안정성**: 이미지 로드 전 placeholder를 제공하여 CLS(Cumulative Layout Shift) 방지합니다.
- **빠른 페이지 로딩**: viewport에 들어왔을 때만 이미지를 로드하고, 작은 사이즈의 blur 이미지를 미리 로딩하여 사용자에게 더 빠른 페이지를 보여줄 수 있습니다.

<br/>

## 3. Image 컴포넌트의 속성

### src (필수)

이미지 소스 경로를 지정합니다. 웹상에 있는 원격 이미지와 public 폴더에[ 있는 정적 이미지를 지정할 수 있습니다. 상대 경로로 이미지를 지정할 때는 next.config.js에 domain 정보를 입력 해두어야합니다.

### width, height (필수)

이미지가 페이지에서 얼마나 많은 자리를 차지하느냐 컨테이너에 따라 얼만큼 변하는야를 결정합니다. 이 값은 layout 속성에 따라 변하게됩니다. layout이 `intrinsic` , `fixed` 이면 이미지의 width, height 속성 값이 그대로 고정되고, layout이 `responseive` , `fill` 일 때는 이미지가 비율에 맞게 확대 축소됩니다.

### layout

이미지가 뷰포트에 따라 어떻게 변화하는지를 정의하는 속성입니다. 기본값은 `intrinsic` 이며, 4가지 속성이 존재합니다.

- `intrinsic` : 기본값 이며, 이미지의 width, height에 따라 얼마나 많은 자리를 차지하는지 계산합니다.
- `fixed` : 이미지의 정확한 width, height를 사용하여 계산합니다.
- `fill` : 이미지를 상위 요소 width, height에 맞추기 위해 자동으로 width, height를 조절합니다. 상위 엘리먼트에 position: relative 속성을 적용해야하며, 이미지 사이즈를 정확하게 모를 때 적용하기 좋습니다.
- `responseive` : 부모 컨테이너의 width에 맞게 이미지를 확대합니다. 반드시 부모 컨테이너에 display: block 속성을 추가해야합니다.

### loader

이미지의 URL을 계산해야 할 때 사용하는 옵션입니다. 아래 예시 처럼 사용합니다.

```jsx
import Image from "next/image";

const customLoader = ({ src, width, quality }) => {
  return `https://s3.amazonaws.com/image/${src}?w=${width}&q=${quality}`;
};

const MyImage = (props) => {
  return (
    <Image
      src="profile.webp"
      width={300}
      height={300}
      alt="User profile"
      quality={80}
      loader={customLoader}
    />
  );
};
```

### placeholder

이미지가 로드 되지 않았을 경우 페이지에서 이미지가 차지하는 크기 만큼 대체 이미지를 표시하는 기능입니다. placeholder 옵션으로 blur, empty 값을 지정할 수 있습니다. empty 값이 기본값이며, blur롤 지정할 경우에는 이미지가 완전히 로드 되기 전 blur 처리한 이미지가 보이게 됩니다. 정적으로 제공되는 이미지의 확장자가 .jpg, .png, .webp, .avf인 경우 자동으로 blurDataURL을 제공해줍니다.

### blurDataURL

placeholder blur 속성 적용 시 보여줄 이미지 소스입니다. URL은 base64로 인코딩된 이미지여야 합니다.

### priority

이 속성은 브라우저가 이미지를 미리 렌더링하도록 설정하는 속성입니다. 랜딩 페이지에서 가장 먼저 보이는 이미지에 이 속성을 지정하는 것이 좋습니다.

### quality

이미지의 품질을 결정하는 속성입니다. 1~100 까지 값이 있으며 기본 값은 75입니다.

### sizes

이미지의 사이즈를 화면 뷰포트에 맞게 미리 정해두기 위해 사용하는 옵션입니다. Next.js의 Image 컴포넌트는 source set를 자동으로 생성하는데 source set을 생성하면 뷰포트에 맞는 사이즈의 이미지를 로딩해서 페이지의 레이아웃을 깨지지않게 합니다. next.config.js 파일에 deviceSizes, imageSizes 값을 참조해서 srcSet를 생성하게됩니다.

### loading

브라우저가 페이지를 로딩할 때 이미지 로딩을 어떤 형식으로 처리할 지 지정하는 옵션입니다. `lazy` , `eager` 두 가지 옵션이 존재합니다.

- `lazy` : 뷰포트 계산이 끝날 때 까지 이미지를 대기하는 기능입니다. 페이지 레이아웃이 망치는 것을 방지할 수 있습니다.
- `eager` : 이미지를 즉시 로딩하는 옵션입니다.

### objectFit

layout 속성이 `fill` 인 경우 부모 요소에 대비 사이즈를 조정하는 옵션입니다. CSS의 object-fit 속성과 동일합니다. 해당 옵션 값으로는 `fill` , `cover`, `contain` 이 있습니다.

### obejectPosition

이미지의 위치를 이미지가 위치하는 부모(상위) 콘텐츠 기준으로 어느 위치로 배치할 것인가를 결정하는 속성입니다. 기본 값은 50% 50%으로 가운데로 지정되어 있습니다. 옵션 값으로 top, right, bottom, left를 줄 수 있습니다. obejectPosition: “top left” 이런 식으로 사용합니다.

### onLoadingComplete

이미지 로딩이 끝나면 호출되는 콜백 함수입니다. `naturalHeight` , `naturalWidth` 값을 가지는 객체를 파라미터로 가집니다.

### style

이미지의 특정 CSS스타일을 지정하는 속성입니다. className으로 스타일을 지정할 수도 있습니다. layout 속성으로 지정된 스타일은 style 속성 보다 우선시 되며, style 속성으로 width를 조정하고 싶을 때 height=auto로 설정해야합니다. 그렇지 않으면 이미지가 왜곡됩니다.

### nextjs.config.js Image 설정

Image 컴포넌트에 대한 옵션으로는 `loader` , `domains` 옵션이 존재합니다.

- loader : 옵션은 상대 경로의 이미지에 대해 정확한 path를 지정하게 해줍니다.
  - loader를 통해 상대 경로로 이미지 위치를 지정하지만, 빌드 시에 절대 경로로 Next.js가 자동으로 변환하여 줍니다.
- domains : 외부 이미지 host name를 지정할 수 있습니다. 지정된 host name 이외의 호스트에서 이미지를 불러오면 400 Bad Request가 반환됩니다.

<br/>

## 4. Image 컴포넌트 사용하기

### 1 ) 일반 img 태그와 Image 컴포넌트 비교

**img태그 사용**

```jsx
import Link from "next/link";
import React from "react";
import Image from "next/legacy/image";

export default function ImageTest() {
  const DATA = [
    "https://images.unsplash.com/photo-1711299253442-de19d4dacaae?q=80&w=1475&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D",
    "https://images.unsplash.com/photo-1710880079976-0b838851c8c5?q=80&w=1472&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D",
    "https://images.unsplash.com/photo-1470071459604-3b5ec3a7fe05?q=80&w=1574&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D",
    "https://images.unsplash.com/photo-1426604966848-d7adac402bff?q=80&w=1470&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D",
  ];
  return (
    <div style={{ margin: "0 auto", maxWidth: "500px", height: "auto" }}>
      <ul style={{ width: "100%", height: "100%" }}>
        {DATA.map((url, index) => (
          <li
            key={url}
            style={{
              marginBottom: "2rem",
              position: "relative",
              width: "100%",
              height: "100%",
            }}
          >
            <Link href={url} style={{ fontSize: "20px", fontWeight: "bold" }}>
              {"img" + index}
            </Link>
            <img
              style={{ padding: "1rem" }}
              src={url}
              width={500}
              height={340}
              alt={"img" + index}
            />
          </li>
        ))}
      </ul>
    </div>
  );
}
```

일반 img 태그를 사용하였을 경우 전체 용량이 약 **692KB**

![img](https://github.com/NamJongtae/nextjs-study/assets/113427991/54a04d03-e90a-4ec4-af22-25669232d4bf)

<br/>

**Image 컴포넌트를 사용한 경우**

```jsx
import Link from "next/link";
import React from "react";
import Image from "next/legacy/image";

export default function ImageTest() {
  const DATA = [
    "https://images.unsplash.com/photo-1711299253442-de19d4dacaae?q=80&w=1475&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D",
    "https://images.unsplash.com/photo-1710880079976-0b838851c8c5?q=80&w=1472&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D",
    "https://images.unsplash.com/photo-1470071459604-3b5ec3a7fe05?q=80&w=1574&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D",
    "https://images.unsplash.com/photo-1426604966848-d7adac402bff?q=80&w=1470&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D",
  ];
  return (
    <div style={{ margin: "0 auto", maxWidth: "500px", height: "auto" }}>
      <ul style={{ width: "100%", height: "100%" }}>
        {DATA.map((url, index) => (
          <li
            key={url}
            style={{
              marginBottom: "2rem",
              width: "100%",
              height: "100%",
            }}
          >
            <Link
              href={url}
              style={{
                fontSize: "20px",
                fontWeight: "bold",
                marginBottom: "10px",
                display: "block",
              }}
            >
              {"img" + index}
            </Link>
            <Image
              style={{ padding: "1rem" }}
              src={url}
              alt={"img" + index}
              width={500}
              height={340}
            />
          </li>
        ))}
      </ul>
    </div>
  );
}
```

image 컴포넌트를 사용하였을 경우 전체 용량이 약 **97KB**

img 태그에 비해 약 71% 용량이 감소하였습니다.

![img2](https://github.com/NamJongtae/nextjs-study/assets/113427991/2afaae86-1908-4af9-82b0-ca5ef97eabd5)

Image 태그의 lazyloading 기능 이미지 태그는 기본적으로 화면에 이미지가 보일때 만 로딩되도록 lazyloding 처리되어 최적화 되어있습니다. 아래 이미지에서 스크롤을 내릴때 네트워크 탭에서 이미지가 로드되는 것을 볼 수 있습니다.

![img_gif](https://github.com/NamJongtae/nextjs-study/assets/113427991/7c38bb40-3eac-4a05-816c-acd221841f80)

### 2 ) Image 컴포넌트 layout 속성 적용하기

**intrisic(기본값)**

기본 img 태그와 동일한 동작을 합니다.

컨테이너 사이즈가 이미지 사이즈보다 작아졌을 때 컨테이너에 맞게 크기가 줄어듭니다.

```jsx
<Image
  style={{ padding: "1rem" }}
  src={url}
  alt={"img" + index}
  layout="intrinsic"
  width={500}
  height={340}
/>
```

![intrisic](https://github.com/NamJongtae/nextjs-study/assets/113427991/54e1bd7c-0b8a-4e78-9cef-718cb9921a9a)

<br/>
**fixed**

이미지의 크기와 높이가 고정됩니다.

컨테이너 사이즈를 줄이더라도 이미지 크기는 변하지 않고 고정됩니다.

```jsx
<Image
  style={{ padding: "1rem" }}
  src={url}
  alt={"img" + index}
  layout="intrinsic"
  width={500}
  height={340}
/>
```

![fixed](https://github.com/NamJongtae/nextjs-study/assets/113427991/03b41611-1e81-4e0c-8974-54f93e4b2871)

**responsive**

이미지 비율이 유지되며, 작은 컨테이너에서는 크기가 줄어들고, 큰 컨테이너에서는 크기가 늘어납니다.

```jsx
<Image
  style={{ padding: "1rem" }}
  src={url}
  alt={"img" + index}
  layout="responsive"
  width={500}
  height={340}
/>
```

![responsive](https://github.com/NamJongtae/nextjs-study/assets/113427991/76b4871f-e4b6-4e48-ae0f-279574bf17d9)

**fill**

relative 포지션을 가진 조상의 너비, 높이와 동일하게 조정하며. objectFit 속성과 함께 사용합니다. 이미지 비율은 보장되지 않습니다.

주로 이미지 사이즈를 정확히 모르는 경우 사용합니다.

자동으로 너비 높이가 조정되기 때문에 width, height를 설정하지 않습니다.

```jsx
<div style={{ postition: "relative", height: "340px" }}>
  <Image
    style={{ padding: "1rem" }}
    src={url}
    alt={"img" + index}
    layout="fill"
    objectFit="cover"
  />
</div>
```

![fill](https://github.com/NamJongtae/nextjs-study/assets/113427991/d6562bd3-f0d8-4cb6-abed-c005155f793b)

<br/>

### 3 ) sizes 옵션 설정하기

layout 옵션을 `responsive` , `fill` 로 설정하였을 경우 설정 할 수 있습니다.

sizes를 설정하지 않으면 기본 값 설정에 따라 뷰포트에 따라 이미지의 너비가 가변적으로 변하게 됩니다. 이로 인해 해당 이미지 크기가 실제 렌더링 되는 이미지 사이즈 보다 더 큰 사이즈로 가져오게 되는 문제가 발생할 수 있습니다.

sizes값을 설정하지 않으면 적용되는 기본값은 아래와 같습니다.

```jsx
  images: {
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
  }
```

<br/>

**sizes 값 지정**

화면의 너비가 768px 이하일 경우: 이미지는 화면 너비의 100%를 차지합니다.

화면의 너비가 768px을 초과하고 1200px 이하일 경우: 이미지는 화면 너비의 50%를 차지합니다 .

화면의 너비가 1200px을 초과할 경우: 이미지는 화면 너비의 33%를 차지합니다.

아래 사이트에서 Lint Images 북마크에 추가하고 북마크로 불러오면 현재 화면 이미지를 검사하여 최적화된 이미지 사이즈인지를 체크하고 아니라면 필요한 sizes를 자동으로 계산하여 sizes를 제시하여줍니다. 또한 다양한 lint 옵션이 적용되어 있어 이미지에 필요한 검사를 추가적으로 해줍니다.

[✨ RespImageLint](https://ausi.github.io/respimagelint/)

```jsx
<div style={{ postition: "relative", height: "340px" }}>
  <Image
    style={{ padding: "1rem" }}
    src={url}
    alt={"img" + index}
    layout="fill"
    objectFit="cover"
    sizes="(max-width: 768px) 100vw, (max-width:1200px) 50vw, 33vw"
  />
</div>
```

<br/>

### 4 ) placeholder

placeholder의 기본값은 empty입니다. empty값은 단어 그대로 빈 화면을 출력해주는 것을 의미합니다. 정적 이미지를 불러오는 경우에는 Image 컴포넌트는 자동으로 placeholder 설정을 blur로 처리하며, blurDataURL를 자동으로 생성하여 줍니다. 하지만 정적 이미지가 아닌 URL로 불러오는 이미지 리소스 같은 경우에는 placeholder 설정를 따로 해주어야합니다. blurDataURL은 base64 형식을 취해야합니다.

아래 링크는 blurDataURL 생성 시 유용한 이미지를 base64 형식으로 바꾸어 주는 사이트입니다.

[🖼 Image to blurred data URLs using Blurhash](https://blurred.dev/)

```jsx
<div style={{ position: "relative", width: "100%", height: "340px" }}>
  <Image
    src={url}
    alt={"img" + index}
    layout="fill"
    objectFit="cover"
    placeholder="blur"
    blurDataURL="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAoAAAAKCAYAAACNMs+9AAAAFklEQVR42mN8//HLfwYiAOOoQvoqBABbWyZJf74GZgAAAABJRU5ErkJggg=="
  />
</div>
```

placeholder 적용 후

![placeholder_blur](https://github.com/NamJongtae/nextjs-study/assets/113427991/82c91ef8-3bd7-4097-9986-9b6f2550f86a)
