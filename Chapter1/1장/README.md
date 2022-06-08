## 웹 성능 최적화를 하는 이유

1. 사용자가 떠나지 않도록 하기 위해 = **수익 증대**를 위해
2. 프론트엔드 개발자로서 경쟁력을 갖추기 위해

## 웹 성능 결정 요소

1. 로딩 성능(각 리소스를 불러오는 성능)
    - 이미지 사이즈 최적화
    - Code Split
    - 텍스트 압축
2. 렌더링 성능(불러온 리소스를 화면에 그려주는 성능)
    - Bottleneck 코드 최적화

## Audits(Lighthouse) 툴을 이용한 페이지 검사

- devtools 에서 사용하면 된다. (chrome 신규 버전에서 추가)
- 점수로 확인 가능하다.
- Performance
    - METRICS : 검사 지표율
    - 스크린샷 : 페이지가 로드되는 스크린샷 흐름을 살필 수 있음
    - Opportunities : 웹페이지의 문제점, 리소스의 관점에서 제공(로딩 성능 최적화)
    - Diagnostics : 문제점에 해당하는 가이드라인, 페이지의 실행관점에서 제공(렌더링 성능 최적화)
    - Passed Audits : 우리가 이미 잘 적용하고 있는 항목들
    - Runtime Settings : 검사할 때 사용하는 환경을 요약해서 보여줌

## 이미지 사이즈 최적화

- 이미지 사이즈의 경우 너비기준으로 2배정도의 이미지를 사용하는 것이 적절하다. (120x120 ⇒ 240x240)
- 이미지 CDN을 사용하자.

→ CDN이란 물리적 거리의 한계를 극복하기 위해 소비자(사용자)와 가까운 곳에 컨텐츠 서버를 두는 기술

→ 그냥 CDN과 이미지 CDN은 다르다. 이미지 CDN은 image processing CDN으로 불리며, 기본적인 CDN의 개념과 이미지를 사용자에게 보내기 전에 특정형태로 가공을 해서(사이즈를 줄이거나, 이미지 포맷을 변경) 사용자에게 전달한다.

→ 예시

```jsx
http://cdn.image.com?src=[img src]&width=200&height=100
```

→ imageix 같은 솔루션을 사용하는게 편함

## Bottleneck 코드 탐색

- Performance tab으로 체크가능
- 컴포넌트가 길게 사용되고, 함수가 무의미하게 실행되는 부분을 찾을 수 있다.

## Bottleneck 코드 수정

- 기존 코드

```jsx
/*
 * 파라미터로 넘어온 문자열에서 일부 특수문자를 제거하는 함수
 * (Markdown으로 된 문자열의 특수문자를 제거하기 위함)
 * */
function removeSpecialCharacter(str) {
  const removeCharacters = ['#', '_', '*', '~', '&', ';', '!', '[', ']', '`', '>', '\n', '=', '-']
  let _str = str
  let i = 0,
    j = 0

  for (i = 0; i < removeCharacters.length; i++) {
    j = 0
    while (j < _str.length) {
      if (_str[j] === removeCharacters[i]) {
        _str = _str.substring(0, j).concat(_str.substring(j + 1))
        continue
      }
      j++
    }
  }

  return _str
}
```

위 코드의 해결방안은 크게 두가지로 나뉜다.

1. 특수 문자를 효율적으로 제거하기
    - replace 함수와 정규식 사용
    - 마크다운의 특수문자를 지워주는 라이브러리 사용(remove-markdown)

```jsx
function removeSpecialCharacter(str) {
  let _str = str;

  _str = _str.replace(/[\#\_\*\~\&\;\~\`\n\=\-]/g, "");

  return _str;
}
```

1. 작업하는 양 줄이기

```jsx
function removeSpecialCharacter(str) {
  let _str = str.substring(0, 300);

  _str = _str.replace(/[\#\_\*\~\&\;\~\`\n\=\-]/g, "");

  return _str;
}
```

## Bundle 파일 분석(bundle-analyzer)

- chunk file 의 상세내용 확인
- cra 에서는 eject 해서 사용해야 함. 아니면 웹팩을 커스텀할 수 있는 걸 설치해야 하는데 요즘은 cra-bundle-analyzer 가 나왔다.

```jsx
npm i -D cra-bundle-analyzer

npx cra-bundle-analyzer
```

- 크기가 큰 모듈이 어디서 사용하는지 확인하자

## Code Splitting & Lazy Loading

- 코드스플리팅이란?

→ 코드를 분할하는 것, 덩치가 큰 번들파일을 쪼개서 작은사이즈의 파일을 만드는 것을 말한다.

→ 사용하지 않은 페이지의 모듈까지 다른페이지에서 불러온다면 로딩속도가 저하된다. 그것을 분리하는 것을 말한다. (페이지 별로, 모듈 별로 분리 가능하다.)

→ 불필요한 코드 또는 중복되는 코드가 없이 적절한 사이즈의 코드가 적절한 타이밍에 로드될 수 있도록 하는 것을 의미한다.

→ Route-based code splitting을 적용해보자. (Lazy, Suspense)

→ Webpack 설정도 필요하다.(CRA, Next는 기본적으로 설정이 다 되어 있다.)

```jsx
import React, { Suspense, lazy } from "react";
import { Switch, Route } from "react-router-dom";
import "./App.css";
//import ListPage from "./pages/ListPage/index";
//import ViewPage from "./pages/ViewPage/index";

const ListPage = lazy(() => import("./pages/ListPage/index"));
const ViewPage = lazy(() => import("./pages/ViewPage/index"));

function App() {
  return (
    <div className="App">
      <Suspense fallback="Loading...">
        <Switch>
          <Route path="/" component={ListPage} exact />
          <Route path="/view/:id" component={ViewPage} exact />
        </Switch>
      </Suspense>
    </div>
  );
}

export default App;
```

## 텍스트 압축 사용

- 항상 최적화를 테스트 할 때는 dev환경 prod 환경 모두 테스트해봐야 한다.
- Performance test 시 `Enable text compression` 은 서버로부터 텍스트를 받을 때 압축을 해서 받아라 라는 의미이다. (Bundle 파일 텍스트 압축)
- 압축 알고리즘에서는 일반적으로 GZIP 과 Deflate(LZ7)를 사용함. (GZIP이 압출률이 더 좋음)

변경 전

```jsx
serve": "npm run build && node ./node_modules/serve/bin/serve.js -u -s build
```

변경 후

```jsx
serve": "npm run build && node ./node_modules/serve/bin/serve.js -s build
```

*파일의 크기가 2kb 이상이면 인코딩을 하고 그 이하면 하지 않는게 좋다.