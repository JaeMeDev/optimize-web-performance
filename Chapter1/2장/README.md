## 2장

- 렌더링 성능 최적화
    - 애니메이션 최적화
- 로딩 성능 최적화
    - 컴포넌트 Lazy Loading
    - 컴포넌트 Preloading
    - 이미지 Preloading

## 애니메이션 분석(Reflow와 Repaint 이론)

- Performance CPU를 설정하면 애니메이션의 속도가 적당한지 확인가능하다.
- 애니메이션이 버벅이는 현상을 **쟁크 발생**이라 한다. 초당 Frame이 적을 때 발생한다. (Frame 누실)
- 브라우저 렌더링 과정을 알아야 한다.
    1. HTML, CSS 가공(DOM + CSSOM)
    2. 조합(Render Tree)
    3. 위치, 크기 계산(Layout)
    4. 색 채워 넣기(Paint)
    5. 각 레이어 합성(Composite)
- 위 전체적인 과정을 `Critical Rendering Path` , `Pixel Pipline` 이라고 부른다.

### Reflow

- width, height(위치나 크기)가 변경되었을 때 위 렌더링 과정을 모두 재실행

### Repaint

- color, background-color(색깔) 변경되었을 때 Layout을 생략한다.

### Reflow, Repaint 피하기(GPU 도움받기)

- transform, opacity(GPU가 관여할 수 있는 속성)을 변경했을 때 Layout, Paint 과정을 생략한다.

## 애니메이션 최적화

- 기존코드

```jsx
const BarGraph = styled.div`
    position: absolute;
    left: 0;
    top: 0;
    width: ${({width}) => width}%;
    transition: width 1.5s ease;
    height: 100%;
    background: ${({isSelected}) => isSelected ? 'rgba(126, 198, 81, 0.7)' : 'rgb(198, 198, 198)'};
    z-index: 1;
`
```

- 변경된 코드

```jsx
const BarGraph = styled.div`
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    transform: scaleX(${({width}) => width / 100});
    transform-origin: center left;
    transition: transform 1.5s ease;
    height: 100%;
    background: ${({isSelected}) => isSelected ? 'rgba(126, 198, 81, 0.7)' : 'rgb(198, 198, 198)'};
    z-index: 1;
`
```

## 컴포넌트 Lazy Loading(Code Splitting)

- cra-bundle-analyzer 설치 후 확인
- 수정된 코드

```jsx
const LazyImageModal = lazy(() => import("./components/ImageModal"));

function App() {
    const [showModal, setShowModal] = useState(false)

    return (
        <div className="App">
            <Header />
            <InfoTable />
            <ButtonModal onClick={() => { setShowModal(true) }}>올림픽 사진 보기</ButtonModal>
            <SurveyChart />
            <Footer />
            <Suspense fallback={null}>
            {showModal ? <LazyImageModal closeModal={() => { setShowModal(false) }} /> : null}
            </Suspense>
        </div>
    )
}
```

## 컴포넌트 Preloading

- Lazy Loading의 단점
    - 최초페이지에서는 성능이 빨라졌지만 모달을 띄울 때는 성능이 느려짐
- 컴포넌트 Preload
    - Preload(Click 하기 전에) 모달을 열기전에 모달과 관련된 코드를 미리 로드한다.
- 컴포넌트 Preload 타이밍
    - 버튼 위에 마우스를 올려 놨을 때

    ```jsx
    const LazyImageModal = lazy(() => import("./components/ImageModal"));
    
    function App() {
        const [showModal, setShowModal] = useState(false)
    
        const handleMouseEnter = () => {
            const component = import('./components/ImageModal')
        }
    
        return (
            <div className="App">
                <Header />
                <InfoTable />
                <ButtonModal onClick={() => { setShowModal(true) }} onMouseEnter={handleMouseEnter}>올림픽 사진 보기</ButtonModal>
                <SurveyChart />
                <Footer />
                <Suspense fallback={null}>
                {showModal ? <LazyImageModal closeModal={() => { setShowModal(false) }} /> : null}
                </Suspense>
            </div>
        )
    }
    ```

    - 최초 페이지 로드가 되고, 모든 컴포넌트의 마운트가 끝났을 때

    ```jsx
    const LazyImageModal = lazy(() => import("./components/ImageModal"));
    
    function App() {
        const [showModal, setShowModal] = useState(false)
    
        useEffect(() => {
            const component = import('./components/ImageModal')
        },[])
    
        // const handleMouseEnter = () => {
        //     const component = import('./components/ImageModal')
        // }
    
        return (
            <div className="App">
                <Header />
                <InfoTable />
                <ButtonModal onClick={() => { setShowModal(true) }}>올림픽 사진 보기</ButtonModal>
                <SurveyChart />
                <Footer />
                <Suspense fallback={null}>
                {showModal ? <LazyImageModal closeModal={() => { setShowModal(false) }} /> : null}
                </Suspense>
            </div>
        )
    }
    ```

    - 더 쉽게 사용하는 법(팩토리 패턴)

    ```jsx
    function lazyWithPreload(importFunction){
        const Component = React.lazy(importFunction);
        Component.preload = importFunction;
        return Component;
    }
    
    const LazyImageModal = lazyWithPreload(() => import('./components/ImageModal'))
    
    function App() {
        const [showModal, setShowModal] = useState(false)
    
        useEffect(() => {
            LazyImageModal.preload()
        },[])
    
        // const handleMouseEnter = () => {
        //     const component = import('./components/ImageModal')
        // }
    
        return (
            <div className="App">
                <Header />
                <InfoTable />
                <ButtonModal onClick={() => { setShowModal(true) }}>올림픽 사진 보기</ButtonModal>
                <SurveyChart />
                <Footer />
                <Suspense fallback={null}>
                {showModal ? <LazyImageModal closeModal={() => { setShowModal(false) }} /> : null}
                </Suspense>
            </div>
        )
    }
    ```

  ## 이미지 Preloading

    - 이미지를 미리 로드하는 것

    ```jsx
    const img = new Image()
    img.src=""
    ```

    - img.src 를 할 때 이미지가 로드되는 것을 이용하면 된다.