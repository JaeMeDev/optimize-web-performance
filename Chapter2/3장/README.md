- 이미지 레이지(lazy)로딩
- 이미지 사이즈 최적화
- 동영상 최적화
- 폰트 최적화
- 캐시 최적화
- 불필요한 CSS 제거

## 이미지 지연(lazy) 로딩 (intersection observer)

- 성능을 테스트 할 때 network의 preset을 만들어주면 좋다. (기존에 있는 것은 너무 느림)
- 2000단위로 생성해주면 좋음

  ![스크린샷 2022-06-09 오후 1.18.46.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/42cb4f9b-87e2-48cb-bb41-b766ffba39fe/스크린샷_2022-06-09_오후_1.18.46.png)

- 테스트할 때는 캐시 사용 중지를 한다.
- 해당 예시에서 동영상을 빨리 불러오는 방법
    1. 이미지를 빠르게 불러온다.(이미지를 불러오고 동영상을 불러오기 때문)
    2. 이미지는 당장 눈에 보이는 요소가 아니기 때문에 나중에 이미지를 다운로드 하도록 한다.(동영상이 먼저 다운로드 될 수 있도록) ⇒ 이미지가 보여지는 순간 직전에 이미지를 다운로드한다.
        - 스크롤이벤트 활용 시 매스크롤 마다 많은 함수가 호출되는 단점을 가진다.
        - 해결방법 : intersection observer을 활용한다. intersection ovserver를 통해서 특정 객체를 옵저브를 하면 해당 요소가 스크롤 이용해서 화면에 보여지는지 안보여지는지를 판단이 가능하다. ⇒ 화면에 들어올 경우 특정 함수 호출이 가능해짐.

예제)

```jsx
import React, { useRef, useEffect } from "react";

function Card(props) {
  const imgRef = useRef(null);

  useEffect(() => {
    const callback = (entries, observer) => {
      entries.forEach((entry) => {
		 if(entry.isIntersecting){
			 entry.target.src =  entry.target.dataset.src
			 observer.unobserve(entry.target)
		 }
	  });
    };
    const options = {};
    const observer = new IntersectionObserver(callback, options);

    observer.observe(imgRef.current);
  }, []);

  return (
    <div className="Card text-center">
      <img data-src={props.image} ref={imgRef} />
      <div className="p-5 font-semibold text-gray-700 text-xl md:text-lg lg:text-xl keep-all">
        {props.children}
      </div>
    </div>
  );
}

export default Card;
```

## 이미지 사이즈 최적화

- 압축(이미지 크기 줄이고, 용량을 낮추자)
- 이미지 포멧
    - png
    - jpg
    - webp
- webp란 구글에서 나온 차세데 이미지 포멧으로 jpg보다 더 좋은 이미지 포맷이다. 지원하지 않은 브라우저가 있는 것이 단점이지만 화질, 사이즈면에서 월등이 좋다.
- [squoosh.app](http://squoosh.app) 에서 이미지 사이즈 최적화(webp 변경)가 가능하다.
- webp가 지원하지 않는 브라우저에서는 추가적인 이미지 분기가 필요하다. picture 태그를 사용한다.

```jsx
useEffect(() => {
    const callback = (entries, observer) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const target = entry.target;
          const previousSibling = target.previousSibling;
          target.src = target.dataset.src;
          previousSibling.srcset = previousSibling.dataset.srcset;
          observer.unobserve(entry.target);
        }
      });
    };
    const options = {};
    const observer = new IntersectionObserver(callback, options);

    observer.observe(imgRef.current);
  }, []);

<picture>
        <source data-src={props.webpImage} type="image/webp" />
        <img data-src={props.image} ref={imgRef} />
</picture>
```

## 동영상 사이즈 최적화

- 동영상 압축
- 온라인 동영상 압축 사이트에서 하기
- [https://www.media.io/ko/video-compressor.html](https://www.media.io/ko/video-compressor.html)
- 효율이 좋은 포맷은 WEBM 포멧을 사용한다.
- 지원하지 않은 브라우저가 있다.
- 해당 방법으로 모두 지원 가능하다.

```jsx
<video
          className="absolute translateX--1/2 h-screen max-w-none min-w-screen -z-1 bg-black min-w-full min-h-screen"
          autoPlay
          loop
          muted
        >
          <source src={webmVideo} type="video/webm" />
          <source src={mp4Video} type="video/mp4" />
</video>
```

- 크기를 줄일 때는 무조건 줄이는 것보다 사용자가 불편하지 않도록 제공하는 것이 좋다.
- 화질도 좋게, 사이즈도 좋게하고 싶으면 동영상의 길이를 짧게 하거나, 화질을 줄이고 위에 패턴을 덧붙여서 사용자가 눈치채지 못하게 한다. css의 filter 효과를 줘도 된다.

## 폰트 최적화1(폰트 적용 시점 컨트롤)

- web font는 네트워크를 통해 받는 거여서 다운로드가 지연되서 적용되는 시점이 문제가 생길 수 있다.
- 웹 폰트의 문제점
    1. FOUT(Flash of Unstyled Text)

  → 폰트가 다운로드가 되기 전에는 기본 폰트로 텍스트를 보여준다.

  → IE, Edge 에서 사용하는 방식

    1. FOIT(Flash of Invisible Text)

  → 폰트가 다운로드 되기 전에는 텍스트를 노출 안함

  → Safari, Chrome 에서 사용하는 방식

- 웹 폰트의 최적화 방법
    1. 폰트 적용 시점 컨트롤 하기
    2. 폰트 사이즈 줄이기
- 폰트 적용 시점 컨트롤하기
    - css font-dispay를 사용한다.
        - auto : 브라우저 기본 동작
        - block : FOIT (timeout = 3s)
        - swap : FOUT
        - fallback : FOIT (timeout = 0.1s) 3초 후에도 불러오지 못했을 시, 기본 폰트로 유지, 이후 캐시
        - optional : FOIT (timeout = 0.1s) 이후 네트워크 상태에 따라, 기본 폰트로 유지할지 웹 폰트를 적용할지 결정, 이후 캐시
- 구글에서는 optional 방식을 권장한다.

```jsx
@font-face {
	font-family: BMYEONSUNG;
	src: url('./assets/fonts/BMYEONSUNG.ttf');
	font-display: optional;
}
```

- 강사는 block을 권장한다.(fade-in 애니메이션을 줘서 체감 성능을 개선한다.)

→ fontfaceobserver 모듈을 사용한다.

```jsx
npm i fontfaceobserver
```

```jsx
@font-face {
	font-family: BMYEONSUNG;
	src: url('./assets/fonts/BMYEONSUNG.ttf');
	font-display: block;
}

import React, { useState, useEffect } from "react";
import mp4Video from "../assets/banner-video.mp4";
import webmVideo from "../assets/banner-video.webm";
import FontFaceObserver from "fontfaceobserver";

function BannerVideo() {
  const [isFontLoaded, setIsFontLoaded] = useState(false);

  const font = new FontFaceObserver("BMYEONSUNG");

  useEffect(() => {
    font.load().then(() => {
      console.log("font load");
      setIsFontLoaded(true);
    });
  }, []);

  return (
    <div className="BannerVideo w-full h-screen overflow-hidden relative bg-texture">
      <div className="absolute h-screen w-full left-1/2">
        <video
          className="absolute translateX--1/2 h-screen max-w-none min-w-screen -z-1 bg-black min-w-full min-h-screen"
          autoPlay
          loop
          muted
        >
          <source src={webmVideo} type="video/webm" />
          <source src={mp4Video} type="video/mp4" />
        </video>
      </div>
      <div
        className="w-full h-full flex justify-center items-center"
        style={{
          opacity: isFontLoaded ? 1 : 0,
          transition: "opacity 0.3s ease",
        }}
      >
        <div className="text-white text-center">
          <div className="text-6xl leading-none font-semibold">KEEP</div>
          <div className="text-6xl leading-none font-semibold">CALM</div>
          <div className="text-3xl leading-loose">AND</div>
          <div className="text-6xl leading-none font-semibold">RIDE</div>
          <div className="text-5xl leading-tight font-semibold">LONGBOARD</div>
        </div>
      </div>
    </div>
  );
}

export default BannerVideo;
```

## 폰트 최적화 2(subset, unicode-range, data-uri)

- 폰트 사이즈 줄이는 방법
    - 웹폰트 포멧 사용
    - local 폰트 사용
    - Subset 사용
    - Unicode Range 적용
    - data-uri로 변환
- 웹폰트 포멧 사용
    - EOT > TTF/OTF > WOFF > WOFF2
    - 사이트 : transfonter.org

    ```jsx
    @font-face {
    	font-family: BMYEONSUNG;
    	src: url('./assets/fonts/BMYEONSUNG.woff2') format('woff2');
    	font-display: block;
    }
    ```

    - 지원하지 않는 브라우저가 있을 수 있다.
    - 해결방법

    ```jsx
    @font-face {
    	font-family: BMYEONSUNG;
    	src: url('./assets/fonts/BMYEONSUNG.woff2') format('woff2'), 
    		url('./assets/fonts/BMYEONSUNG.woff') format('woff'),
    		url('./assets/fonts/BMYEONSUNG.ttf') format('truetype');
    	font-display: block;
    }
    ```

    - 로컬에 있는 경우를 체크할 경우

    ```jsx
    @font-face {
      font-family: BMYEONSUNG;
      src: local("BMYEONSUNG"),
        url("./assets/fonts/BMYEONSUNG.woff2") format("woff2"),
        url("./assets/fonts/BMYEONSUNG.woff") format("woff"),
        url("./assets/fonts/BMYEONSUNG.ttf") format("truetype");
      font-display: block;
    }
    ```

- 폰트 사이즈 줄이기(Subset)
    - [transfonter.org](http://transfonter.org) 에서 한다.
    - 사용하는 글자만으로 subset 한다

    ```jsx
    @font-face {
      font-family: BMYEONSUNG;
      src: local("BMYEONSUNG"),
        url("./assets/fonts/subset-BMYEONSUNG.woff2") format("woff2"),
        url("./assets/fonts/subset-BMYEONSUNG.woff") format("woff"),
        url("./assets/fonts/BMYEONSUNG.ttf") format("truetype");
      font-display: block;
    }
    ```

- Unicode-range
    - 지정된 유니코드에 대해서만 폰트를 적용하겠다.

    ```jsx
    @font-face {
      font-family: BMYEONSUNG;
      src: local("BMYEONSUNG"),
        url("./assets/fonts/subset-BMYEONSUNG.woff2") format("woff2"),
        url("./assets/fonts/subset-BMYEONSUNG.woff") format("woff"),
        url("./assets/fonts/BMYEONSUNG.ttf") format("truetype");
      font-display: block;
      unicode-range: u+0041; /* A */
    }
    ```

- data-uri
    - 파일로 불러왔던걸 uri로 바꾼다. (Base64 encode option)

## 폰트 최적화 3(preload)

```jsx
<link rel="preload" href="BMYEONSUNG.woff2" as="font" type="font/woff2" crossorigin>
```

- css 보다 먼저 불려야 한다. (HTML Head 안에 link 태그로 명시해준다.)
- font를 어떤 리소스보다 빠르게 로드한다.
- webpack 커스텀이 필요하다.

```jsx
npm i -D react-app-rewired -D

"start": "npm run build:style && react-app-rewired start",
"build": "npm run build:style && react-app-rewired build",
```

- preload-webpack-plugin을 사용한다.

```jsx
npm i -D preload-webpack-plugin@3.0.0-beta.3
```

```jsx
const PreloadWebpackPlugin = require("preload-webpack-plugin");

module.exports = function override(config, env) {
  config.plugins.push(
    new PreloadWebpackPlugin({
      rel: "preload",
      as: "font",
      include: "allAssets",
      fileWhitelist: [/(.woff2?)/i],
    })
  );
  //do stuff with the webpack config...
  return config;
};
```