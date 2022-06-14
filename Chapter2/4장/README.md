## 배울 것

- 로딩 성능 최적화
    - 이미지 지연(Lazy) 로딩
- 렌더링 성능 최적화
    - Layout Shift 피하기
    - useSelector 렌더링 문제 해결
    - Redux Reselect를 통한 렌더링 최적화
    - 병목 함수에 memoization 적용
    - 병목 함수 로직 개선하기

## 분석 툴

- 크롬 Network 탭
- 크롬 Performance 탭
- Lighthouse
- React Developer Tools(Profiler)
- Reux DevTools

## Layout Shift 피하기

- Layout Shift 란?

⇒ 화면상에서 요소들, 엘레먼트들이 어떤 요인에 의해서 위치나 사이즈가 바뀌는 것

- Performance 성능 체크할때 Layout Shift 로 표시됨
- Lighthouse에서도 표시됨(Cumulative Layout Shift)
- Layout Shift 원인
    - 사이즈가 정해져 있지 않은 이미지
    - 사이즈가 정해져 있지 않은 광고
    - 동적으로 삽입된 콘텐츠
    - Web font(FOIT, FOUT)
- 해결 방법

⇒ 사이즈를 정해주면 된다.

```jsx
// 변경전
const ImageWrap = styled.div``;

const Image = styled.img`
  cursor: pointer;
  width: 100%;
`;

// 변경후
const ImageWrap = styled.div`
 width: 100%;
 padding-bottom: 56.25%;
 position: relative;
`;

const Image = styled.img`
  cursor: pointer;
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
`;
```

## 이미지 지연(lazy) 로딩 (react-lazyload)

- intersection observer 가 아닌 react-lazyload 라이브러리를 활용한 lazy 로딩

```jsx
npm i react-lazyload
```

- intersection observer 보다 간단하게 구현이 가능하다는 장점이 있다.

```jsx
<ImageWrap>
      <LazyLoad offset={500}>
        <Image src={urls.small} alt={alt} onClick={openModal} />
      </LazyLoad>
</ImageWrap>
```

- scroll 이벤트를 사용해서 구현되어 있음

## useSelect 렌더링 문제 해결

- react devtools의 component에 Highlight updates when components render 옵션을 켜서 렌더링을 확인하자.
- useSelector 동작 원리(새로운 State를 비교해서 값이 다르다면 rendering 한다)
- useSelector 가 동작하는 방식이 리턴값을 기준으로 한다.

```jsx
useSelector(state => ({
	a: state.a,
  b: state.b
});
```

⇒ 이런식으로 적으면 다른 state가 변해도 object가 변하기 때문에 rerendering 한다.

- useSelector 문제 해결 방법
    - Object를 새로 만들지 않도록 State 쪼개기
    
    ```jsx
    const a = useSelector(state => state.a);
    const b = useSelector(state => state.b);
    ```
    
    - 새로운 Equality Function 사용
    
    ```jsx
    useSelector(state => ({
    	a: state.a,
      b: state.b
    }, shallowEqul);
    ```
    

## Redux Reselect를 통한 렌더링 최적화

- reselect는 라이브러리 이름으로 redux와 함께 사용한다. (불필요한 코드 제거 용의)
- 설치

```jsx
npm i reselect
```

```jsx
// 변경전
const {category, allPhotos, loading} = useSelector(state =>({
	category: state.category.category,
	allPhotos: state.photos.allPhotos,
	loading : state.photos.loading
}), shallowEqual)

// rerendering 방지
const photos = category === "all" ? allPhotos ? allPhotos.filter(photo => photo.category === category);

// 변경후
const selectFilteredPhotos = createSelector([
 (state) => state.photos.data,
state => state.category.category,
], (photos, category) => category === "all" ? photos ? photos.filter(photo => photo.category === category))

const photos = useSelector(selectFilteredPhotos);
const loading = useSelector(state => state.photos.loading);
```

- reslect는 memoization 기법을 사용하였다.
- 함수에 똑같은 인자가 들어오면 이 결과값을 미리 캐시된 값으로 반환을 해준다. (매번 계산을 하지 않는다.)
- 보통 selector 폴더하나를 만들고 미리 생성해둔다.

## 병목 함수에 memoization 적용

- memoization의 단점 : 최초에는 렌더링이 느릴 수 있다.(계산을 해야하기 때문에)
- memoization은 pure 함수에서만 사용 가능하다.

```jsx
// 변경전
export function getAverageColorOfImage(imgElement) {
  const canvas = document.createElement('canvas');
  const context = canvas.getContext && canvas.getContext('2d');
  const averageColor = {
    r: 0,
    g: 0,
    b: 0,
  };

  if (!context) {
    return averageColor;
  }

  const width = (canvas.width =
    imgElement.naturalWidth || imgElement.offsetWidth || imgElement.width);
  const height = (canvas.height =
    imgElement.naturalHeight || imgElement.offsetHeight || imgElement.height);

  context.drawImage(imgElement, 0, 0);

  const imageData = context.getImageData(0, 0, width, height).data;
  const length = imageData.length;

  for (let i = 0; i < length; i += 4) {
    averageColor.r += imageData[i];
    averageColor.g += imageData[i + 1];
    averageColor.b += imageData[i + 2];
  }

  const count = length / 4;
  averageColor.r = ~~(averageColor.r / count); // ~~ => convert to int
  averageColor.g = ~~(averageColor.g / count);
  averageColor.b = ~~(averageColor.b / count);

  return averageColor;
}
```

```jsx
// 변경 후
const cache = {};

export function getAverageColorOfImage(imgElement) {
  if (cache.hasOwnProperty(imgElement)) {
    return cache[imgElement];
  }

  const canvas = document.createElement('canvas');
  const context = canvas.getContext && canvas.getContext('2d');
  const averageColor = {
    r: 0,
    g: 0,
    b: 0,
  };

  if (!context) {
    return averageColor;
  }

  const width = (canvas.width =
    imgElement.naturalWidth || imgElement.offsetWidth || imgElement.width);
  const height = (canvas.height =
    imgElement.naturalHeight || imgElement.offsetHeight || imgElement.height);

  context.drawImage(imgElement, 0, 0);

  const imageData = context.getImageData(0, 0, width, height).data;
  const length = imageData.length;

  for (let i = 0; i < length; i += 4) {
    averageColor.r += imageData[i];
    averageColor.g += imageData[i + 1];
    averageColor.b += imageData[i + 2];
  }

  const count = length / 4;
  averageColor.r = ~~(averageColor.r / count); // ~~ => convert to int
  averageColor.g = ~~(averageColor.g / count);
  averageColor.b = ~~(averageColor.b / count);

  cache[imgElement] = averageColor;

  return averageColor;
}
```

- 위 코드 처럼 하면 memozation이 안된다. key 값이 object라 다 동일하게 되기 때문

```jsx
// 변경 후
const cache = {};

export function getAverageColorOfImage(imgElement) {
  if (cache.hasOwnProperty(imgElement.src)) {
    return cache[imgElement.src];
  }

  const canvas = document.createElement('canvas');
  const context = canvas.getContext && canvas.getContext('2d');
  const averageColor = {
    r: 0,
    g: 0,
    b: 0,
  };

  if (!context) {
    return averageColor;
  }

  const width = (canvas.width =
    imgElement.naturalWidth || imgElement.offsetWidth || imgElement.width);
  const height = (canvas.height =
    imgElement.naturalHeight || imgElement.offsetHeight || imgElement.height);

  context.drawImage(imgElement, 0, 0);

  const imageData = context.getImageData(0, 0, width, height).data;
  const length = imageData.length;

  for (let i = 0; i < length; i += 4) {
    averageColor.r += imageData[i];
    averageColor.g += imageData[i + 1];
    averageColor.b += imageData[i + 2];
  }

  const count = length / 4;
  averageColor.r = ~~(averageColor.r / count); // ~~ => convert to int
  averageColor.g = ~~(averageColor.g / count);
  averageColor.b = ~~(averageColor.b / count);

  cache[imgElement.src] = averageColor;

  return averageColor;
}
```

- 수정 : 고유 값인 src값으로 cache 저장
- 위 방법보 말고  memoization 함수로 바꿔주는 방법도 있다.