## 1. 개요

### About

> 한 종목에 투자금을 집중했다가 큰 돈을 잃어본 경험이 있나요?
> 
- 주식, 채권 등의 다양한 자산을 종합적으로 고려한 포트폴리오 생성 시뮬레이터
- 과거를 분석하고 미래를 예측하여 손실 위험을 최소화하고, 위기에서 반등의 기회를 제공
- 본 시뮬레이터는 아래 질문에 대한 해답을 제공할 두 서비스로 구성
    - 자산배분 : 현재 보유한 이 종목들의 비중을 어떻게 가져갔어야 지금 수익률이 최대였을까?
    - 트레이딩 : 종목 상관없이, 내가 구상한 매매 전략으로 진행하면 일년 후에는 어떻게 될까?
- 시뮬레이션 생성을 위한 정보를 form에 입력하여 제출하면, 연동된 AI model이 분석결과 생성
- 진행한 시뮬레이션 히스토리 내역을 확인하여 실제 투자에 참고 반영

### Tech Stack

![Untitled](https://user-images.githubusercontent.com/40906871/175820678-644b0195-4939-4b46-bd8b-627c93c3a630.png)


## 2. Role

> 자산배분, 트레이딩 시뮬레이터 서비스 전반의 front-end 개발
> 

### Routing

- **react-router-dom lib** 사용 (Browser Router)
- page component들은 React.lazy를 통해 코드 스플리팅. 비동기적으로 로딩 (for 렌더링 최적화)
- asset allocation, trading 두 서비스로 묶이는 페이지들은 각각 sub route 생성
- 로그인이 필요한 페이지의 경우 login check를 담당하는 PrivateRoute를 만들어 감쌌음
    - 해당 페이지들 상단에 login check code를 각각 포함시켰는데, 유지보수를 위해 분리

### Sign in / out

- **google 로그인** 사용
    - login 버튼 클릭시 구글 로그인 페이지로 이동
- url parameter로 redirect uri 를 전달 ( ~/loginprocess )
    - 구글 로그인 성공시 해당 redirect uri에 get method로 user token 생성 전달
- LoginProcess component에서 해당 token parsing
    - token을 포함한 user 정보를 redux store에 저장
    - isAuthenticated 등 인증 변수 활성화
- 로그아웃 시 user 관련 정보 및 변수 초기화, 메인 화면으로 이동

### form control

- react-hook-form, material-ui lib 사용
- 각 type별 input component 는 mui 사용
- 페이지 최상단 component에서 useForm Hooks 사용
- mui component를 Controller로 감싸고, control 함수를 각 input component Controller에 전달
- submit 클릭 시 form state를 가공하여 시뮬레이션 생성 axios 요청

```jsx
const { handleSubmit, control } = useForm();
import TextField from "@mui/material/TextField";
import Slider from "@mui/material/Slider";

const onSubmit = (data) => {
	// form data가 param으로 전달. 데이터 가공 후 시뮬레이션 생성 axios 요청
}

<form onSubmit={handleSubmit(onSubmit)}>
	<Controller control={control}	render={()=><TextField type="number" .../>} />
	<Controller control={control}	render={()=><Slider .../>} />
</form>
```

### state control

- redux
    - account : userId, isAuthenticated, jwt 등
    - simulation : isSimulating, simulationList 등
    - 그 외 여러 page, component 공용 state
- useState
    - chart mode, modal 등 지역 state 관리

### Axios Request / Response

- 종목 리스트 조회, 시뮬레이션 생성 등의 api 요청/응답 관리
- 공통 axios instance를 만들어 사용
- interceptors middleware를 추가하여 요청/응답 전후 에러 처리

```jsx
const instance = axios.create({
	baseURL : API_BASE_URL,
	headers : { "Content-Type" : "application/json" },
	withCredentials : false
});

instance.interceptors.request.use( // response도 동일한 방식으로 추가
	(config) => config,
	(error) => Promise.reject(error)
);
```

### Chart

- 사용자가 생성한 시뮬레이션 결과를 직관적으로 전달하도록 적절한 chart 활용
- Chart.js 라이브러리 사용 ( Pie, Doughnut, Line, Stacked Line )
- Chart comonent props : labels(각 dataset의 이름) / datasets(chart에 표시될 수치들)

```tsx
interface ChartProps {
	labels : String[]
	datasets : Array<{data: Number[]}>
	// datasets.length === labels.length
}
```
![Untitled (1)](https://user-images.githubusercontent.com/40906871/175820722-3ec27abb-0c8a-41ca-a5a5-de3278c54764.png)



### UI/UX 개선

- chart
    - 사용자의 목적에 맞는 분석 도모
    - line chart : 선형, 로그형 toggle 추가
    - pie chart : list view toggle 추가
    - chart zoom + panning + reset 기능
- CSS
    - styled-componenet
        - 기존에는 bootstrap과 className을 혼용하여 수정이 번거로웠음
        - styled-component로 통일하여 design 파일 분리 → 유지보수 용이
    - 반응형 작업
        - window 크기에 따른 component 크기 자동 조정
        - @media 쿼리와 react-responsive lib 활용
    - mui
        - Text, Date, Number, Range, Slider 등의 요소를 활용
        - form control 직관화. 모바일 호환
- modal
    - 종목 추가시 [칸 추가 → 모달 on → 종목 선택 후 닫기] 과정을 매 종목마다 반복해야 했음
        - 10개의 종목 추가시 모달을 10번 열어야 했음
    - 한번의 modal on으로 원하는 개수만큼 선택 후 닫는 방식으로 mouse click 최소화
        - multi-select 방식
    - **before**
        
        ![Untitled (2)](https://user-images.githubusercontent.com/40906871/175820726-90db58e1-f6c7-4f8d-b4f2-e8e0fcad459f.png)

        
    - **after**
        
        ![Untitled (3)](https://user-images.githubusercontent.com/40906871/175820727-aa6e38b6-1673-4035-940c-3ed8c405c64c.png)

        

## 3. 성능 최적화

### 측정 환경

- 브라우저
    - chrome secret mode + without cookie
- 도구
    - Lighthouse (**3회 측정 평균**)
- 대상
    - 메인페이지 ( ~.co.kr/ )
    - 시뮬레이션 생성 페이지 ( ~.co.kr/asset/backtest )
    - 시뮬레이션 결과 확인 페이지 ( ~.co.kr/asset/result )
- 지표
    - 성능 종합 점수
    - FCP (First Contentful Paint) - 첫 text/image 표시 시간
    - LCP (Largest Contentful Paint) - 최대 text/image 표시 시간
    - T2I (Time to Interactive) - 완전히 페이지와 상호작용할 수 있게 될 때까지 걸리는 시간
    - TBT (Total Blocking Time) - FCP와 T2I 사이 모든 시간의 합
    - CLS (Cumulative Layout Shift) - 표시 영역 안에 보이는 요소의 이동 측정. 시각적 안정성
    - Speed Index - 페이지 콘텐츠가 얼마나 빨리 표시되는가

### 최적화 순서

1. React.memo 적용
    - 컴포넌트 리렌더링 상황
        - props 변경, state 변경, 부모 컴포넌트의 리렌더링, forceUpdate 함수 실행 ( 발생빈도 ↓ )
    - 필요성
        - 컴포넌트의 props, state가 변경되면 rerendering 되어야 한다.
        - 하지만 부모 컴포넌트가 리렌더링 되었다 하여 무조건 자식 컴포넌트를 리렌더링 하는 것은 비효율적
        - 자식 컴포넌트의 props, state가 변경되지 않았다면, 기존 컴포넌트 재사용 하도록 React.memo 사용
    - 대상
        - props를 전달받지 않는 단순 컴포넌트
        - props가 자주 변하는 컴포넌트는 지양 → prevProps, nextProps 비교에 메모리 낭비
        - 각 라우터 최상위 index.tsx 파일은 적용 X
        - 하위 component에 적용
        
        ```tsx
        // before
        export default Component;
        
        // after
        export default React.memo(Component);
        ```
        
2. useMemo, useCallback 적용
    - 목적
        - 컴포넌트를 리렌더링할 때, 변경되지 않은 내부 함수/값을 새로 계산하지 않고 재사용
        - 자식 컴포넌트가 React.memo로 최적화되어 있는 경우, 자식에게 props로 전달하는 값, 콜백함수 등을 useMemo, useCallback으로 생성하여 전달.
    - 차이점
        - useMemo : 주로 숫자, 문자열, 객체 등 일반 값 기억
        - useCallback : 주로 함수
        - 둘 다 값/함수를 저장하는데 사용할 수 있다.
        - 아래 두 코드는 동일한 결과를 불러온다.
        
        ```tsx
        useCallback(()=>{
        	console.log("hello world");
        }, []);
        
        useMemo(()=>{
        	const fn = () => console.log("hello world");
        	return fn
        }, []);
        ```
        
    - 사용하지 말아야 할 경우
        - host component(`div`, `span`, `a`, `img` 등)에 전달하는 모든 항목 (리액트는 신경 안씀)
        - leaf component (최하위 자식 컴포넌트)
        - 전달하려는 값/함수가 새로운 참조여도 상관 없는 경우
    - 사용해야 하는 경우
        - 하위 트리에 많은 consumer가 있는 값을 전달하는 경우
        - 계산 비용이 많이 들고, 사용자 입력 값이  `map`, `filter` 사용했을 때와 같이 이후 렌더링 이후로도 참조적으로 동일할 가능성이 높은 경우
        - 자식 컴포넌트에서 `useEffect`가 반복적으로 trigger 되는 것을 막고 싶을 때
    
3. useState 함수형 업데이트로 변환
    - 기존
        - setFoo 함수에 param으로 변수 직접 전달
            
            → useCallback 최적화시 해당 변수를 의존성에 넣어주어야 하며, 
                값이 변할때마다 재할당됨 (사실상 Hooks 최적화 의미 없음)
            
    - 이후
        - setFoo 함수에 param 조작 방식을 함수로서 전달
            
            → 의존성이 필요 없어 재할당되지 않아 메모리를 아낄 수 있음. 최적화에 적합
            
        - 하지만 본 프로젝트에서는 redux로 상태관리했고, 
        useState를 사용한 경우는 loading변수 정도라 크게 의미가 있지는 않을 것으로 예상
        
        ```tsx
        // state 값을 직접 사용. param 직접 대입 (set 함수 내에 params 전달)
        const onIncrease = useCallback(()=>setNumber(number+1), [number]);
        
        // 이전 상태값을 어떻게 업데이트할지 정의하는 방식 -> 감시 배열 비어도 됨
        // set 함수 내에 **함수** 전달
        const onIncrease = useCallback(()=>setNumber(prev=>prev+1), []); 
        ```
        
    
4. 코드 스플리팅 : React.lazy, Suspense 적용
    - 대상
        - 현재 페이지에 포함되어 있지만, 사용자가 클릭하기 전이라 보여지지 않는 컴포넌트들
            
            ex) Modal, Route 등
            
        - 본 프로젝트에서는 두가지 유형으로 적용하였다.
            - 리액트 공식 문서 추천 : Routes 내부
                
                → 비로그인 상태 최초 화면 로딩 속도가 눈에 띄게 빨라지는 것을 느꼈음
                
            - Modal (image, result)
    - 적용 예시
        
        ```jsx
        const Home = lazy(() => import('./routes/Home'));
        const About = lazy(() => import('./routes/About'));
        
        const App = () => (
          <Router>
            <Suspense fallback={<div>Loading...</div>}>
              <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
              </Routes>
            </Suspense>
          </Router>
        );
        ```
        

1. 압축 적용
    - compress-create-react-app lib 적용
    - build : “react-scripts build && compress-cra”

### 최적화 전후 성능 비교

**메인페이지 ( ~.co.kr/ )**

<img src="https://user-images.githubusercontent.com/40906871/175820761-e54d962b-01b5-4b1d-ba2b-fca3449cc442.png" width="25%" /><img src="https://user-images.githubusercontent.com/40906871/175820907-112d7c6f-9fe3-483b-861b-645c2a85f290.png" width="45%" />


**시뮬레이션 생성 페이지 ( ~.co.kr/asset/backtest )**

<img src="https://user-images.githubusercontent.com/40906871/175820765-81ed38b1-eabf-468b-a622-3360648e7877.png" width="25%" /><img src="https://user-images.githubusercontent.com/40906871/175820768-63412bf8-dfb1-4570-b59f-44bac868c40e.png" width="45%" />

**시뮬레이션 결과 확인 페이지 ( ~.co.kr/asset/result )**

<img src="https://user-images.githubusercontent.com/40906871/175820771-2d0ca9bb-d7ce-4409-b747-adcbeb2a7733.png" width="25%" /><img src="https://user-images.githubusercontent.com/40906871/175820773-cf962c56-7dde-46a9-97ca-f82abccebceb.png" width="45%" />


## 4. 겪은 문제 및 해결

- chart zoom
    - 현상
        - chart-plugin-zoom lib 설치 후 zoom, panning 옵션 추가
        - zoom, panning 기능이 정상 작동하였으나 y축으로만 반응하고 x축 방향은 반응 없음
    - 원인
        - API 응답 데이터 내 x축 형식이 datetime 형식이어서 발생하는 오류로 파악
        - chart.js ㅇ에서 해당 형식 값의 날짜 전후 비교를 못하는 것으로 추정
    - 해결
        - x축의 type을 datetime time으로 변환 처리하여 해결

- log mode chart
    - 현상
        - 동일 데이터를 각각 선형, 로그형으로 paint 하였을 때, log 차트에서 값이 튀는 현상 발생
            
            ![Untitled (10)](https://user-images.githubusercontent.com/40906871/175820823-23229afa-2f68-4709-9c50-c4bd02438ece.png)

            
    - 원인
        - chart.js의 공백 처리 함수의 차이
        - 선형의 경우 없는 값을 무시하고 직전 직후의 데이터 값을 매끄럽게 연결
        - 로그형의 경우 전후 값 상관없이 해당 값을 0으로 강제 지정
    - 해결
        - 현재일 기준으로 과거로 거슬러 올라가며 값 보충

## 5. 마무리

### 아쉬운 점

- 실제 서비스로 배포되지 못하였다.
    - 프로젝트 담당자 8명 중 4명이 전문연구요원
    - 해당 인원들의 병역 문제로 프로젝트 이탈 (자세한 내용은 사내 보안상 생략합니다.)
    - 본 프로젝트는 AI model 이 핵심인데, 모두 전문연구요원 담당이었음
    - 이런 상황에 회장님이 원하는 서비스 노선이 변경되어 본 프로젝트는 여기서 마무리 되었음
- 최적화 각 단계별 향상 성능을 측정하지 않았다.
    - 각 단계별 개선 성능이 측정 환경에 따라 발생하는 오차에 가려질 것이라 생각하였다.
    - 최적화가 끝나고 보니 어떤 단계가 제일 dramatic 했는지 궁금하긴 하다.
    - 하지만 이를 위해서 다시 원상태로 돌려서 다시 진행하기에는 
    그 과정이 너무 힘들었기 때문에 그러지 않았다.
    - 다음에는 우선 순위를 정해보고 세부 단계별로 측정해볼 것이다.
