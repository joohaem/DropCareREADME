## DropCare w. (주)닥터스팹

> Care the Uncared Service

DropCare는 환자의 IV(Intra Venous) 주입 상태 또는 소변 정보에 대한 데이터를 실시간으로 수집 및 모니터링 하는 IoT 서비스입니다.

DropCare는 병동에 있는 의료 감독의 사각 지대에 있는 주요 데이터를 수집하여 환자의 이상 질병에 대한 예측 및 검출기 역할을 하도록 설계되었습니다.

또한 간호사의 업무 부담을 크게 줄이고 의사가 빠르고 정확한 결정을 내리도록 도와 질 높은 의료를 제공하며, 궁극적으로 환자의 생명을 구할 수 있을 것이라 기대합니다.

병실의 IoT 기기는 환자의 IV백 또는 소변백에 설치되어 정보를 Gateway로 송신하고 이 데이터를 웹을 통하여 수신합니다.

![드랍케어 ](https://user-images.githubusercontent.com/47105088/169686799-8d68ab6a-a8da-4daa-a90a-fa66aa607f33.png)

[[SKT IMPACTUPS] 닥터스팹 유투브 영상](https://www.youtube.com/watch?v=vlxZO1mdXaI)

[사용자 메뉴얼 :: WIKI 바로 가기](https://github.com/joohaem/DropCareREADME/wiki/DropCare-%EC%82%AC%EC%9A%A9-%EC%84%A4%EB%AA%85%EC%84%9C)

## 주요 기술

- VAC 패턴 (`components/.../~View.tsx`)
    - `Box` / `BoxView` 컴포넌트 등, 여러 컴포넌트에서 View 컴포넌트를 분리하여 UI 작업과 state 관리, handling을 분리하여 가져갔다
    - VAT 라이브러리를 통해 컴포넌트 작성이 편리하였고, 해당 디자인 패턴으로 `유지 보수 작업이 용이`하였다
    - 하지만, 이에 의해 props drilling 현상을 야기하고 다량의 prop 이 넘김으로써 단점이 존재한다는 것을 깨달았다
    - 이후 컴포넌트의 구성에 따라 적절한 컴포넌트 분리 패턴을 가지고 개발하게 될 것이라고 생각한다

- useCallback 과 React.memo (`components/aBox/BoxChartView.tsx`)
    - 부모 컴포넌트의 State가 하나라도 바뀌면, 자식 컴포넌트 모두가 리렌더링되는 문제 (1초당 바뀌는 데이터로 심각한 성능 문제를 야기하는 것을 확인할 수 있었다)
    - prop이 바뀌지 않을 때 리렌더링을 막는 `React.memo` (실시간 변화하는 컴포넌트가 아닌 곳, 특히 `차트를 보여주는 ChartView 컴포넌트`에 memo 사용하였다)
    - memo 를 위한 props 함수에는 useCallback 사용하여 비효율적인 리렌더링이 일어나지 않도록 구현하였다
    - 직접적인 성능 비교를 통한 성능 개선이 필요하다고 생각하지만, `성능 기록 도구들을 많이 활용하지 못해 아쉬운 마음`이 있다

- SWR vs React Query
    
    [[React][비교] SWR vs React Query vs Recoil selector ?](https://snupi.tistory.com/194)
    
    - 1초에 한 번씩의 잦은 리패칭이 필요하여, 패칭 라이브러리로 `React Query`를 결정하였다
    - React Query로 정보를 `1~2초`마다 불러온다
        - 해당하는 정보를 박스 내에 알맞게 보여주고,
        정보에 해당하는 링거 및 소변 정보가 사용자가 설정한 제한 범위에서 벗어나게 된다면, 해당하는 에러(`isAlertFromRangeValue`)를 확인하고 경고 알람을 표시(`isAlarmCycle`, `isAlertAnywhere`)하도록 구현하였다
    - React Query로 네트워크 끊김 확인하기 (`components/aBox/hooks/useFetchJsonData.ts`)
        - useQuery는 axios 를 통한 fetch를 진행하고, 그 결과를 query로 반환한다
        - 의료 모니터링을 하는 서비스 특성상, 더 신속하게 `10초 이상 연결에 실패`했을 때 우리는 네트워크기 끊긴 connection lost 상태(isFetchError)를 경고하는 것이 필수적이었다
        - 이를 위해 `axios 의 timeout` 값을 11초로 지정해주었고, `react-query의 failureCount 속성`을 이용하여 11초동안 1번 timeout이 일어났을 때부터 경고하도록 구현하였다

- 그래프 차트 (`components/aBox/chart/chartConfig.ts`)
    - JSON & CSV
        ![JSON & CSV](https://user-images.githubusercontent.com/47105088/169864066-9d13d284-4524-4b1b-b577-2dfe899ff50c.png)
       ` 1초에 한 번씩 환자 정보`를 받기 위해 `JSON 통신`을 하였고,
        `축적된 데이터를 차트로 가시화`하기 위해서 `CSV 테이블을 통신`하여 수신하였다
        많은 양의 데이터를 CSV 테이블로 받아옴으로써 보다 더 작은 용량으로 처리할 수 있었다
    - 차트 라이브러리로, Zoom in & out 기능이 편리하게 구현되어 있는 apexchart 라이브러리 사용하였으나, 
      큰 모듈 크기로 페인팅 시간이 방대하여 `chartjs 라이브러리` 사용하여 진행하였다
    - `react-chartjs-2` 라이브러리를 사용하여 `state를 활용하여 설정 정보를 구성하고 렌더링`하였다
    - 직접 `panning` 과 `zoom` 기능을 버튼 클릭을 이용하여 구현하였다
    - `차트 데이터 배열 처리`와 `DOM요소 useRef 훅`을 사용해, `CSV 파일과 PDF 파일 다운로드`를 구현하였다

- `Recoil` 과 `Local Storage` 관리의 이유
    - `Recoil`: 박스 리스트의 구성 정보들을 담고 첫 렌더링 시에 이를 활용할 수 있도록 하는 state
        - 이를 통해
        박스 리스트를 섹션에 알맞게 정렬하여 보여줌과 동시에
        섹션 종류를 바꿀 수 있는 `모드 체인지` 기능과
        `박스 리스트의 활성화 유무를 토글링`하는 기능
        등 을 구현하였다
    - `Local Storage`: 각 박스들의 비휘발성 정보들을 담고 첫 렌더링 시에 초기값들을 설정하도록 하는 string 값

- 드래그 앤 드랍 (`components/main/MainBoxSection.tsx`)
    - `react-beautiful-dnd` 라이브러리를 활용하였다
    - 박스의 헤더를 잡고 드래그하여 원하는 위치로 이동시킬 수 있도록 구현하였다
    - DragDropContext 컨택스트 내에서 DropResult 의 드래깅 이벤트를 정의한다
Droppable 박스를 배치해주고 그 내에서 박스마다의 Draggable 컴포넌트를 정의해주어 사용한다
이 때 Draggable 컴포넌트 내에서 박스를 정의하면서 provided.dragHandleProps 를 박스의 헤더에 전달하여 잡고 드래그 할 수 있는 속성을 부여한다

## 팀원

PM & Design [@nayoung](https://www.linkedin.com/in/na-young/)

Develop [@SNUPI](https://github.com/joohaem)

Main Layout Design [@jihyun](https://www.linkedin.com/in/jihyun-lee-ba5900215/)

## 기간

22/03 ~ 22/05, upper than 400 commits

## 폴더 구조

```
.
├──src
│   ├── assets
│   │   ├── icons
│   │   │   └── ...
│   │   ├── images
│   │   │   └── ...
│   │   └── assets.d.ts
│   ├── styles
│   │   ├── aBox
│   │   │   └── ...
│   │   └── popUp.ts
│   ├── utils
│   │   ├── libs
│   │   │   └── fetchJsonData.ts
│   │   ├── tempCsvData.ts
│   │   ├── initialJsonData.ts
│   │   ├── atoms.ts
│   │   └── types.ts
│   ├── components
│   │   ├── Router.tsx
│   │   ├── pages
│   │   │   ├── Error404.tsx
│   │   │   ├── PreMain.tsx
│   │   │   └── Main.tsx
│   │   ├── hooks
│   │   │   └── useOutClickCloser.ts
│   │   ├── main
│   │   │   ├── popUp
│   │   │   │   └── ...
│   │   │   └── ...
│   │   ├── aBox
│   │   │   ├── common
│   │   │   │   └── ...
│   │   │   ├── fluidBox
│   │   │   │   └── ...
│   │   │   ├── urineBox
│   │   │   │   └── ...
│   │   │   ├── popUp
│   │   │   │   └── ...
│   │   │   ├── chart
│   │   │   │   ├── BoxChart.tsx
│   │   │   │   ├── BoxChartView.tsx
│   │   │   │   ├── chartConfig.ts
│   │   │   │   └── ...
│   │   │   ├── util
│   │   │   │   └── parseToDoubleDigits.ts
│   │   │   ├── hooks
│   │   │   │   ├── useFetchJsonData.ts
│   │   │   │   ├── useRangeValueInput.ts
│   │   │   │   └── useFetchJsonData.ts
│   │   │   ├── ABox.tsx
│   └   ┴   └── ABoxView.tsx
├── .eslintrc.json
├── .prettierrc
└── tsconfig.json
```
